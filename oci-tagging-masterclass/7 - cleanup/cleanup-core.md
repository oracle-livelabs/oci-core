# Cleanup: Remove Core Workshop Resources

## Introduction

In this cleanup lab, you will remove the temporary resources created during the OCI Tagging Core Capabilities workshop. This cleanup covers only Labs 1, 2, 3a, and 3b.

**Estimated Time:** 10-15 minutes

### Objectives

In this lab, you will:

- Terminate the compute instance created for bulk tagging.
- Delete block volumes and Object Storage buckets created in the core labs.
- Delete the budget alert rules and budgets created in the cost-management lab.
- Retire the workshop tag default, tag definitions, and tag namespace if they are not used by other labs or teams.

### Prerequisites

This lab assumes you have:

- Completed the OCI Tagging Core Capabilities labs.
- Administrative access to the workshop compartment and IAM tagging resources.
- The names or OCIDs of the resources you created.

For the CLI options, open OCI Cloud Shell and set these variables once before running the task-specific commands:

```bash
<copy>
export compartment_ocid="<your_workshop_compartment_ocid>"
export compartment_name="<your_workshop_compartment_name>"
export tenancy_ocid=$(awk -F= '/^tenancy=/{print $2; exit}' ~/.oci/config)
export namespace=$(oci os ns get --query data --raw-output)
export budget_display_name="<your_budget_display_name>"
</copy>
```

If a command prints `Query returned empty result`, that means no matching core workshop resource was found.

## Task 1: Delete Compute, Block Volume, and Object Storage Resources

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Compute > Instances** and terminate the workshop test instance, such as `LabCompute1`.

2. Navigate to **Block Storage > Block Volumes** and delete workshop volumes, such as `LabVolume1` and `LabVolume2`.

3. Navigate to **Object Storage > Buckets** and delete workshop buckets, such as `tag-test-bucket` and `tag-test-bucket-cli`.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell:

```bash
<copy>
echo "Terminating the core workshop compute instance..."
instance_ids=$(
  oci compute instance list \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?\"display-name\"=='LabCompute1' && \"lifecycle-state\"!='TERMINATED'].id | join(' ', @)" \
    --raw-output
)

for instance_id in $instance_ids; do
  echo "Terminating instance $instance_id"
  oci compute instance terminate \
    --instance-id "$instance_id" \
    --preserve-boot-volume false \
    --force
done

echo "Deleting core workshop block volumes..."
for volume_name in LabVolume1 LabVolume2; do
  volume_ids=$(
    oci bv volume list \
      --compartment-id "$compartment_ocid" \
      --all \
      --query "data[?\"display-name\"=='$volume_name' && \"lifecycle-state\"!='TERMINATED'].id | join(' ', @)" \
      --raw-output
  )

  for volume_id in $volume_ids; do
    attachment_ids=$(
      oci compute volume-attachment list \
        --compartment-id "$compartment_ocid" \
        --volume-id "$volume_id" \
        --all \
        --query "data[?\"lifecycle-state\"!='DETACHED'].id | join(' ', @)" \
        --raw-output
    )

    for attachment_id in $attachment_ids; do
      echo "Detaching volume attachment $attachment_id"
      oci compute volume-attachment detach \
        --volume-attachment-id "$attachment_id" \
        --force
    done

    echo "Deleting volume $volume_id"
    oci bv volume delete \
      --volume-id "$volume_id" \
      --force
  done
done

echo "Deleting core workshop Object Storage buckets..."
bucket_names=$(
  oci os bucket list \
    --namespace-name "$namespace" \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?name=='tag-test-bucket' || name=='tag-test-bucket-cli'].name | join(' ', @)" \
    --raw-output
)

for bucket_name in $bucket_names; do
  echo "Emptying bucket $bucket_name"
  oci os object bulk-delete \
    --namespace-name "$namespace" \
    --bucket-name "$bucket_name" \
    --force

  echo "Deleting bucket $bucket_name"
  oci os bucket delete \
    --namespace-name "$namespace" \
    --bucket-name "$bucket_name" \
    --empty \
    --force
done
</copy>
```

</details>

## Task 2: Delete Budgets and Alert Rules

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Billing & Cost Management > Budgets** in your home region.

2. Open the budget you created in Lab 3a.

3. Delete any alert rules associated with that budget.

4. Delete the budget.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Budgets are tenancy-scoped. If your CLI is not set to your home region, add `--region <home_region>` to the budget commands.

If you are not sure which budget you created, list budgets first and copy the display name into `budget_display_name`:

```bash
<copy>
oci budgets budget budget list \
  --compartment-id "$tenancy_ocid" \
  --all \
  --query "data[].{Name:\"display-name\",Id:id,TargetType:\"target-type\",Targets:targets}" \
  --output table
</copy>
```

Then run:

```bash
<copy>
if [ -z "$budget_display_name" ] || [ "$budget_display_name" = "<your_budget_display_name>" ]; then
  echo "Set budget_display_name to the budget created in Lab 3a before deleting."
else
  budget_ids=$(
    oci budgets budget budget list \
      --compartment-id "$tenancy_ocid" \
      --all \
      --query "data[?\"display-name\"=='$budget_display_name'].id | join(' ', @)" \
      --raw-output
  )

  for budget_id in $budget_ids; do
    alert_rule_ids=$(
      oci budgets budget alert-rule list \
        --budget-id "$budget_id" \
        --all \
        --query "data[].id | join(' ', @)" \
        --raw-output
    )

    for alert_rule_id in $alert_rule_ids; do
      echo "Deleting alert rule $alert_rule_id"
      oci budgets budget alert-rule delete \
        --budget-id "$budget_id" \
        --alert-rule-id "$alert_rule_id" \
        --force
    done

    echo "Deleting budget $budget_id"
    oci budgets budget budget delete \
      --budget-id "$budget_id" \
      --force
  done
fi
</copy>
```

</details>

## Task 3: Retire Tag Defaults and Tag Definitions

Only remove tags if they were created solely for this workshop and are not used by other teams, policies, automation, or resources.

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Identity & Security > Compartments**, open your workshop compartment, and select **Tag Defaults**.

2. Delete the `LLTagNamespace.Environment` tag default created in Lab 1.

3. Navigate to **Governance & Administration > Tag Namespaces** and open `LLTagNamespace`.

4. Retire the `CostCenter` and `Environment` tag definitions if they are no longer needed.

5. Retire the `LLTagNamespace` namespace if all of its tag definitions have been retired and the namespace is not needed for the advanced workshop.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell. Review the namespace name before retiring anything; tag retirement affects every resource and policy that depends on those tags.

```bash
<copy>
echo "Deleting the core workshop tag default..."
tag_default_ids=$(
  oci iam tag-default list \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?\"tag-definition-name\"=='Environment' && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
    --raw-output
)

for tag_default_id in $tag_default_ids; do
  echo "Deleting tag default $tag_default_id"
  oci iam tag-default delete \
    --tag-default-id "$tag_default_id" \
    --force
done

tag_namespace_id=$(
  oci iam tag-namespace list \
    --compartment-id "$tenancy_ocid" \
    --include-subcompartments true \
    --all \
    --query "data[?name=='LLTagNamespace' || name=='LLTagNameSpace'].id | [0]" \
    --raw-output
)

if [ -n "$tag_namespace_id" ] && [ "$tag_namespace_id" != "null" ]; then
  echo "Found tag namespace $tag_namespace_id"

  for tag_name in CostCenter Environment; do
    tag_state=$(
      oci iam tag get \
        --tag-name "$tag_name" \
        --tag-namespace-id "$tag_namespace_id" \
        --query "data.\"lifecycle-state\"" \
        --raw-output 2>/dev/null || true
    )

    if [ -n "$tag_state" ] && [ "$tag_state" != "RETIRED" ]; then
      echo "Retiring tag $tag_name"
      oci iam tag retire \
        --tag-name "$tag_name" \
        --tag-namespace-id "$tag_namespace_id"
    fi
  done

  echo "Retiring tag namespace $tag_namespace_id"
  oci iam tag-namespace retire \
    --tag-namespace-id "$tag_namespace_id"
fi
</copy>
```

</details>

## Learn More

- [Deleting tag defaults](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/managingtagdefaults.htm)
- [Deleting buckets](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets.htm)
- [Deleting budgets](https://docs.oracle.com/en-us/iaas/Content/Billing/Tasks/delete-budget.htm)

## Acknowledgements

- **Author** - Wynne Yang
- **Contributors** - Daniel Hart, Deion Locklear, Eli Schilling, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
