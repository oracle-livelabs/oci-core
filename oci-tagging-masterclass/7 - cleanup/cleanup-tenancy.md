# Cleanup: Remove Workshop Resources

## Introduction

In this cleanup lab, you will remove the temporary resources created during the OCI Tagging Masterclass. Cleanup is especially important if you created compute instances, block volumes, budgets, IAM test users, Events rules, functions, or Resource Scheduler schedules in a shared tenancy.

**Estimated Time:** 15-20 minutes

### Objectives

In this lab, you will:

- Terminate the compute instance created for bulk tagging.
- Delete block volumes and Object Storage buckets created for the workshop.
- Delete budget alert rules and budgets created for the workshop.
- Delete Events rules, Resource Scheduler schedules, Functions applications, IAM policies, dynamic groups, and test IAM users.
- Retire workshop tag defaults and tag definitions if they are no longer needed.

### Prerequisites

This lab assumes you have:

- Completed the previous labs.
- Administrative access to the workshop compartment and IAM resources.
- The names or OCIDs of the resources you created.

For the CLI options, open OCI Cloud Shell and set these variables once before running the task-specific commands:

```bash
<copy>
export compartment_ocid="<your_workshop_compartment_ocid>"
export compartment_name="<your_workshop_compartment_name>"
export tenancy_ocid=$(awk -F= '/^tenancy=/{print $2; exit}' ~/.oci/config)
export namespace=$(oci os ns get --query data --raw-output)
</copy>
```

If a command prints `Query returned empty result`, that means no matching workshop resource was found.

## Task 1: Delete Compute, Block Volume, and Object Storage Resources

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Compute > Instances** and terminate any workshop test instances, such as `LabCompute1` or non-compliant instances created in Lab 6.

2. Navigate to **Block Storage > Block Volumes** and delete workshop volumes, such as `LabVolume1` and `LabVolume2`.

3. Navigate to **Object Storage > Buckets** and delete workshop buckets, such as `tag-test-bucket`, `tag-test-bucket-cli`, or any bucket name that starts with `lab-tagging-bucket`.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell:

```bash
<copy>
echo "Terminating workshop compute instances..."
instance_ids=$(
  oci compute instance list \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?(\"display-name\"=='LabCompute1' || starts_with(\"display-name\", 'Lab6')) && \"lifecycle-state\"!='TERMINATED'].id | join(' ', @)" \
    --raw-output
)

for instance_id in $instance_ids; do
  echo "Terminating instance $instance_id"
  oci compute instance terminate \
    --instance-id "$instance_id" \
    --preserve-boot-volume false \
    --force
done

echo "Deleting workshop block volumes..."
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

echo "Deleting workshop Object Storage buckets..."
bucket_names=$(
  oci os bucket list \
    --namespace-name "$namespace" \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?name=='tag-test-bucket' || name=='tag-test-bucket-cli' || starts_with(name, 'lab-tagging-bucket')].name | join(' ', @)" \
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

2. Open each workshop budget and delete its alert rules.

3. Delete the workshop budget.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Budgets are tenancy-scoped. If your CLI is not set to your home region, add `--region <home_region>` to the budget commands.

```bash
<copy>
echo "Deleting workshop budgets and alert rules..."
budget_ids=$(
  oci budgets budget budget list \
    --compartment-id "$tenancy_ocid" \
    --all \
    --query "data[?\"display-name\"=='TaggingWorkshopCostCenterBudget' || starts_with(\"display-name\", 'TaggingWorkshop')].id | join(' ', @)" \
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
</copy>
```

</details>

## Task 3: Delete Automation Resources

If you completed Labs 5 or 6, remove the automation resources in this order:

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Governance & Administration > Resource Scheduler > Schedules** and delete schedules such as `daily-tag-default-update`.

2. Navigate to **Observability & Management > Events Service > Rules** and delete rules such as `enforce-tags-on-instance-launch`.

3. Navigate to **Developer Services > Functions**, open the workshop application, and delete functions such as `tag-update`, `scheduled-tag-scan`, and `event-tag-check`.

4. Delete Functions applications such as `tag-update-app` and `tag-enforcement-app`.

5. Navigate to **Identity & Security > Policies** and delete IAM policies such as `TagManagementPolicy` or `TagAutoUpdatePolicy`.

6. Navigate to **Identity & Security > Dynamic Groups** and delete dynamic groups such as `FunctionsTagManagement`.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell:

```bash
<copy>
echo "Deleting Resource Scheduler schedules..."
schedule_ids=$(
  oci resource-scheduler schedule list \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?\"display-name\"=='daily-tag-default-update' || starts_with(\"display-name\", 'TaggingWorkshop')].id | join(' ', @)" \
    --raw-output
)

for schedule_id in $schedule_ids; do
  echo "Deleting schedule $schedule_id"
  oci resource-scheduler schedule delete \
    --schedule-id "$schedule_id" \
    --force
done

echo "Deleting Events rules..."
rule_ids=$(
  oci events rule list \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?\"display-name\"=='enforce-tags-on-instance-launch' || starts_with(\"display-name\", 'TaggingFnValidationLaunchRule')].id | join(' ', @)" \
    --raw-output
)

for rule_id in $rule_ids; do
  echo "Deleting Events rule $rule_id"
  oci events rule delete \
    --rule-id "$rule_id" \
    --force
done

echo "Deleting Functions and applications..."
for app_name in tag-update-app tag-enforcement-app; do
  app_id=$(
    oci fn application list \
      --compartment-id "$compartment_ocid" \
      --all \
      --query "data[?\"display-name\"=='$app_name' && \"lifecycle-state\"!='DELETED'].id | [0]" \
      --raw-output
  )

  if [ -n "$app_id" ] && [ "$app_id" != "null" ]; then
    function_ids=$(
      oci fn function list \
        --application-id "$app_id" \
        --all \
        --query "data[?\"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
        --raw-output
    )

    for function_id in $function_ids; do
      echo "Deleting function $function_id"
      oci fn function delete \
        --function-id "$function_id" \
        --force
    done

    echo "Deleting Functions application $app_id"
    oci fn application delete \
      --application-id "$app_id" \
      --force
  fi
done

echo "Deleting workshop container images from OCIR..."
for repository_name in \
  auto-tag-project/tag-update \
  tag-enforcement/tag-update \
  tag-enforcement/scheduled-tag-scan \
  tag-enforcement/event-tag-check
do
  image_ids=$(
    oci artifacts container image list \
      --compartment-id "$compartment_ocid" \
      --repository-name "$repository_name" \
      --all \
      --query "data[].id | join(' ', @)" \
      --raw-output
  )

  for image_id in $image_ids; do
    echo "Deleting image $image_id from $repository_name"
    oci artifacts container image delete \
      --image-id "$image_id" \
      --force
  done
done

echo "Deleting IAM policies for automation..."
for policy_compartment_id in "$tenancy_ocid" "$compartment_ocid"; do
  for policy_name in TagManagementPolicy TagAutoUpdatePolicy TagManagementPolicyValidation; do
    policy_ids=$(
      oci iam policy list \
        --compartment-id "$policy_compartment_id" \
        --all \
        --query "data[?name=='$policy_name' && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
        --raw-output
    )

    for policy_id in $policy_ids; do
      echo "Deleting policy $policy_id"
      oci iam policy delete \
        --policy-id "$policy_id" \
        --force
    done
  done
done

echo "Deleting dynamic groups for automation..."
dynamic_group_ids=$(
  oci iam dynamic-group list \
    --compartment-id "$tenancy_ocid" \
    --all \
    --query "data[?name=='FunctionsTagManagement' && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
    --raw-output
)

for dynamic_group_id in $dynamic_group_ids; do
  echo "Deleting dynamic group $dynamic_group_id"
  oci iam dynamic-group delete \
    --dynamic-group-id "$dynamic_group_id" \
    --force
done
</copy>
```

</details>

## Task 4: Delete IAM Test Users and Groups

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Identity & Security > Users** and delete test users created for tag-based access control, such as `tagtestuser`.

2. Navigate to **Identity & Security > Groups** and delete test groups such as `TagTestUsers`.

3. Navigate to **Identity & Security > Policies** and delete tag-based access control policies such as `TagDeleteRestrictionDenyPolicy`.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell:

```bash
<copy>
echo "Deleting IAM test users..."
user_ids=$(
  oci iam user list \
    --compartment-id "$tenancy_ocid" \
    --all \
    --query "data[?(name=='tagtestuser' || starts_with(name, 'tagtestuser-')) && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
    --raw-output
)

for user_id in $user_ids; do
  user_group_ids=$(
    oci iam user list-groups \
      --user-id "$user_id" \
      --all \
      --query "data[].id | join(' ', @)" \
      --raw-output
  )

  for user_group_id in $user_group_ids; do
    echo "Removing user $user_id from group $user_group_id"
    oci iam group remove-user \
      --group-id "$user_group_id" \
      --user-id "$user_id" \
      --force
  done

  echo "Deleting user $user_id"
  oci iam user delete \
    --user-id "$user_id" \
    --force
done

echo "Deleting IAM test groups..."
group_ids=$(
  oci iam group list \
    --compartment-id "$tenancy_ocid" \
    --all \
    --query "data[?name=='TagTestUsers' && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
    --raw-output
)

for group_id in $group_ids; do
  echo "Deleting group $group_id"
  oci iam group delete \
    --group-id "$group_id" \
    --force
done

echo "Deleting tag-based access-control policies..."
for policy_compartment_id in "$tenancy_ocid" "$compartment_ocid"; do
  policy_ids=$(
    oci iam policy list \
      --compartment-id "$policy_compartment_id" \
      --all \
      --query "data[?name=='TagDeleteRestrictionDenyPolicy' && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
      --raw-output
  )

  for policy_id in $policy_ids; do
    echo "Deleting policy $policy_id"
    oci iam policy delete \
      --policy-id "$policy_id" \
      --force
  done
done
</copy>
```

</details>

## Task 5: Retire Tag Defaults and Tag Definitions

Only remove tags if they were created solely for this workshop and are not used by other teams or resources.

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Identity & Security > Compartments**, open your workshop compartment, and select **Tag Defaults**.

2. Delete tag defaults such as `LLTagNamespace.Environment` or `LLTagNamespace.ExpirationDate`.

3. Navigate to **Governance & Administration > Tag Namespaces** and open `LLTagNamespace`.

4. Retire tag definitions such as `CostCenter`, `Environment`, and `ExpirationDate` if they are no longer needed.

5. Retire the `LLTagNamespace` namespace if all of its tag definitions have been retired.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell. Review the namespace name before retiring anything; tag retirement affects every resource and policy that depends on those tags.

```bash
<copy>
echo "Deleting workshop tag defaults..."
tag_default_ids=$(
  oci iam tag-default list \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?(\"tag-definition-name\"=='Environment' || \"tag-definition-name\"=='ExpirationDate') && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
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

  for tag_name in CostCenter Environment ExpirationDate; do
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
- [Deleting functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsdeleting.htm)


## Acknowledgements

- **Author** - Wynne Yang
- **Contributors** - Daniel Hart, Deion Locklear, Eli Schilling, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
