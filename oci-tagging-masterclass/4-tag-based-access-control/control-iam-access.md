# Lab 4: Tag-Based Access Control

## Introduction

In this lab, you will use **Defined Tags within IAM policies** to control access to resources in Oracle Cloud Infrastructure (OCI). Tag-based access control allows administrators to grant or restrict permissions based on tag values instead of only using compartments. This provides more flexible and scalable governance.You will create a test group and user, write an IAM policy that evaluates the tag value, and confirm that users cannot delete resources tagged as **Production**.

**Estimated Time:** 25-30 minutes

### Objectives
In this lab, you will:
- Reuse the bucket created in Lab 1
- Create a test group and user
- Create a tag-based IAM policy
- Restrict deletion of Production resources
- Validate tag-based access control


### Prerequisites

This lab assumes you have:

- Completed previous tagging labs
- A defined tag namespace (`LLTagNamespace`)
- A defined tag key (`Environment`)
- A bucket created in Lab 1 with the tag default applied
- Administrative access to IAM
- Access to Cloud Shell or OCI CLI installed and configured (optional for CLI steps)

## Task 1: Navigate to your Bucket

In **Lab 1**, you created a bucket to confirm that Tag Defaults automatically applied tags to resources created in a compartment. You will reuse this resource to demonstrate how IAM policies evaluate tag values.

### Console Steps

1. Navigate to **Object Storage → Buckets**.

    ![Screenshot showing navigation to Object Storage - Buckets](./images/01-navigate-to-object-storage-buckets.png " ")

2. Locate the bucket created in Lab 1.

    ![Screenshot showing list of buckets](./images/02-list-of-buckets.png " ")

3. Open the bucket. 

4. Scroll to the **Tags** section. 

5. Confirm the following tag still exists

    LLTagNamespace.Environment = Prod

    ![Screenshot showing bucket tags](./images/03-bucket-tags-prod-environment.png " ")


<details>
<summary>CLI Method (Optional)</summary>

1. Retrieve bucket details:

    ```bash
    <copy>
    oci os bucket get --bucket-name tag-test-bucket-cli --namespace-name <object_storage_namespace>
    </copy>
    ```

2. Or we could streamline the command since we're here to look at tags:

    ```bash
    <copy>
    oci os bucket get --bucket-name tag-test-bucket-cli --namespace-name <object_storage_namespace> --query 'data."defined-tags"'
    </copy>
    ```

    You'll notice in the output it only returns the contents of the defined tags JSON element:

    ```text
    {
        "LLTagNamespace": {
            "Environment": "Prod"
        }
    }
    ```

</details>


## Task 2: Create a Test Group and User
To validate tag-based access control, you will create a separate group and user.
This user will attempt to delete the tagged resource.
Console Steps

1. **Navigate to Identity & Security → Domains**.

    ![Screenshot showing navigation to IAM Domains](./images/04-navigate-to-iam-domains.png " ")

2. Click on your Domain

    ![Screenshot showing default domain](./images/05-select-default-domain.png " ")

3. Navigate to User Management.

    ![Screenshot showing Domain Groups](./images/06-domain-groups-page.png " ")

4. Scroll and click  **Create Group**.
5. Enter
    **Name:**
    ```text
    <copy>
    TagTestUsers
    </copy>
    ```

6. Click **Create**.

    ![Screenshot showing create group dialog](./images/07-create-group-dialog.png " ")

7. Navigate to **Users** and click **Create**.

    ![Screenshot showing create users tab](./images/08-users-tab-create-user.png " ")

8. Enter **First Name**, **Last Name**, and an **Email** that is *not* currently associated with your cloud account.

7. Add the user to the **TagTestUsers** group.

8. Click **Create**.

    ![Screenshot showing create user dialog](./images/09-create-user-dialog.png " ")

9. Confirm the user is in the group **TagTestUsers**.

    ![Screenshot showing new user details and group membership](./images/10-user-details-group-membership.png " ")


<details>
<summary>CLI Method (Optional)</summary>

1. Create the group

    ```bash
    <copy>
    oci iam group create \
    --compartment-id <tenancy_ocid> \
    --name TagTestUsers \
    --description "Tag testing group"
    </copy>
    ```

2. Create the user

    ```bash
    <copy>
    oci iam user create \
    --compartment-id <tenancy_ocid> \
    --name tagtestuser \
    --description "Tag policy test user"
    </copy>
    ```

3. Add user to group (make sure to retrieve group and user OCIDs from output of previous commands)

    ```bash
    <copy>
    oci iam group add-user \
    --group-id <group_ocid> \
    --user-id <user_ocid>
    </copy>
    ```

</details>

## Task 3: Review a Tag-Based Deny Policy
Now you will review a **deny policy** that can block deletion of resources tagged as Prod. Treat this task as a policy-design exercise unless you are working in a tenancy where IAM deny policies are already enabled.

> **Important:** IAM deny policies are an opt-in tenancy feature. Enabling them is permanent, and tag-based deny policies can take up to 24 hours to become fully effective after the feature is first enabled. For this workshop, do not enable deny policies in a shared or production tenancy just to complete the lab. Use the sample policy below to understand the pattern, then try it in a tenancy where deny policies have already been enabled or where you are intentionally testing the feature.

### Console Steps

1. Navigate to **Identity & Security → Policies**.

    ![Screenshot showing navigation to IAM Policies](./images/11-navigate-to-iam-policies.png " ")

2. Click **Create Policy**.

    ![Screenshot showing list of policies](./images/12-list-of-policies-page.png " ")

3. Enter **Name:**

     ```text
        <copy>
        TagDeleteRestrictionDenyPolicy
        </copy>
        ```

4. Enter **Description:**

    ```text
       <copy>
       Deny delete of Prod-tagged resources for TagTestUsers
       </copy>
    ```

5. Toggle on Show Manual Editor.

6. In the **policy statement field**, enter:

    ```text
    <copy>
    Deny group TagTestUsers to manage buckets in tenancy where target.resource.tag.LLTagNamespace.Environment = 'Prod'
    </copy>
    ```

    If deny policies are not enabled in your tenancy, you can still review this statement and continue to the validation discussion below. If deny policies are enabled in your own test tenancy, you may create the policy and validate the behavior.

7. Click **Create**.

    ![Screenshot showing policy creation confirmation](./images/13-create-policy-confirmation.png " ")

This policy means:

Members of **TagTestUsers** are explicitly denied bucket management actions (including delete) when the bucket tag is `LLTagNamespace.Environment = Prod`.

<details>
<summary>CLI Method (Optional)</summary>

Run this command only in a tenancy where IAM deny policies are already enabled:

```text
<copy>
oci iam policy create \
  --compartment-id <tenancy_ocid> \
  --name TagDeleteRestrictionDenyPolicy \
  --description "Deny delete of Prod-tagged resources for TagTestUsers" \
  --statements '[
    "Deny group TagTestUsers to manage buckets in tenancy where target.resource.tag.LLTagNamespace.Environment = '\''Prod'\''"
  ]'
</copy>
```

> **Note:** If the CLI returns `Deny statement passed into a Allow compilation`, IAM deny policies have not been enabled in the tenancy. Do not enable the feature unless your tenancy owner has explicitly decided to test IAM deny policies.
</details>

## Task 4: Validate Tag-Based Access Control
If you created the deny policy in a tenancy where deny policies are enabled, confirm that the policy is working. If you did not create the policy, read through the validation steps so you understand the expected behavior.

### Console Validation

1. Sign out of the administrator account.

    ![Screenshot showing log out with existing user](./images/14-logout-admin-user-menu.png " ")

2. Sign in as **tagtestuser**.

    ![Screenshot showing log in with tag user](./images/15-login-tagtestuser-screen.png " ")

3. Navigate to **Object Storage → Buckets**.

    ![Screenshot showing navigation to Object Storage](./images/01-navigate-to-object-storage-buckets.png " ")

4. Attempt to **Delete tag-test-bucket**.

    ![Screenshot showing attempt to delete bucket](./images/16-attempt-delete-bucket.png " ")

5. You should receive a permissions error.

    ![Screenshot showing permissions error](./images/17-delete-bucket-permission-error.png " ")


This happens because the bucket is tagged as Prod, and the deny policy explicitly blocks deletion (a manage-level action) for that tag value.

<details>
<summary>CLI Validation (Optional)</summary>

Using Cloud Shell (or configured CLI profile for tagtestuser):

    ```bash
    <copy>
    oci os bucket delete --bucket-name tag-test-bucket-cli --namespace-name <object_storage_namespace> --force
    </copy>
    ```
If the deny policy was created and has propagated, you should receive a "Not authorized" error.

</details>

## Task 5: (Optional Test)

1. Log back in as **Administrator**.
2. Change the bucket tag value from **Prod to Dev**.
3. Log back in as **tagtestuser**.
4. Attempt deletion again. This time, deletion should succeed. This demonstrates how tag values directly influence access control.

**Check Your Work**

You should now have:

- A bucket tagged as Production
- A test group and user, if you completed the hands-on identity setup
- A sample tag-based deny policy you can use in your own deny-policy-enabled tenancy
- An understanding of how deletion is blocked for Production-tagged resources when the policy is active
- An understanding of how changing the tag value changes access behavior

## Summary

In this lab, you:

- Applied a defined tag to a resource
- Created a test group and user
- Reviewed a tag-based IAM deny policy
- Learned how deletion can be restricted based on tag value
- Reviewed the validation flow for dynamic access control behavior
- Tag-based access control allows you to move beyond compartment-only permissions and implement flexible, metadata-driven security in OCI.

## Learn More

- [Using tags to manage access](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/managingaccesswithtags.htm)
- [Deny Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/policysyntax/denypolicies.htm)
- [Deny Policies Known Issues](https://docs.oracle.com/en-us/iaas/Content/Identity/known-issues/known-issues-deny-policies.htm)
- [Concepts guide: Tag-based access control](https://docs.oracle.com/en/engineered-systems/private-cloud-appliance/3.0-latest/concept/concept-tag-access.html#tag-access-example-taggedtargetcompt)
- [Improving the Aministrative ... Experience ...](https://blogs.oracle.com/cloud-infrastructure/improving-console-experience-with-tbac-in-oci)

## Acknowledgements

- **Author** - Deion Locklear
- **Contributors** - Daniel Hart, Eli Schilling, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
