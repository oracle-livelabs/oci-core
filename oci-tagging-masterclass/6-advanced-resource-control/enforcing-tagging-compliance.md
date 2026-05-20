# Lab 6: Enforcing Tagging Compliance with OCI Functions

## Introduction

In the previous labs, you created tags, applied defaults, managed costs, and defined access control policies — all foundational elements of a tagging strategy. But what happens when a resource is created without the required tags, or when someone removes a tag after the fact? Without enforcement, even the best tagging strategy will degrade over time.

In this lab, you will implement automated enforcement of your tagging policies using OCI Functions and the Events service. You will build two serverless functions written in Python:

1. **A scheduled compliance scan** that searches for non-compliant compute instances across a compartment and stops them.
2. **An event-driven function** that is triggered automatically whenever a new compute instance is launched, checks whether it has the required tags, and stops it immediately if it does not comply.

Together, these two patterns cover both retroactive enforcement (catching existing non-compliant resources) and proactive enforcement (catching non-compliant resources at the time of creation).

**Estimated Time:** 30 minutes

### Objectives

In this lab, you will:

- Create a Dynamic Group and IAM policies to authorize OCI Functions to manage resources in your compartment.
- Create an OCI Functions application and configure it for your compartment and VCN.
- Deploy a Python function that scans for non-compliant compute instances and stops them.
- Create an Events rule that triggers a function whenever a new compute instance is launched.
- Deploy a second Python function that evaluates newly launched instances for tag compliance and stops non-compliant instances automatically.
- Test both enforcement patterns by creating compute instances with and without the required tags.

### Prerequisites

This lab assumes you have:

- Completed the previous labs in this workshop (specifically, you should have a tag namespace and defined tags already created).
- Access to OCI Cloud Shell or a local development environment with the Fn CLI and OCI CLI installed and configured.
- A Virtual Cloud Network (VCN) with at least one subnet in your workshop compartment.
- Familiarity with the OCI Console navigation.

## Task 1: Create a Dynamic Group and IAM Policies

**Important** If you completed lab 5, you can skip directly to task 2!

---

Before your functions can interact with other OCI services — such as querying for compute instances or stopping them — you must authorize them using a Dynamic Group and IAM policies. A Dynamic Group allows you to treat functions as principals (similar to users) and grant them permissions through policy statements.

1. Open the **Navigation Menu** in the OCI Console, navigate to **Identity & Security**, and select **Dynamic Groups**.

2. Click **Create Dynamic Group** and enter the following:

    - **Name:** `FunctionsTagManagement`
    - **Description:** `Dynamic group for tag management function(s)`
    - **Matching Rule:** Enter the following rule, replacing `<compartment_ocid>` with the OCID of your workshop compartment

    ```
    ALL {resource.type = 'fnfunc', resource.compartment.id = '<compartment_ocid>'}
    ```

    This rule includes all functions deployed in your workshop compartment.

3. Click **Create**.

4. Next, navigate to **Identity & Security > Policies** and click **Create Policy**. Enter the following:

    - **Name:** `TagManagementPolicy`
    - **Description:** `Allows Serverless functions to perform tag-related activities`
    - **Compartment:** Select a compartment where the policy can reference both your Functions application compartment and your isolated test compartment.

5. Add the following policy statements (click **Show manual editor** to enter them):

    ```
    <copy>
    Allow dynamic-group FunctionsTagManagement to manage instance-family in compartment <isolated_test_compartment_name>
    Allow dynamic-group FunctionsTagManagement to read all-resources in compartment <isolated_test_compartment_name>
    Allow service cloudEvents to use functions-family in compartment <functions_compartment_name>
    Allow any-user to manage functions-family in compartment <functions_compartment_name> where all {request.principal.type='resourceschedule'}
    </copy>
    ```

    The first statement allows the functions to stop compute instances in the isolated test compartment. The second allows them to search and read resource details there. The third allows the Events service to invoke functions in your Functions compartment.

6. Click **Create**.

## Task 2: Create a Functions Application

A Functions application is a logical grouping of functions. It defines the network (subnet) that your functions will use to communicate with other OCI services.

1. Open the **Navigation Menu**, navigate to **Developer Services**, and select **Functions**.

2. Make sure you are in your workshop compartment, then click **Create Application**.

3. Enter the following:

    - **Name:** `tag-enforcement-app`
    - **VCN:** Select the VCN in your workshop compartment.
    - **Subnet:** Select a subnet (a private subnet with a service gateway or a public subnet will work).

4. Click **Create**.

5. On the application detail page, note the **Getting Started** section. If you are using OCI Cloud Shell, follow the Cloud Shell setup steps. If you are using a local environment, follow the local setup steps to configure the Fn CLI context for your application.

    The key commands are:

    ```bash
    fn list context
    fn use context <your_region>
    fn update context oracle.compartment-id <your_compartment_ocid>
    fn update context oracle.profile DEFAULT
    fn update context oracle.region <your_region>
    fn update context api-url https://functions.<your_region>.oci.oraclecloud.com
    fn update context registry <region_code>.ocir.io/<tenancy_namespace>/tag-enforcement
    ```

    If you are running Fn CLI from a local machine rather than OCI Cloud Shell, also make sure the context provider is `oracle`:

    ```bash
    fn update context provider oracle
    ```

    > **Note:** If you already completed Lab 5, use the same Cloud Shell session. These parameters are already set.

## Task 3: Deploy the Scheduled Compliance Scan Function

This function scans all compute instances in a specified compartment using the OCI Resource Search service. For each instance, it checks whether the required defined tag is present with the expected value. If an instance is non-compliant, the function stops it.

This function is designed to be invoked on a schedule (for example, using the OCI Resource Scheduler or an external cron trigger) to periodically sweep for resources that have drifted out of compliance.

> **Important:** This function can stop running compute instances. For the workshop, point `COMPARTMENT_OCID` at an isolated test compartment that contains only resources you are willing to stop. Do not run the scheduled scan against a shared, production, or mixed-use compartment.

1. In Cloud Shell or your local terminal, create a new function boilerplate:

    >Note: If you are still in the *`tag-update`* folder inside Cloud Shell, type `cd ..` to move up one directory before you continue.

    ```bash
    fn init --runtime python scheduled-tag-scan
    cd scheduled-tag-scan
    ```

2. Open the `requirements.txt` file and replace its contents with:

    ```
    fdk>=0.1.105
    oci>=2.110.0
    ```

3. Open the `func.yaml` file. Increase the timeout and memory to accommodate the scan operation:

    ```yaml
    schema_version: 20180708
    name: scheduled-tag-scan
    version: 0.0.1
    runtime: python
    build_image: fnproject/python:3.11-dev
    run_image: fnproject/python:3.11
    entrypoint: /python/bin/fdk /function/func.py handler
    memory: 512
    timeout: 120
    ```

4. Replace the contents of `func.py` with the following code:

    ```python
    import io
    import json
    import logging

    import oci
    from fdk import response

    # Configure logging
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.INFO)

    # --- Configuration ---
    # These values can be overridden by setting application or function
    # configuration variables in the OCI Console.
    DEFAULT_TAG_NAMESPACE = "LLTagNamespace"
    DEFAULT_TAG_KEY = "CostCenter"
    DEFAULT_COMPLIANT_VALUE = None  # Set a value to require a specific tag value;
                                    # leave as None to only require the key exists.


    def get_signer():
        """Authenticate using Resource Principals (used when running in OCI Functions)."""
        try:
            signer = oci.auth.signers.get_resource_principals_signer()
            return signer
        except Exception as e:
            logger.error("Failed to get resource principals signer: %s", e)
            raise


    def get_config(ctx):
        """Read configuration from function/application config, with defaults."""
        cfg = ctx.Config()
        tag_namespace = cfg.get("TAG_NAMESPACE", DEFAULT_TAG_NAMESPACE)
        tag_key = cfg.get("TAG_KEY", DEFAULT_TAG_KEY)
        compliant_value = cfg.get("COMPLIANT_VALUE", DEFAULT_COMPLIANT_VALUE)
        compartment_id = cfg.get("COMPARTMENT_OCID")

        if not compartment_id:
            raise ValueError(
                "COMPARTMENT_OCID must be set as a configuration variable "
                "on the function or application."
            )

        return tag_namespace, tag_key, compliant_value, compartment_id


    def find_non_compliant_instances(search_client, compartment_id,
                                     tag_namespace, tag_key, compliant_value):
        """Search for running compute instances that lack the required tag."""
        query = (
            f"query instance resources where "
            f"compartmentId = '{compartment_id}' && "
            f"lifecycleState = 'RUNNING'"
        )

        search_details = oci.resource_search.models.StructuredSearchDetails(
            query=query,
            type="Structured"
        )

        non_compliant = []

        # Use pagination to handle compartments with many instances
        search_response = oci.pagination.list_call_get_all_results(
            search_client.search_resources,
            search_details
        )

        for resource in search_response.data:
            defined_tags = resource.defined_tags or {}
            namespace_tags = defined_tags.get(tag_namespace, {})

            if compliant_value:
                # Require both the key and a specific value
                if namespace_tags.get(tag_key) != compliant_value:
                    non_compliant.append(resource)
            else:
                # Require only that the key exists (any value is acceptable)
                if tag_key not in namespace_tags:
                    non_compliant.append(resource)

        return non_compliant


    def stop_instance(compute_client, instance_id, display_name):
        """Stop a running compute instance."""
        try:
            compute_client.instance_action(instance_id, "STOP")
            logger.info("Stopped instance: %s (%s)", display_name, instance_id)
            return {"id": instance_id, "name": display_name, "action": "STOPPED"}
        except oci.exceptions.ServiceError as e:
            logger.error(
                "Failed to stop instance %s (%s): %s",
                display_name, instance_id, e.message
            )
            return {"id": instance_id, "name": display_name, "action": "FAILED",
                    "error": e.message}


    def handler(ctx, data: io.BytesIO = None):
        """
        Main function handler.

        Scans for non-compliant running instances and stops them. Returns a
        JSON summary of actions taken.
        """
        try:
            tag_namespace, tag_key, compliant_value, compartment_id = get_config(ctx)
        except ValueError as e:
            logger.error(str(e))
            return response.Response(
                ctx,
                response_data=json.dumps({"error": str(e)}),
                headers={"Content-Type": "application/json"},
                status_code=400
            )

        signer = get_signer()
        search_client = oci.resource_search.ResourceSearchClient(
            config={}, signer=signer
        )
        compute_client = oci.core.ComputeClient(config={}, signer=signer)

        compliance_rule = f"{tag_namespace}.{tag_key}"
        if compliant_value:
            compliance_rule += f" = {compliant_value}"

        logger.info(
            "Scanning compartment %s for instances without: %s",
            compartment_id, compliance_rule
        )

        non_compliant = find_non_compliant_instances(
            search_client, compartment_id, tag_namespace, tag_key, compliant_value
        )

        if not non_compliant:
            logger.info("All running instances are compliant.")
            return response.Response(
                ctx,
                response_data=json.dumps({
                    "message": "All running instances are compliant.",
                    "instances_scanned": "all in compartment",
                    "non_compliant_count": 0
                }),
                headers={"Content-Type": "application/json"}
            )

        results = []
        for resource in non_compliant:
            result = stop_instance(
                compute_client, resource.identifier, resource.display_name
            )
            results.append(result)

        summary = {
            "message": f"Found {len(non_compliant)} non-compliant instance(s).",
            "compliance_rule": compliance_rule,
            "actions": results
        }

        logger.info("Compliance scan complete: %s", json.dumps(summary))

        return response.Response(
            ctx,
            response_data=json.dumps(summary),
            headers={"Content-Type": "application/json"}
        )
    ```

5. Deploy the function to your application:

    ```bash
    fn deploy --app tag-enforcement-app
    ```

6. Before invoking the function, set the required configuration variable. Navigate to your **Functions Application** in the OCI Console, select the **Configuration** tab, and add the following key:

    | Key | Value |
    |-----|-------|
    | `COMPARTMENT_OCID` | The OCID of an isolated test compartment that contains only resources you are willing to stop |
    | `TAG_NAMESPACE` | The tag namespace you created in a previous lab (e.g., `LLTagNamespace`) |
    | `TAG_KEY` | The tag key to enforce (e.g., `CostCenter`) |

    > **Note:** Setting these values as application-level configuration variables allows you to change the enforcement criteria without redeploying the function code.

7. Test the function from Cloud Shell or your terminal:

    ```bash
    fn invoke tag-enforcement-app scheduled-tag-scan
    ```

    The function will return a JSON response indicating how many non-compliant instances were found and what actions were taken.

    > **Tip:** To test this safely, create a small compute instance without the required tag in your isolated test compartment and verify that the function stops it. You can then add the required tag to the instance and re-run the scan to confirm it is no longer flagged.

## Task 4: Deploy the Event-Driven Compliance Function

The scheduled scan from Task 3 is useful for catching existing non-compliant resources, but there is a gap between scans where a non-compliant resource could be running. To close that gap, you will now create a function that is triggered by the OCI Events service whenever a new compute instance finishes launching.

When the Events service detects that a compute instance has been created, it sends the event payload to the function. The function then reads the instance OCID from the event, retrieves the instance details, checks for the required tag, and stops the instance if it is non-compliant.

1. From Cloud Shell or your terminal, create a new function:

    ```bash
    fn init --runtime python event-tag-check
    cd event-tag-check
    ```

2. Replace the contents of `requirements.txt`:

    ```
    fdk>=0.1.105
    oci>=2.110.0
    ```

3. Replace the contents of `func.yaml`:

    ```yaml
    schema_version: 20180708
    name: event-tag-check
    version: 0.0.1
    runtime: python
    build_image: fnproject/python:3.11-dev
    run_image: fnproject/python:3.11
    entrypoint: /python/bin/fdk /function/func.py handler
    memory: 256
    timeout: 60
    ```

4. Replace the contents of `func.py` with the following code:

    ```python
    import io
    import json
    import logging
    import time

    import oci
    from fdk import response

    # Configure logging
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.INFO)

    # --- Configuration ---
    DEFAULT_TAG_NAMESPACE = "LLTagNamespace"
    DEFAULT_TAG_KEY = "CostCenter"
    DEFAULT_COMPLIANT_VALUE = None


    def get_signer():
        """Authenticate using Resource Principals."""
        try:
            signer = oci.auth.signers.get_resource_principals_signer()
            return signer
        except Exception as e:
            logger.error("Failed to get resource principals signer: %s", e)
            raise


    def get_config(ctx):
        """Read configuration from function/application config."""
        cfg = ctx.Config()
        tag_namespace = cfg.get("TAG_NAMESPACE", DEFAULT_TAG_NAMESPACE)
        tag_key = cfg.get("TAG_KEY", DEFAULT_TAG_KEY)
        compliant_value = cfg.get("COMPLIANT_VALUE", DEFAULT_COMPLIANT_VALUE)
        return tag_namespace, tag_key, compliant_value


    def parse_event(data):
        """Parse the Cloud Event payload from the Events service."""
        try:
            body = json.loads(data.getvalue())
            event_type = body.get("eventType", "")
            instance_id = body.get("data", {}).get("resourceId")
            compartment_id = body.get("data", {}).get("compartmentId")
            resource_name = body.get("data", {}).get("resourceName", "unknown")

            if not instance_id:
                raise ValueError("No resourceId found in event payload.")

            return {
                "event_type": event_type,
                "instance_id": instance_id,
                "compartment_id": compartment_id,
                "resource_name": resource_name
            }
        except (json.JSONDecodeError, ValueError) as e:
            logger.error("Failed to parse event data: %s", e)
            raise


    def check_instance_compliance(compute_client, instance_id,
                                   tag_namespace, tag_key, compliant_value):
        """
        Retrieve the instance and check whether it has the required tag.

        Returns a tuple of (is_compliant: bool, instance_details).
        """
        # Brief delay to allow tags to propagate after instance creation
        time.sleep(5)

        instance = compute_client.get_instance(instance_id).data

        if instance.lifecycle_state not in ("RUNNING", "STARTING"):
            logger.info(
                "Instance %s is in state %s — skipping compliance check.",
                instance_id, instance.lifecycle_state
            )
            return True, instance

        defined_tags = instance.defined_tags or {}
        namespace_tags = defined_tags.get(tag_namespace, {})

        if compliant_value:
            is_compliant = namespace_tags.get(tag_key) == compliant_value
        else:
            is_compliant = tag_key in namespace_tags

        return is_compliant, instance


    def handler(ctx, data: io.BytesIO = None):
        """
        Event-driven handler.

        Receives a Cloud Event from the Events service when a compute instance
        is launched. Checks the instance for required tags and stops it if
        non-compliant.
        """
        tag_namespace, tag_key, compliant_value = get_config(ctx)

        try:
            event = parse_event(data)
        except (ValueError, json.JSONDecodeError) as e:
            return response.Response(
                ctx,
                response_data=json.dumps({"error": str(e)}),
                headers={"Content-Type": "application/json"},
                status_code=400
            )

        logger.info(
            "Received event: %s for instance %s (%s)",
            event["event_type"], event["resource_name"], event["instance_id"]
        )

        signer = get_signer()
        compute_client = oci.core.ComputeClient(config={}, signer=signer)

        try:
            is_compliant, instance = check_instance_compliance(
                compute_client, event["instance_id"],
                tag_namespace, tag_key, compliant_value
            )
        except oci.exceptions.ServiceError as e:
            logger.error("Failed to retrieve instance details: %s", e.message)
            return response.Response(
                ctx,
                response_data=json.dumps({
                    "error": f"Failed to retrieve instance: {e.message}"
                }),
                headers={"Content-Type": "application/json"},
                status_code=500
            )

        compliance_rule = f"{tag_namespace}.{tag_key}"
        if compliant_value:
            compliance_rule += f" = {compliant_value}"

        if is_compliant:
            logger.info(
                "Instance %s (%s) is compliant with %s.",
                instance.display_name, event["instance_id"], compliance_rule
            )
            return response.Response(
                ctx,
                response_data=json.dumps({
                    "instance": instance.display_name,
                    "status": "COMPLIANT",
                    "rule": compliance_rule,
                    "action": "NONE"
                }),
                headers={"Content-Type": "application/json"}
            )

        # Non-compliant — stop the instance
        logger.warning(
            "Instance %s (%s) is NON-COMPLIANT. Missing: %s. Stopping.",
            instance.display_name, event["instance_id"], compliance_rule
        )

        try:
            compute_client.instance_action(event["instance_id"], "STOP")
            action = "STOPPED"
        except oci.exceptions.ServiceError as e:
            logger.error("Failed to stop instance: %s", e.message)
            action = f"STOP_FAILED: {e.message}"

        return response.Response(
            ctx,
            response_data=json.dumps({
                "instance": instance.display_name,
                "status": "NON_COMPLIANT",
                "rule": compliance_rule,
                "action": action
            }),
            headers={"Content-Type": "application/json"}
        )
    ```

5. Deploy the function:

    ```bash
    fn deploy --app tag-enforcement-app
    ```

6. Set the same configuration variables on this function (or on the application, which will be inherited by both functions):

    | Key | Value |
    |-----|-------|
    | `TAG_NAMESPACE` | Your tag namespace (e.g., `LLTagNamespace`) |
    | `TAG_KEY` | Your tag key (e.g., `CostCenter`) |

## Task 5: Create an Events Rule

Now that the event-driven function is deployed, you need to create an Events rule that triggers it whenever a new compute instance is launched in your isolated test compartment.

1. Open the **Navigation Menu**, navigate to **Observability & Management**, and select **Events Service > Rules**.

2. Click **Create Rule** and enter the following:

    - **Display Name:** `enforce-tags-on-instance-launch`
    - **Description:** `Triggers tag compliance check when a compute instance is launched`

3. Under **Rule Conditions**, configure the following:

    - **Condition:** `Event Type`
    - **Service Name:** `Compute`
    - **Event Type:** `Instance - Launch End`

4. To scope the rule to your isolated test compartment only, add an additional condition:

    - Click **+ Another Condition**.
    - **Condition:** `Attribute`
    - **Attribute Name:** `compartmentId`
    - **Attribute Values:** Enter the OCID of your isolated test compartment.

5. Under **Actions**, configure:

    - **Action Type:** `Functions`
    - **Function Compartment:** Select your workshop compartment.
    - **Function Application:** `tag-enforcement-app`
    - **Function:** `event-tag-check`

6. Click **Create Rule**.

    The rule is active immediately. Any compute instance that finishes launching in your isolated test compartment will now trigger the `event-tag-check` function.

## Task 6: Test the Enforcement

You now have two enforcement mechanisms in place. Let's test them.

### Test the Event-Driven Enforcement

1. Navigate to **Compute > Instances** and click **Create Instance**.

2. In your isolated test compartment, create a small instance (for example, using the `VM.Standard.E4.Flex` shape with 1 OCPU and 8 GB memory). **Do not add the required defined tag**.

3. After the instance reaches the **Running** state, the Events service will fire the `Instance - Launch End` event. Within a minute, the `event-tag-check` function will be triggered.

4. Navigate back to your instance. You should see that its state has changed to **Stopping** or **Stopped**.

5. To verify, check the function logs. Navigate to **Developer Services > Functions**, select your application, then select the `event-tag-check` function and click **Logs**. You should see log entries indicating the instance was found to be non-compliant and was stopped.

### Test with a Compliant Instance

1. In the same isolated test compartment, create another small instance, but this time, expand the **Tagging** section during creation.

2. Under **Defined Tags**, select your tag namespace (e.g., `Operations`), select the key (e.g., `CostCenter`), and assign a value.

3. After the instance reaches the **Running** state, wait for the function to be triggered.

4. This time, the instance should remain running. The function logs should show that the instance was evaluated and found to be compliant.

### Test the Scheduled Scan

1. If you still have the non-compliant instance from the first test (now stopped), start it again from the Console or add another instance without the required tag.

2. Invoke the scheduled scan function manually:

    ```bash
    fn invoke tag-enforcement-app scheduled-tag-scan
    ```

3. Review the JSON output. It should report the non-compliant instance and confirm that it was stopped.

## Task 7: Clean Up (Optional)

If you want to remove the resources created in this lab:

1. Delete the Events rule (`enforce-tags-on-instance-launch`).
2. Delete the functions and the application (`tag-enforcement-app`) from the Functions console.
3. Delete the IAM policy (`TagManagementPolicy`).
4. Delete the Dynamic Group (`FunctionsTagManagement`).
5. Terminate any test compute instances you created.

Congratulations - you've completed the workshop!!

## Learn More

- [Overview of OCI Functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Concepts/functionsoverview.htm)
- [Overview of the Events Service](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventsoverview.htm)
- [Services That Produce Events (Event Type Reference)](https://docs.oracle.com/en-us/iaas/Content/Events/Reference/eventsproducers.htm)
- [OCI SDK Authentication Methods (Resource Principals)](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdk_authentication_methods.htm)
- [OCI Functions QuickStart Guides](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsquickstartguidestop.htm)
- [Invoking Functions Automatically with Cloud Events (Oracle Developers Blog)](https://blogs.oracle.com/developers/oracle-functions-invoking-functions-automatically-with-cloud-events)
- [Governance and Automation with OCI Events and Functions](https://www.ateam-oracle.com/post/little-known-ways-to-do-oci-governance-using-events-and-functions)

## Acknowledgements

- **Author** - Eli Schilling
- **Contributors** - Daniel Hart, Deion Locklear, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
