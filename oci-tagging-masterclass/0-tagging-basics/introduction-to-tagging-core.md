# Introduction to OCI Tagging Core Capabilities

### Objectives

In this workshop, you will:

- Describe the difference between free-form tags and defined tags.
- Create a tag namespace, tag key definitions, and tag defaults.
- Use OCI CLI queries to find and update tagged resources in bulk.
- Use cost-tracking tags with Budgets, Cost Analysis, and cost reports.
- Explain how budgets and quotas support a practical cloud cost-management strategy.

## The Challenge

As organizations scale their cloud footprint, the number of resources in a tenancy can grow rapidly. Without a structured tagging strategy, teams often struggle to answer basic but critical questions: Who created this resource? Which project does it belong to? Which department should be billed for it? Is it still needed?

The core tagging labs focus on building that foundation. You will create governed tags, apply them automatically with tag defaults, use the OCI CLI to work with tagged resources at scale, and connect tags to financial visibility through budgets and cost-management tools.

### What is Tagging?

A tag is a key-value pair that can be attached to a cloud resource. In Oracle Cloud Infrastructure (OCI), tagging is part of Identity and Access Management (IAM), which makes it a foundational tool for resource governance, cost allocation, search, automation, and access control.

### Key Concepts

#### Tag Types

OCI supports two types of tags:

- **Free-form tags** are simple key-value pairs that can be applied without prior setup. They are flexible, but they do not enforce consistent keys or values.
- **Defined tags** are created and managed by administrators. They use a tag namespace, tag key, and tag value. Values can be free-form or constrained to a predefined list.

#### Tag Namespace

A tag namespace is a container for related tag key definitions. In these labs, you will create a namespace called `LLTagNamespace` and use it to hold the workshop tag keys.

#### Cost Tracking Tags

Cost tracking is a feature of defined tags. When a tag key is enabled for cost tracking, its values can appear in billing reports and can be used by Cost Analysis and Budgets. This allows teams to align cloud spend with departments, projects, environments, or other financial dimensions.

#### Tag Defaults

Tag defaults automatically apply defined tags to new resources created in a compartment. This helps enforce a baseline tagging standard without requiring every user to remember every tag.

#### Tag Variables

Tag variables add dynamic values to tag defaults. For example, `${iam.principal.name}` resolves to the name of the user who created a resource. Variables are useful for attribution and operational reporting.

### Supporting OCI Services

This core workshop uses the following OCI services:

- **IAM and Tagging:** Create namespaces, tag definitions, tag defaults, and cost-tracking tags.
- **Object Storage, Compute, and Block Volume:** Create sample resources that demonstrate tag inheritance and bulk tag updates.
- **OCI Command Line Interface (CLI):** Query and update resources programmatically.
- **Budgets and Cost Analysis:** Connect tags to cost visibility, reporting, and budget alerts.

## About This Workshop

This workshop covers the core capabilities every OCI tagging strategy needs before moving into access control or automation. You will start with foundational tag design, then use tags to support operational bulk changes and cost-management workflows.

By the end of this workshop, you will be able to:

- Create defined tags and tag defaults.
- Validate that new resources inherit required tags.
- Use OCI CLI `--query` filters to find and update tagged resources.
- Explain how cost-tracking tags support budgets, cost analysis, showback, and chargeback.
- Describe where budgets and quotas fit into a broader cost-governance model.

### Workshop Labs

| Lab | Title | Estimated Time | Key Services |
|-----|-------|---------------|-------------|
| 1 | Working with Free-Form Tags, Defined Tags, and Tag Defaults | 20-25 minutes | IAM, Tagging, Object Storage |
| 2 | Making Bulk Tag Changes with the OCI CLI | 30 minutes | OCI CLI, Compute, Block Volume, Tagging |
| 3a | Cost Management with Tags and Budgets | 20 minutes | Budgets, Cost Analysis, Tagging |
| 3b | Cost Management Strategies | 25 minutes | Budgets, Quotas, Cost Analysis |

**Estimated Total Workshop Time:** 1 hour 35 minutes

### Prerequisites

- Access to an Oracle Cloud Infrastructure tenancy with administrator privileges or a pre-configured LiveLabs sandbox environment.
- Basic familiarity with the OCI Console, including navigation and compartment structure.
- A compartment designated for workshop activities.
- Access to OCI Cloud Shell or a local terminal with the OCI CLI installed and configured.
- No prior experience with OCI tagging is required.

## Learn More

- [Overview of Tagging](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm)
- [Managing Tag Defaults](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/managingtagdefaults.htm)
- [OCI Command Line Interface](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cliconcepts.htm)
- [OCI Budgets](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/budgetsoverview.htm)
- [Overview of Compartment Quotas](https://docs.oracle.com/en-us/iaas/Content/Quotas/Concepts/resourcequotas.htm)

## Acknowledgements

- **Author** - Eli Schilling
- **Contributors** - Daniel Hart, Deion Locklear, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
