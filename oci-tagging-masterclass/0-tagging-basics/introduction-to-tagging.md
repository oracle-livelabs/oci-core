# Introduction to Tagging in Oracle Cloud Infrastructure

### Objectives

In this lab, you will:
- Describe the difference between free-form tags and defined tags.
- Explain how tag namespaces, tag defaults, tag variables, and cost-tracking tags work together.
- Identify the OCI services used throughout this workshop for cost management, access control, and automation.


## The Challenge

As organizations scale their cloud footprint, the number of resources in a tenancy can grow rapidly. Without a structured approach to organization, teams often struggle to answer basic but critical questions: Who created this resource? Which project does it belong to? Is this instance still needed, or is it generating unnecessary cost? Which department should be billed for this storage bucket?

These challenges are not unique to any one organization — they are a natural consequence of cloud adoption at scale. Left unaddressed, they lead to cost overruns, security blind spots, and operational inefficiency. A well-designed tagging strategy is one of the most effective tools available to solve these problems.

### What is Tagging?

A tag is a key-value pair that can be attached to a cloud resource. While simple in concept, tags unlock a broad range of capabilities when applied consistently. In Oracle Cloud Infrastructure (OCI), tagging is built directly into the Identity and Access Management (IAM) service, making it a foundational element of resource governance. By tagging resources systematically, organizations can enable cost tracking, resource discovery and filtering, access control, and automated lifecycle management — all from a single, unified framework.

### Key Concepts

#### Tag Types

OCI supports two types of tags:

- **Free-form tags** are the simplest form. They consist of a key and a value and can be applied to any resource without any prior configuration. Free-form tags are flexible and easy to use, but they do not enforce consistency — any user can define any key-value combination.

- **Defined tags** are created and managed by an administrator. They provide a greater level of control by requiring a **tag namespace** (a logical container for related tag keys), a **tag key**, and a **tag value**. The tag value can be free-form text or constrained to a pre-defined list of acceptable values. This structure helps organizations enforce naming conventions and ensure data quality across the tenancy.

#### Tag Namespace

A tag namespace is a container for a set of related tag key definitions. Namespaces help organize tags logically — for example, an organization might create a namespace called `CostCenter` or `ProjectTracking`. Namespaces also serve as a boundary for IAM policy, allowing administrators to control who can create, apply, and manage specific sets of tags.

#### Cost Tracking Tags

Cost tracking is a feature of defined tags. Administrators can enable up to 10 tag key definitions for cost-tracking purposes. When a cost-tracking tag is applied to a resource, the tag key and value will appear in billing reports and the Cost Analysis tool in the OCI Console. This allows organizations to attribute cloud spending to specific departments, projects, environments, or any other dimension that aligns with their financial reporting needs.

#### Tag Defaults

Tag defaults enable administrators to automatically apply tags to all resources created within a specific compartment. When a user creates a resource in that compartment, the default tags are applied without any action required from the user. This is a powerful mechanism for enforcing tagging policies — for example, ensuring every resource in a production compartment is automatically tagged with `Environment: Production`.

#### Tag Variables

Tag variables add dynamic behavior to tag defaults. Rather than assigning a static value, administrators can use a variable expression that resolves at the time the resource is created. For example, the variable `${iam.principal.name}` will resolve to the name of the user who created the resource. This enables automatic attribution without relying on users to manually tag their own resources. Other useful variables include `${iam.principal.type}` and `${oci.datetime}`.

### Supporting OCI Services

Tags become even more powerful when combined with other OCI services. This workshop will make use of the following:

- **Budgets:** A cost management feature that enables administrators to set spending limits based on compartments or cost-tracking tags. Budgets can generate automated alerts when spending approaches or exceeds a defined threshold, helping to prevent unexpected overages.

- **Events Service:** Enables administrators to create automation rules that are triggered by state changes to resources within the tenancy — for example, when a compute instance is launched or a tag is modified. Events can trigger notifications, functions, or streaming pipelines.

- **Functions:** OCI Functions is a serverless, event-driven compute platform. Functions can be triggered by the Events service or executed on a schedule using the OCI Scheduler. In the context of tagging, functions can be used to enforce compliance — for example, by automatically stopping or terminating resources that are missing required tags.

- **OCI Command Line Interface (CLI):** The OCI CLI provides programmatic access to OCI services from the command line. It is particularly useful for performing bulk operations on resources, such as applying or updating tags across a large number of instances simultaneously.

## About This Workshop

This workshop provides a hands-on framework for understanding the capabilities enabled through a comprehensive tagging strategy in OCI. Through a series of guided labs, you will move from foundational concepts — creating and applying tags — to advanced use cases like tag-based access control and serverless enforcement of tagging policies.

By the end of this workshop, you will be equipped to:

- Improve visibility into who owns and manages cloud resources.
- Reduce the administrative overhead of managing large resource pools.
- Simplify cost allocation and financial reporting.
- Control and mitigate overspending through budgets and alerts.
- Enforce governance standards through automated tagging and compliance checks.

### Workshop Labs

| Lab | Title | Estimated Time | Key Services |
|-----|-------|---------------|-------------|
| 1 | Working with Free-Form Tags, Defined Tags, and Tag Defaults | 20 minutes | IAM, Tagging |
| 2 | Making Bulk Tag Changes with the OCI CLI | 20 minutes | OCI CLI, Tagging |
| 3 | Cost Management with Tags and Budgets | 20 minutes | Budgets, Cost Analysis, Tagging |
| 4 | Tag-Based Access Control with IAM Policies | 20 minutes | IAM Policies, Tagging |
| 5 | Automated Resource Updates with OCI Functions | 25 minutes | Functions, Events, Tagging |
| 6 | Enforcing Tagging Compliance with OCI Functions | 30 minutes | Functions, Events, Tagging |

**Estimated Total Workshop Time:** 2 hours 15 minutes

### Prerequisites

- Access to an Oracle Cloud Infrastructure tenancy with administrator privileges (or a pre-configured LiveLabs sandbox environment).
- Basic familiarity with the OCI Console, including navigation and compartment structure.
- A compartment designated for workshop activities (the lab instructions will guide you through creating resources within this compartment).
- Access to OCI Cloud Shell or a local terminal with the OCI CLI installed and configured.
- No prior experience with OCI tagging is required — the workshop starts from the fundamentals.

## Learn More

- [Overview of Tagging](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)
- [Tagging Best Practices](https://www.ateam-oracle.com/oracle-cloud-infrastructure-tagging-best-practices-enable-mandatory-tagging-for-compartments)
- [Tagging in Oracle Cloud Infrastructure](https://medium.com/@johnny.cree/tagging-in-oracle-cloud-infrastructure-c279c350f692)
- [OCI Command Line Interface](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cliconcepts.htm)
- [OCI Budgets](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/budgetsoverview.htm)
- [OCI Events](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventsoverview.htm)
- [OCI Functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Concepts/functionsoverview.htm)

## Acknowledgements

- **Author** - Eli Schilling
- **Contributors** - Daniel Hart, Deion Locklear, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
