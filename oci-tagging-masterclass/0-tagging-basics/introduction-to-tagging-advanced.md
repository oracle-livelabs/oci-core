# Tagging Review for OCI Tagging Advanced Features

### Objectives

In this workshop, you will:

- Review the tagging foundation required for advanced governance patterns.
- Use defined tags in IAM policy conditions.
- Deploy OCI Functions that read, update, and enforce tag values.
- Use Resource Scheduler and Events to automate tag-governance workflows.
- Safely test tag-compliance enforcement in an isolated compartment.

## The Challenge

Creating tags is only the first step. Once tags are consistently applied, they become a control plane for governance: policies can evaluate tag values, automation can update tag defaults, and compliance functions can identify or stop resources that drift from standards.

The advanced labs focus on these higher-level patterns. You will use the tags created in the core workshop, then layer on IAM, Functions, Resource Scheduler, and Events to demonstrate how tagging can drive access control and operational automation.

### Tagging Foundation Review

Before starting the advanced labs, make sure you have the following tagging resources from the core workshop or an equivalent setup in your own tenancy:

- A defined tag namespace, such as `LLTagNamespace`.
- A defined tag key named `Environment`.
- A defined tag key named `CostCenter`.
- A bucket tagged with `LLTagNamespace.Environment = Prod` for the tag-based access-control lab.
- A workshop compartment where Functions applications and temporary test resources can be created.

The advanced labs also introduce an `ExpirationDate` tag key and tag default. This tag is used by the automation lab to demonstrate how a function can keep a default value current over time.

### Supporting OCI Services

This advanced workshop uses the following OCI services:

- **IAM Policies and Dynamic Groups:** Authorize users and functions based on group membership, resource type, compartment, and tag values.
- **Object Storage:** Reuse a tagged bucket to discuss tag-based access-control behavior.
- **OCI Functions and Fn CLI:** Deploy Python functions that update tag defaults and enforce tag compliance.
- **Resource Scheduler:** Invoke a function on a recurring schedule.
- **Events Service:** Invoke a function when a compute instance launch event occurs.
- **Compute:** Create isolated test instances for compliance validation.

## About This Workshop

This workshop assumes you already understand OCI defined tags, tag namespaces, tag defaults, and basic OCI CLI usage. It focuses on what those tagging primitives unlock after they exist.

By the end of this workshop, you will be able to:

- Explain how tag values can be used in IAM policy conditions.
- Review and adapt a tag-based deny policy for a tenancy where deny policies are enabled.
- Deploy a function that updates an `ExpirationDate` tag default.
- Schedule a function with OCI Resource Scheduler.
- Deploy event-driven and scheduled compliance functions that stop non-compliant compute instances in an isolated test compartment.

### Workshop Labs

| Lab | Title | Estimated Time | Key Services |
|-----|-------|---------------|-------------|
| 4 | Tag-Based Access Control with IAM Policies | 25-30 minutes | IAM Policies, Tagging, Object Storage |
| 5 | Automated Resource Updates with OCI Functions | 25 minutes | Functions, Resource Scheduler, Tagging |
| 6 | Enforcing Tagging Compliance with OCI Functions | 30 minutes | Functions, Events, Compute, Tagging |

**Estimated Total Workshop Time:** 1 hour 20 minutes

### Prerequisites

- Completion of the OCI Tagging Core Capabilities workshop, or equivalent tagging resources already created in your tenancy.
- Administrative access to IAM, Functions, Events, Resource Scheduler, and the workshop compartment.
- Access to OCI Cloud Shell with OCI CLI and Fn CLI available.
- A VCN and subnet in the workshop compartment for OCI Functions applications.
- An isolated test compartment for Lab 6 that contains only compute instances you are willing to stop or terminate during validation.
- Awareness that IAM deny policies are an opt-in tenancy feature. Do not enable deny policies in a shared or production tenancy just to complete this workshop.

## Learn More

- [Using Tags to Manage Access](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/managingaccesswithtags.htm)
- [Deny Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/policysyntax/denypolicies.htm)
- [OCI Functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Concepts/functionsoverview.htm)
- [Overview of Resource Scheduler](https://docs.oracle.com/en-us/iaas/Content/resource-scheduler/home.htm)
- [OCI Events](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventsoverview.htm)

## Acknowledgements

- **Author** - Eli Schilling
- **Contributors** - Daniel Hart, Deion Locklear, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
