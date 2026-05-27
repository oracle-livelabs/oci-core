# Lab 3b: Cloud Cost Management and Tagging for Finance Teams

## Introduction

Cloud computing introduces a fundamentally different cost model from traditional on-premises infrastructure. Instead of fixed capital expenditures planned months in advance, cloud spending is primarily operational — variable, usage-based, and distributed across teams, projects, and environments. For finance and accounting professionals, this shift demands new tools and practices to maintain the visibility, accountability, and control that sound financial management requires.

Oracle Cloud Infrastructure (OCI) provides a suite of native cost management capabilities that, when combined with a well-designed tagging strategy, give finance teams the ability to track spending at a granular level, attribute costs to the correct business owners, enforce spending limits, and generate the data needed for chargeback, showback, and financial reporting.

This activity is designed for finance line-of-business users. It does not require hands-on technical work in the OCI Console (though optional exercises are included for those with Console access). Instead, it focuses on building a working understanding of the tools, strategies, and workflows that connect cloud resource tagging to financial outcomes.

**Estimated Time:** 25 minutes

### Objectives

In this activity, you will learn:

- How cost tracking tags work and why they are the foundation of cloud financial management in OCI.
- How to use Cost Analysis to explore, filter, and visualize cloud spending by department, project, environment, or any other business dimension.
- How cost reports (including FOCUS-compliant reports) support chargeback, showback, and cross-departmental billing.
- How OCI Budgets provide threshold-based spending alerts to prevent overages before they happen.
- How compartment quotas create hard limits on resource consumption and work alongside budgets to minimize overspend.
- How the OCI Cost Estimator supports forecasting and capacity planning.

### Prerequisites

This activity assumes you have:

- Read-only access (at minimum) to the OCI Console, or access to screenshots and walkthroughs provided by your cloud operations team.
- A basic understanding of your organization's compartment structure in OCI (which compartments map to which teams or projects).
- Familiarity with general accounting concepts such as cost allocation, chargeback, and budget variance analysis.

## Topic 1: How Tags Drive Financial Visibility

### The Problem Tags Solve

In a cloud environment, dozens or hundreds of resources — compute instances, databases, storage buckets, network components — may be running at any given time. Each of these resources generates cost. Without a structured way to label and categorize them, answering even basic financial questions becomes difficult: Which department is responsible for this database? Is this compute instance part of a production workload or a development experiment? How much did the marketing team spend on cloud services last quarter?

Tags solve this problem by attaching metadata — key-value pairs — to every cloud resource. When tags are designed with financial reporting in mind, they transform raw usage data into meaningful business intelligence.

### Cost Tracking Tags in OCI

OCI supports a special category of defined tags called **cost tracking tags**. These are tags that an administrator has explicitly enabled for cost tracking purposes. When a cost tracking tag is applied to a resource, the tag key and value appear directly in billing data, cost reports, and the Cost Analysis tool.

Key characteristics of cost tracking tags:

- Up to **10 tag key definitions** can be enabled for cost tracking across a tenancy. This is a hard limit, so it is important to choose cost tracking dimensions carefully.
- Cost tracking tags must be **defined tags** (not free-form tags). This means they are created within a tag namespace by an administrator and can be constrained to a pre-defined list of acceptable values.
- Once a cost tracking tag is applied to a resource, the tag's key and value are included in **cost reports** and available as a **grouping dimension** and **filter** in Cost Analysis.

### Designing Tags for Financial Reporting

The most effective tagging strategies are designed collaboratively between finance and cloud operations teams. Finance teams understand the reporting dimensions that matter — cost centers, project codes, business units, environments — while cloud operations teams understand how to implement and enforce them at the resource level.

Common tag keys used for financial reporting include:

| Tag Key | Example Values | Purpose |
|---------|---------------|---------|
| `CostCenter` | `CC-1001`, `CC-2050`, `CC-3200` | Maps cloud spend to existing cost center codes in your ERP or general ledger. |
| `Project` | `ProjectAlpha`, `DataMigration-2026` | Attributes spend to a specific project for capital vs. operating expense tracking. |
| `Environment` | `Production`, `Development`, `Test`, `Sandbox` | Distinguishes production workloads from non-production, which often have different budget thresholds and approval workflows. |
| `Department` | `Finance`, `Engineering`, `Marketing` | Enables departmental showback or chargeback reporting. |
| `Owner` | `jane.smith@company.com` | Identifies who is accountable for a resource and its associated cost. |

> **Key Takeaway:** Cost tracking tags are the bridge between cloud resource consumption and your organization's financial reporting structure. If the tags are not applied consistently, the financial data downstream will have gaps. This is why tag defaults (which automatically apply tags when resources are created) and tag enforcement (covered in other labs) are so important.

## Topic 2: Cost Analysis

### What It Does

Cost Analysis is the primary tool in the OCI Console for exploring and visualizing cloud spending. It provides an interactive dashboard where you can query your spending data across multiple dimensions, generate charts, and download results.

You can access Cost Analysis from the OCI Console by navigating to **Billing & Cost Management > Cost Analysis**.

### How Tags Enhance Cost Analysis

Without tags, Cost Analysis can only group and filter spending by service type, compartment, or resource OCID — which are technical dimensions that may not align with how finance teams think about costs. Cost tracking tags add business-relevant dimensions to the data.

With cost tracking tags in place, you can:

- **Group by tag:** View a chart of monthly spending grouped by `CostCenter`, showing exactly how much each cost center consumed.
- **Filter by tag:** Isolate spending for a single department (e.g., `Department = Engineering`) to analyze its consumption trends over time.
- **Combine dimensions:** Group by `Service` and filter by `Environment = Production` to see which OCI services are driving production costs.

### Working with Cost Analysis

When using Cost Analysis, the key query fields are:

- **Date range and granularity:** Select the time period and whether to view data at daily, monthly, or cumulative granularity.
- **Report type:** Choose between costs, credits, or usage quantity.
- **Grouping dimensions:** The dimension by which data is broken down in the chart. Common grouping dimensions include Service, Compartment, Tag (your cost tracking tags), and Resource OCID.
- **Filters:** Narrow the results to a specific service, compartment, tag value, or other criteria.

Cost Analysis also allows you to save frequently used queries as **saved reports**, so that standard financial views (e.g., "Monthly Spend by Cost Center" or "Production Environment Costs by Service") can be recalled without rebuilding the query each time.

> **Tip for Finance Teams:** If you want to export data for further analysis in Excel or a BI tool, use the **Download Table as CSV** option below the chart. This exports the underlying data table, which includes all the tag-based grouping and filtering you applied.

## Topic 3: Cost Reports and Chargeback

### Cost Reports

Cost reports are CSV files generated automatically by OCI that provide a detailed, resource-level breakdown of usage and cost. They include fields such as the service name, resource OCID, usage quantity, unit price, and total cost — as well as columns for any cost tracking tags applied to the resource.

OCI generates two types of cost reports:

- **OCI Proprietary Format:** The original cost report format with detailed fields specific to OCI's billing model.
- **FOCUS Format:** Reports conforming to the **FinOps Open Cost & Usage Specification (FOCUS)**, an open-source, vendor-neutral schema for cloud billing data maintained by the FinOps Foundation. FOCUS reports use standardized column names and definitions, making it easier to integrate OCI billing data with multi-cloud cost management platforms or internal BI tools that support the FOCUS schema.

> **Note:** OCI deprecated standalone usage reports on January 31, 2025. Cost reports (in both OCI proprietary and FOCUS formats) now serve as the unified source for both usage and cost data.

Cost reports can be accessed from **Billing & Cost Management > Cost Reports** in the OCI Console. They are stored as compressed CSV files organized by date.

### Using Cost Reports for Chargeback and Cross-Charge

Chargeback (billing internal departments for their actual cloud consumption) and showback (reporting consumption without formal billing) are among the most common use cases for cost tracking tags. The workflow typically looks like this:

1. **Tag resources consistently.** Every resource that generates cost should carry the tag keys that map to your chargeback dimensions (e.g., `CostCenter`, `Department`). Tag defaults and enforcement automation help ensure coverage.

2. **Download or programmatically retrieve cost reports.** Cost reports include a `tags/` column for each cost tracking tag. For example, if `CostCenter` is a cost tracking tag, the cost report will include a column named `tags/YourNamespace.CostCenter` containing the tag value for each line item.

3. **Aggregate and distribute.** Using Excel, a BI tool, or an automated pipeline, aggregate the cost report data by your chargeback dimension (e.g., sum all costs where `CostCenter = CC-1001`). The resulting totals can be used to generate internal invoices, journal entries, or showback reports.

4. **Reconcile against budgets.** Compare actual chargeback amounts against the budgets set for each department or project to identify variances.

> **Key Consideration:** Resources that are missing cost tracking tags will appear in cost reports with blank tag columns. These "untagged" costs represent a gap in your chargeback model. Monitoring the percentage of untagged spending is an important metric for governance — it tells you how complete your tagging coverage is and where enforcement needs attention.

## Topic 4: Budgets and Alerts

### What Budgets Do

OCI Budgets allow you to set spending thresholds and receive automated email alerts when actual or forecasted spending approaches or exceeds those thresholds. Budgets are a soft control — they do not prevent resources from being created or running, but they provide early warning so that appropriate action can be taken.

### How Budgets Work

Budgets can be scoped in two ways:

- **Compartment-based budgets:** Track all spending within a specific compartment and its child compartments.
- **Tag-based budgets:** Track all spending associated with a specific cost tracking tag key and value, regardless of which compartment the resources reside in.

Tag-based budgets are particularly powerful for finance teams because they align directly with financial reporting dimensions. For example, you can create a budget that tracks all resources tagged with `CostCenter = CC-2050` and sets a monthly threshold of $15,000 — even if those resources are spread across multiple compartments.

### Budget Alerts

Each budget can have one or more alert rules. An alert rule specifies:

- **Threshold type:** Whether the alert is based on actual spend (what has already been consumed) or forecasted spend (what OCI projects you will spend by the end of the period based on current trends).
- **Threshold value:** Either a percentage of the budget amount (e.g., alert at 80%) or an absolute dollar amount (e.g., alert at $12,000).
- **Recipients:** Email addresses that will receive the alert notification.
- **Custom message:** An optional message body included in the alert email (useful for including instructions, such as "Contact Cloud Ops to review resource utilization").

Budgets are evaluated approximately every 24 hours. When an alert threshold is crossed, OCI sends an email to the configured recipients.

### Budget Strategy Recommendations

For finance teams, a common budget strategy includes multiple alert tiers:

| Alert | Threshold | Purpose |
|-------|-----------|---------|
| Early warning | 50% of budget (forecasted) | Gives teams time to adjust if spending trends are higher than expected. |
| Caution | 80% of budget (actual) | Signals that the budget is being consumed and close review is needed. |
| Critical | 100% of budget (actual) | Indicates the budget has been reached. May trigger a review process or escalation. |

> **Tip:** Budgets can also trigger events through the OCI Events service. Advanced teams can use this integration to automatically invoke a serverless function when a budget threshold is crossed — for example, to send a Slack notification, log a ticket, or even restrict resource creation by modifying compartment quotas programmatically.

## Topic 5: Compartment Quotas — Hard Limits on Spending

### Why Budgets Are Not Enough

Budgets provide alerts, but they do not prevent spending from continuing beyond the threshold. If a team does not act on a budget alert, costs will continue to accrue. For situations where a hard cap on resource consumption is required, OCI provides **compartment quotas**.

### What Compartment Quotas Do

Compartment quotas are policy statements that set maximum limits on the number or size of resources that can be created within a specific compartment. Unlike budgets, quotas are enforced at the point of resource provisioning — if a user attempts to create a resource that would exceed the quota, OCI denies the request.

Quotas are defined using a simple declarative policy language with three statement types:

- **`set`** — Sets the maximum number of a resource in a compartment.
- **`unset`** — Removes the quota and reverts to the default service limit.
- **`zero`** — Completely blocks a resource type in a compartment.

### Example Quota Policies

Limit the development team's compartment to 100 compute cores:

```
set compute-core quota standard-e4-core-count to 100 in compartment Development
```

Block all database provisioning in a sandbox compartment:

```
zero database quotas in compartment Sandbox
```

Allocate different resource levels by environment:

```
set compute-core quota standard-e4-core-count to 500 in compartment Production
set compute-core quota standard-e4-core-count to 200 in compartment Testing
set compute-core quota standard-e4-core-count to 100 in compartment Development
zero compute-core quotas in compartment Archived
```

### How Quotas and Budgets Work Together

Quotas and budgets serve complementary purposes. Used together, they form a layered cost control framework:

| Control | Type | Scope | Enforcement |
|---------|------|-------|-------------|
| **Budget** | Soft limit | Compartment or tag | Alert only — does not prevent resource creation. |
| **Quota** | Hard limit | Compartment | Enforced — OCI denies requests that exceed the quota. |

A practical approach is to set quotas at levels that represent the maximum acceptable resource footprint for a compartment, and then set budgets at lower thresholds to provide advance notice before the quota ceiling is reached. For example, a department with a quota of 200 compute cores might have a budget alert set to fire when spending hits 80% of the projected cost for 200 cores.

> **Key Consideration for Finance Teams:** Quotas are expressed in resource units (e.g., number of cores, terabytes of storage), not in dollar amounts. Translating quota limits into estimated cost requires knowledge of the unit pricing for each resource type. The OCI Cost Estimator (covered in the next topic) can help with this translation.

## Topic 6: Forecasting with the OCI Cost Estimator

### What It Does

The OCI Cost Estimator is a web-based planning tool that allows you to model the cost of running workloads in OCI before committing to them. It is available at [oracle.com/cloud/costestimator.html](https://www.oracle.com/cloud/costestimator.html) and does not require an OCI account to use.

### How Finance Teams Can Use It

The Cost Estimator is useful in several financial planning scenarios:

- **New project scoping:** Before a new project begins, model the expected infrastructure (compute instances, databases, storage, networking) to generate a monthly cost estimate. This estimate can inform budget requests, approval workflows, and project financial plans.
- **Capacity planning:** If a department is planning to increase its compute footprint, use the estimator to model the incremental cost and validate that it falls within the department's approved budget.
- **What-if analysis:** Compare the cost of different architecture choices — for example, the cost difference between using a larger compute shape versus distributing work across multiple smaller shapes.
- **Quota-to-cost translation:** If cloud operations has set a compartment quota of 200 compute cores, use the estimator to calculate what the maximum monthly cost would be if all 200 cores were fully utilized. This helps finance teams set meaningful budget thresholds.

### Key Features

- **Service-level configuration:** Select specific OCI services (Compute, Database, Object Storage, etc.) and configure instance types, shapes, storage amounts, and utilization hours.
- **Reference architectures:** Pre-built templates for common workload patterns that automatically populate the estimator with a realistic set of resources.
- **Rate card integration:** If you have an OCI account, the estimator can use your organization's negotiated pricing rather than list prices, providing more accurate projections.
- **Save and share:** Estimates can be saved, exported, and shared with stakeholders for review and approval.

## Topic 7: Putting It All Together — A Finance Team's Cost Governance Workflow

The tools and concepts covered in this activity work together as a cohesive cost governance framework. Here is how they connect in practice:

**1. Define financial dimensions with tags.** Work with your cloud operations team to establish cost tracking tags that map to your reporting structure (cost centers, projects, departments). Ensure that tag defaults are configured to automatically apply these tags to new resources, and that enforcement mechanisms are in place to catch gaps.

**2. Set budgets aligned to your financial plan.** Create tag-based budgets in OCI that correspond to each cost center or department's approved cloud budget. Configure alert rules at multiple thresholds (e.g., 50% forecast, 80% actual, 100% actual) with notifications routed to the appropriate finance and operations contacts.

**3. Establish quotas as guardrails.** Work with cloud operations to define compartment quotas that cap resource consumption at levels consistent with approved budgets. Quotas prevent uncontrolled scaling and ensure that spending cannot exceed a predictable ceiling.

**4. Monitor with Cost Analysis.** Use Cost Analysis on a regular cadence (weekly or monthly) to review spending by cost center, department, or project. Save standard queries as reports for consistency across reporting periods.

**5. Generate chargeback with cost reports.** Download FOCUS-compliant or OCI proprietary cost reports and aggregate them by cost tracking tag to produce departmental chargeback or showback reports. Reconcile these reports against budgets to identify variances.

**6. Forecast with the Cost Estimator.** Use the Cost Estimator to model upcoming projects and infrastructure changes. Feed these estimates into your budgeting process to ensure budgets are set at realistic levels before spending begins.

## Summary

| Tool / Concept | What It Does | Why Finance Teams Care |
|----------------|-------------|----------------------|
| **Cost Tracking Tags** | Attach financial metadata to cloud resources. | Enables cost attribution to cost centers, projects, and departments. |
| **Cost Analysis** | Interactive dashboard to explore and visualize spending. | Provides on-demand visibility into spending trends by any tagged dimension. |
| **Cost Reports (FOCUS)** | Detailed, downloadable CSV billing data at resource level. | Powers chargeback, showback, and integration with BI and ERP systems. |
| **Budgets & Alerts** | Spending thresholds with automated email notifications. | Provides early warning before budgets are exceeded. |
| **Compartment Quotas** | Hard limits on resource provisioning per compartment. | Prevents uncontrolled resource creation and associated cost. |
| **Cost Estimator** | Pre-commitment cost modeling for new workloads. | Supports budget planning, what-if analysis, and project approvals. |

## Learn More

- [OCI Cost Analysis](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costanalysisoverview.htm)
- [OCI Cost Reports](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costusagereportsoverview.htm)
- [OCI Budgets](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/budgetsoverview.htm)
- [Overview of Compartment Quotas](https://docs.oracle.com/en-us/iaas/Content/Quotas/Concepts/resourcequotas.htm)
- [OCI Cost Estimator](https://www.oracle.com/cloud/costestimator.html)
- [Tracking Costs with OCI Tagging (Oracle Blog)](https://blogs.oracle.com/cloud-infrastructure/tracking-costs-with-oracle-cloud-infrastructure-tagging)
- [Track and Manage Usage and Cost (OCI Well-Architected Framework)](https://docs.oracle.com/en/solutions/oci-best-practices/track-and-manage-usage-and-cost1.html)
- [Use Quotas for Better Control and More Effective Cost Management (Oracle Blog)](https://blogs.oracle.com/cloud-infrastructure/use-quotas-for-better-control-and-more-effective-cost-management-in-the-cloud)

## Acknowledgements

- **Author** - Eli Schilling
- **Contributors** - Daniel Hart, Deion Locklear, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
