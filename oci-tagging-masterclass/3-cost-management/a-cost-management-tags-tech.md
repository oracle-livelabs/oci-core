# Lab 3a (Tech): Working with Cost Management and Tags

## Introduction

In this lab, we are going to explain how to use cost-tracking tags, create a budget and budget alert rule, build and export a cost analysis report, and explain what FOCUS cost reports cover.

**Estimated Time:** 20 minutes

### Objectives

In this lab, you will:

- Create a budget
- Create a budget alert rule
- Create a cost analysis report
- Download a FOCUS Report

### Prerequisites

This lab assumes you have:
- completed the previous labs
- permissions to interact with tools through Identity and Access Management policies

## Task 1: Ensure you have created a cost-tracking tag
In this task, we are readdressing cost tracking tags. Please refer to the cost-tracking tag previously created in Lab 1. This cost-tracking tag is used for tracking resource costs that span multiple compartments or belong to different Cost Centers. Cost-tracking tags are also used for billing purposes. However, you can use *any defined tag* as a scope for your budgets, Cost Analysis, and Cost Reports.


## Task 2a: Budgets using OCI Console
In this task we are going to create a budget based on a cost tracking tag and create a budget alert rule for the budget.
1. Click the **Navigation Menu** on the upper left. Navigate to **Billing & Cost Management**, and select **Budgets**

     ![Screenshot showing how to navigate to Budgets](./images/navigate-to-budgets.png)

2. Make sure you are in the correct region on the upper left. You can only interact with budgets in your Home Region.

     ![Screenshot showing how to select your home region](./images/use-correct-region.png)

3. Click Create Budget.

     ![Screenshot showing how to create a budget](./images/create-budget.png)

4. Select Tags as the Budget Scope.

     ![Screenshot showing how to select tags as the budget scope](./images/budget-tag-scope.png)

5. Select the Tag Namespace and Tag Key for the cost-tracking tag you created earlier. Select the value you would like to use for the budget scope.

     ![Screenshot showing how to select cost tracking tag created in lab 1](./images/budget-tag-key-input.png)

6. Select Monthly for Schedule.

     ![Screenshot showing how to select Monthly schedule type](./images/budget-monthly.png)

7. Enter $1 as the Budget Amount and 1 as the day of the month for recurring budget processing.

     ![Screenshot showing how to input budget amount and day of the month for budget processing](./images/budget-set-amount.png)

    The budget will now evaluate consumption costs for resources under your cost-tracking tag every month on the 1st.
    We will now create a Budget Alert Rule for this budget. You can also create Budget Alert Rules after you have created your budget.

8. Select Actual Spend for the Threshold Metric for the Budget Alert Rule.

     ![Screenshot showing how to select Actual Spend for the budget alert rule](./images/budget-alert-threshold.png)

9. Select Percentage of Budget as Threshold Type.

     ![Screenshot showing how to select percentage of budget for threshold type](./images/budget-alert-threshold-type.png)

10. Input 1% as the threshold % amount.

     ![Screenshot showing how to input percentage amount for threshold amount](./images/budget-alert-threshold-amount.png)

11. Input email recipient for the budget alert.

     ![Screenshot showing how to input email recipient for budget alert](./images/budget-alert-recipient.png)

12. Input email message:

      ```text
      <copy>
      Hello, you are receiving this message because your budget has been reached. 
      </copy>
      ```

     ![Screenshot showing how to input email message for budget alert rule](./images/budget-alert-message.png)

13. Click Create. 

     ![Screenshot showing how to click create for budget](./images/budget-finish-create.png)


> **Note:** Your budget alert notification may not be delivered right away. We have included an image of what a delivered budget rule notification looks like. 

We have included an example of what a budget alert notification would look like delivered to a recipient.

   ![Screenshot showing an example of a budget alert notification](./images/budget-alert-example.png)

You have now created your first budget and budget alert rule. 

You may now proceed to the next task.

## Task 2b (Optional): Budgets using OCI CLI 

You can also use OCI CLI to compose commands to take action in your tenancy.

This is the CLI command for creating a monthly budget on a compartment:

 ```text
    <copy>
oci budgets budget budget create \
  --compartment-id <tenancy_ocid> \
  --target-type COMPARTMENT \
  --targets '["<target_compartment_ocid>"]' \
  --amount <budget_amount> \
  --reset-period MONTHLY \
  --display-name "<budget_display_name>" \
  --description "<budget_description>"
</copy>
```
> **Note:** In the current OCI CLI, the budget resource is created under the tenancy/root compartment. Use the target fields to scope the budget to a compartment or tag.


This is the CLI command for creating a monthly budget on a cost-tracking tag:
 ```text
    <copy>
oci budgets budget budget create \
  --compartment-id <tenancy_ocid> \
  --target-type TAG \
  --targets '["<tag_namespace>.<tag_key_name>.<tag_key_value>"]' \
  --amount <budget_amount> \
  --reset-period MONTHLY \
  --display-name "<budget_display_name>"
</copy>
```

This is the CLI command for creating a budget alert rule:

 ```text
    <copy>
oci budgets budget alert-rule create \
--budget-id <budget_ocid> \
--threshold <threshold_amount> \
--threshold-type ABSOLUTE or PERCENTAGE \
--type ACTUAL or FORECAST \
--message "message_for_recipients" \
--recipients "<email_address>" \
--display-name "<name_of_budget_alert_rule>"
</copy>
```


You have now created your first budget and budget alert rule. 

You may now proceed to the next task.


## Task 3a: Cost Analysis & Reports using OCI Console 
In this task, we are going to generate a cost analysis report using filters and dimensions and export the generated report. We are also going to view and download a FOCUS cost report. 

1. Click the **Navigation Menu** on the upper left. Navigate to **Billing & Cost Management**, and select **Cost Analysis**

     ![Screenshot showing how to navigate to Cost Analysis](./images/navigate-to-cost-analysis.png)

2. Click Add Filter then click Tag.

     ![Screenshot showing how to add a tag filter for cost analysis](./images/cost-analysis-add-filter.png)

3. Input the Tag Namespace and Tag Key for the cost-tracking tag that you created earlier. Keep it as Match any value.

     ![Screenshot showing how to input tag key for cost analysis filter](./images/cost-analysis-add-filter-tag.png)

4. Click Select.

     ![Screenshot showing how to select tag](./images/cost-analysis-select-filter.png)

5. Click Grouping Dimensions then click Region.

     ![Screenshot showing how to apply grouping dimension](./images/cost-analysis-grouping-dimension.png)

6. Click Apply.

     ![Screenshot showing how to click apply](./images/cost-analysis-filter-grouping-dimension.png)

     You should see the generated cost analysis based on your filter and grouping dimension selections.

7. Click on Bars to change the visualization to Lines.

     ![Screenshot showing how to change visualization from bars to lines](./images/cost-analysis-bars.png)

8. Click on Tab Actions to download your visualization to your local machine.

     ![Screenshot showing how to download your cost analysis visualization](./images/cost-analysis-download.png)

    Next, we will download and view a FOCUS cost report.

9. Click the **Navigation Menu** on the upper left. Navigate to **Billing & Cost Management**, and select **Cost and Usage Reports**.

     ![Screenshot showing how to navigate to cost and usage reports](./images/navigate-to-cost-and-usage-reports.png)

10. Navigate to FOCUS Reports and click the arrow to expand it.

     ![Screenshot showing how to view FOCUS Reports](./images/cost-report-page.png)

    > **Note:** FOCUS Reports are organized by year, month, day, and multiple time stamps during the day.

    ![Screenshot showing how to expand FOCUS reports](./images/cost-report-FOCUS.png)

11. Once you have decided on a FOCUS cost report to download, click on the 3 dots to the right side of the report. Click Download Report to download your FOCUS cost report to your local machine.

     ![Screenshot showing how to download FOCUS reports](./images/cost-report-download.png)

12. Open the FOCUS cost report file you just downloaded.

The FOCUS report contains data on resource usage and consumption costs in your tenancy. This data is stored in OCI Object Storage and can easily be used to build visual dashboards in Oracle native services such as Oracle Analytics Cloud. You can also import FOCUS cost reports to third party tools to digest and analyze data contents.

> **Note:** Your tenancy may not have generated any FOCUS cost reports yet.

We have included an example of a FOCUS cost report would look like for an OCI tenancy. The FOCUS cost report has many columns but here are some important ones to note: UsageQuantity, Tags, PricingUnit, ListUnitPrice, BilledCost.

   ![Screenshot showing an example of a FOCUS report](./images/FOCUS-report-example-final.png)

You have now generated your first cost analysis report and downloaded a FOCUS cost report.

You may now proceed to the next task.

## Task 3b (Optional): Cost Reports using OCI CLI

You can also use OCI CLI to compose commands to take action in your tenancy.

This is the CLI command for downloading a cost report.

 ```text
    <copy>
oci os object get \
  --namespace-name bling \
  --bucket-name "<your_tenancy_ocid>" \
  --name "reports/cost-csv/<report_name>.csv.gz" \
 --file "<local_report_name>.csv.gz"
</copy>
```




Congratulations! You may move on to the next part of the lab.


## Learn More

- [Using Cost Tracking Tags](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/usingcosttrackingtags.htm)
- [Quickly and easily apply budgets to manage OCI spending](https://www.ateam-oracle.com/apply-budgets-easily)
- [FOCUS Cost Reports Explained](https://blogs.oracle.com/cloud-infrastructure/announcing-focus-support-for-oci-cost-reports)
- [Creating a Budget](https://docs.oracle.com/en-us/iaas/Content/Billing/Tasks/create-budget.htm)
- [How to use Cost Analysis](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costanalysisoverview.htm)

## Acknowledgements

- **Author** - Wynne Yang
- **Contributors** - Daniel Hart, Deion Locklear, Eli Schilling, Wynne Yang
- **Last Updated By/Date** - Published May, 2026
