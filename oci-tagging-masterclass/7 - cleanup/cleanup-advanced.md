# Cleanup: Remove Advanced Workshop Resources

## Introduction

In this cleanup lab, you will remove the temporary resources created during the OCI Tagging Advanced Features workshop. This cleanup covers only Labs 4, 5, and 6.

**Estimated Time:** 15-20 minutes

### Objectives

In this lab, you will:

- Delete IAM test users, groups, and optional tag-based deny policies created in Lab 4.
- Delete Resource Scheduler schedules, Events rules, Functions applications, and OCIR images created in Labs 5 and 6.
- Terminate Lab 6 test compute instances from the isolated test compartment.
- Remove the `ExpirationDate` tag default and tag definition if they were created only for the advanced labs.

### Prerequisites

This lab assumes you have:

- Completed the OCI Tagging Advanced Features labs.
- Administrative access to IAM, Functions, Events, Resource Scheduler, and the workshop compartment.
- The names or OCIDs of the resources you created.

For the CLI options, open OCI Cloud Shell and set these variables once before running the task-specific commands:

```bash
<copy>
export compartment_ocid="<your_workshop_or_functions_compartment_ocid>"
export isolated_compartment_ocid="<your_lab6_isolated_test_compartment_ocid>"
export compartment_name="<your_workshop_or_functions_compartment_name>"
export tenancy_ocid=$(awk -F= '/^tenancy=/{print $2; exit}' ~/.oci/config)
export namespace=$(oci os ns get --query data --raw-output)
</copy>
```

If a command prints `Query returned empty result`, that means no matching advanced workshop resource was found.

## Task 1: Delete IAM Test Users and Groups

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Identity & Security > Users** and delete test users created for tag-based access control, such as `tagtestuser`.

2. Navigate to **Identity & Security > Groups** and delete test groups such as `TagTestUsers`.

3. Navigate to **Identity & Security > Policies** and delete tag-based access-control policies such as `TagDeleteRestrictionDenyPolicy`, if you created the optional deny policy in a deny-policy-enabled tenancy.

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

echo "Deleting optional tag-based access-control policies..."
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

## Task 2: Delete Automation Resources

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Governance & Administration > Resource Scheduler > Schedules** and delete schedules such as `daily-tag-default-update`.

2. Navigate to **Observability & Management > Events Service > Rules** and delete rules such as `enforce-tags-on-instance-launch`.

3. Navigate to **Developer Services > Functions**, open the workshop applications, and delete functions such as `tag-update`, `scheduled-tag-scan`, and `event-tag-check`.

4. Delete Functions applications such as `tag-update-app` and `tag-enforcement-app`.

5. Delete workshop container images from OCIR repositories such as `auto-tag-project/tag-update`, `tag-enforcement/scheduled-tag-scan`, and `tag-enforcement/event-tag-check`.

6. Navigate to **Identity & Security > Policies** and delete automation policies such as `TagManagementPolicy`.

7. Navigate to **Identity & Security > Dynamic Groups** and delete dynamic groups such as `FunctionsTagManagement`.

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
  tag-enforcement/scheduled-tag-scan \
  tag-enforcement/event-tag-check
do
  image_ids=$(
    oci artifacts container image list \
      --compartment-id "$compartment_ocid" \
      --repository-name "$repository_name" \
      --all \
      --query "data[].id | join(' ', @)" \
      --raw-output 2>/dev/null || true
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
for dynamic_group_name in FunctionsTagManagement FunctionsTagUpdate; do
  dynamic_group_ids=$(
    oci iam dynamic-group list \
      --compartment-id "$tenancy_ocid" \
      --all \
      --query "data[?name=='$dynamic_group_name' && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
      --raw-output
  )

  for dynamic_group_id in $dynamic_group_ids; do
    echo "Deleting dynamic group $dynamic_group_id"
    oci iam dynamic-group delete \
      --dynamic-group-id "$dynamic_group_id" \
      --force
  done
done
</copy>
```

</details>

## Task 3: Delete Lab 6 Test Instances

Only run this task against the isolated test compartment used for Lab 6. Do not use these commands against a shared, production, or mixed-use compartment.

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Compute > Instances**.

2. Switch to the isolated test compartment used in Lab 6.

3. Terminate the temporary compliant and non-compliant instances you created to validate the scheduled scan and event-driven enforcement.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell. The command lists candidate instances first and asks for confirmation before terminating them.

```bash
<copy>
if [ -z "$isolated_compartment_ocid" ] || [ "$isolated_compartment_ocid" = "<your_lab6_isolated_test_compartment_ocid>" ]; then
  echo "Set isolated_compartment_ocid to the Lab 6 isolated test compartment before terminating instances."
else
  echo "Candidate Lab 6 instances in the isolated test compartment:"
  oci compute instance list \
    --compartment-id "$isolated_compartment_ocid" \
    --all \
    --query "data[?\"lifecycle-state\"!='TERMINATED'].{Name:\"display-name\",State:\"lifecycle-state\",Id:id}" \
    --output table

  read -p "Type TERMINATE-LAB6 to terminate all listed instances: " confirm
  if [ "$confirm" = "TERMINATE-LAB6" ]; then
    instance_ids=$(
      oci compute instance list \
        --compartment-id "$isolated_compartment_ocid" \
        --all \
        --query "data[?\"lifecycle-state\"!='TERMINATED'].id | join(' ', @)" \
        --raw-output
    )

    for instance_id in $instance_ids; do
      echo "Terminating instance $instance_id"
      oci compute instance terminate \
        --instance-id "$instance_id" \
        --preserve-boot-volume false \
        --force
    done
  else
    echo "No instances were terminated."
  fi
fi
</copy>
```

</details>

## Task 4: Remove Advanced Tag Defaults and Tag Definitions

Only remove the `ExpirationDate` tag default and tag definition if they were created solely for the advanced labs and are not used by other teams, policies, automation, or resources.

<details>
<summary>Option 1: Delete Resources via UI</summary>

1. Navigate to **Identity & Security > Compartments**, open your workshop compartment, and select **Tag Defaults**.

2. Delete the `LLTagNamespace.ExpirationDate` tag default created in Lab 5.

3. Navigate to **Governance & Administration > Tag Namespaces** and open `LLTagNamespace`.

4. Retire the `ExpirationDate` tag definition if it is no longer needed.

5. Leave `LLTagNamespace`, `Environment`, and `CostCenter` in place unless you are also cleaning up the core workshop resources.

</details>

<details>
<summary>Option 2: Delete via CLI</summary>

Run the following commands in Cloud Shell. This task intentionally removes only `ExpirationDate`; it does not retire the core namespace or the core `Environment` and `CostCenter` tags.

```bash
<copy>
echo "Deleting the advanced workshop ExpirationDate tag default..."
tag_default_ids=$(
  oci iam tag-default list \
    --compartment-id "$compartment_ocid" \
    --all \
    --query "data[?\"tag-definition-name\"=='ExpirationDate' && \"lifecycle-state\"!='DELETED'].id | join(' ', @)" \
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
  tag_state=$(
    oci iam tag get \
      --tag-name ExpirationDate \
      --tag-namespace-id "$tag_namespace_id" \
      --query "data.\"lifecycle-state\"" \
      --raw-output 2>/dev/null || true
  )

  if [ -n "$tag_state" ] && [ "$tag_state" != "RETIRED" ]; then
    echo "Retiring ExpirationDate tag"
    oci iam tag retire \
      --tag-name ExpirationDate \
      --tag-namespace-id "$tag_namespace_id"
  fi
fi
</copy>
```

</details>

## Learn More

- [Deleting functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsdeleting.htm)
- [Managing tag defaults](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/managingtagdefaults.htm)
- [Using tags to manage access](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/managingaccesswithtags.htm)
- [Overview of Resource Scheduler](https://docs.oracle.com/en-us/iaas/Content/resource-scheduler/home.htm)

## Acknowledgements

- **Author** - Wynne Yang
- **Contributors** - Daniel Hart, Deion Locklear, Eli Schilling, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
