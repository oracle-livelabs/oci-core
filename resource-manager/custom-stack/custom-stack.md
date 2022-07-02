# Create Custom Stack

## Introduction

This lab is very similar to the previous one; however, it uses a custom terraform configuration instead of a provided template to create the stack. For the purposes of this lab, we have provided a terraform configuration which you will download and use to create a custom stack with Resource Manager. 

In this lab, you will create configure identity access manager, create a resource manager stack, and execute the stack job.  As an optional exercise, you can migrate your Terraform configuration to Gitlab and redirect the source of your resource manager stack.

**Note:** *Some of the UIs might look a little different than the screenshots included in the instructions. However, you can still use the instructions to complete the hands-on labs.*

### Prerequisites

- Oracle Cloud Infrastructure account credentials (User, Password, and Tenant)
- Basic understanding of Terraform concepts
- (Optional) Gitlab.com account.

**Note:** You can skip the steps below if you are using an user with admin privileges. If this were a real production system, it's both more secure and practical to create additional groups with more granular permissions. For example, it is likely we'd need to create a development team group that can only use predefined stacks and run jobs against it (use-orm-stack and use-orm-job, respectively).
[Check Best Practices for IAM](https://docs.cloud.oracle.com/iaas/Content/Security/Concepts/security_features.htm#IdentityandAccessManagementIAMService).

If you are not the administrator, you have to request the admin to give you permissions to manage Resource Manager Stacks by creating the following IAM policies in a compartment of your choice with the following statements:

  - `Allow group <group_name> to manage orm-stacks in compartment <compartment_name>`
  - `Allow group <group_name> to manage orm-jobs in compartment <compartment_name>`



## Task 1: Create Resource Manager Stack

 A Stack represents definitions for a collection of OCI resources within a specific compartment. With this in mind, we're going to configure a new stack in the compartment of your choice and name it "HA Load Balanced Simple Web App". As the stack's name suggests, its configuration files define the load balancing, networking, and compute resources to deploy the target architecture plus an HTTP server.

 Download [HA Load Balanced Simple Web App](https://objectstorage.us-ashburn-1.oraclecloud.com/p/Jp3iMGo_6czdM4qk45cBROd9hO5R1BhtTKkFm2kjoB8MoiyZk6NBSa-5JUb6fSIW/n/ociobenablement/b/hol-labs/o/orm-lbass-demo.zip) and save to your local machine.

  <!--- old par https://objectstorage.us-phoenix-1.oraclecloud.com/p/9avXPYdf7I4aoZCsBrESnmoimx1ZlHD8neRL45HMNOnEwq2KWKeGYTz4aL9wSHWk/n/ociobenablement/b/hol-files/o/orm-lbass-demo.zip --->

1. Click the **Navigation Menu** in the upper left, navigate to **Developer Services**, and select **Stacks**.

	![](https://oracle-livelabs.github.io/common/images/console/developer-resmgr-stacks.png " ")

2. Click **Create Stack**.

      - Select **My Configuration**, choose the **.ZIP FILE** button, click **Browse** link and select the terraform configuration zip file [orm-lbass-demo.zip](https://objectstorage.us-ashburn-1.oraclecloud.com/p/Jp3iMGo_6czdM4qk45cBROd9hO5R1BhtTKkFm2kjoB8MoiyZk6NBSa-5JUb6fSIW/n/ociobenablement/b/hol-labs/o/orm-lbass-demo.zip). Click **Select**.

      ![](./images/zip-file.png) 

      - **Name:** HA Load Balanced Simple Web App
      - **Description:** Provisions a primary load balancer and a failover load balancer into public subnets distributing load across 2 compute instances hosting a simple web app application.
      - **Create in Compartment:** Select an existing compartment
      - **Terraform Version:** Select 0.13.x

    ![](./../custom-stack/images/CreateStack01.png " ")

3. Click **Next**.   
      - **Configure Variables:** Configure the variables for the infrastructure resources that this stack will create when you run the apply job for this execution plan.
        - **Select a Flex Load Balancer with Minimum and Maximum Bandwidth:** 10Mbps for both minimum and maximum bandwidth
        - **Select Compute Shape:** VM.Standard2.1
        - **Select Availability Domain:** <*Pick one Avaiability Domain*>
        - **SSH Key Configuration:** <*Enter the content of your public ssh key*>
      - **Virtual Cloud Network Configuration:** 
        - **Enter your VCN Name:** vcn01
        - **Enter your CIDR Block:** 10.0.0.0/16
        - **Enter your Subnet Name:** subnet

        ![](./../custom-stack/images/CreateStack02.png " ")

4. Click **Next**.
      - **Verify your configuration variables**
      - Click **Create**

     ![](./../custom-stack/images/CreateStack03.png " ")

5. Before moving on to executing a job, quickly review the newly configured stack and then click on the hyperlinked stack name.

    ![](./../custom-stack/images/image002.png " ")

## Task 2: Execute Jobs: Plan & Apply

Jobs perform actions against the Terraform configuration files associated with a stack. You can perform 3 actions and they are plan, apply and destroy. Since Terraform command execution is not atomic, it is crucial to prevent any race conditions or state corruption from occurring due to parallel execution. To prevent this from happening, the Resource Manager ensures only one job can run against a stack at a given time against a single state file.

From the Stack Details page, we can completely manage the stack's configuration (i.e., update, delete, add tag/s, edit variables) and also download the zip archive containing the latest Terraform configuration - which can be especially helpful when troubleshooting.

1. Lets execute a plan by clicking on **Plan** and enter the following information:

      - **Name:** HA LB App Plan
      - Click **Plan**

    ![](./../custom-stack/images/plan01.png " ")

    ![](./../custom-stack/images/plan02.png " ")

    **Note:** Once the modal closes, notice the job's state appears as "Accepted" - which indicates that the platform is spinning up resources needed for executing the command  - followed by "In Progress" and then either "Succeeded" or "Failed".

    ![](./../custom-stack/images/plan03.png " ")

2. Once the job succeeded, on the Job Details page review the information and scroll through the logs containing the Terraform output. You may also edit the job or download the Terraform Configuration and logs.

3. Since the previous plan action succeeded, lets go back to the Stack page by clicking Stack Details breadcrumb on top of the page. On the Stack details page you can select the Apply button. Click on **Apply**.

    ![](./../custom-stack/images/apply01.png " ")

4. Enter the following information:

      - **Name:** HA LB App Apply
      - **Apply Job Plan Resolution** HA LB App Plan (you can select the latest succeed plan job to apply)
      - Click **Apply**

    ![](./../custom-stack/images/apply02.png " ")

5. The job state is updated as the job execution nears completion:

   ![](./../custom-stack/images/apply03.png " ")
   ![](./../custom-stack/images/apply04.png " ")
   
6. Once the apply action succeeds, verify the resources have been provisioned by reading the Terraform output contained with the logs or navigate to Networking and view the different resources that now exist (VCN, load balancer, subnets, etc.) and that the 2 instances are listed in Compute. The Health Status of the Load Balancer will need a few minutes to get into OK status.

    ![](./../custom-stack/images/loadbalancer.png " ")
    ![](./../custom-stack/images/instances.png " ")

7. When you see the Load Balancer status change to OK, copy the **IP Address** and paste it into a new web browser tab.  You should see the sample web page load and atop the page it indicates which web server you are connected to.  Press **F5** a couple of times and see the web server change as you refresh the page.  Congratulations - your sample application deployed successfully.


## Task 3: Execute Jobs: Destroy

Now that we've successfully applied our Terraform to build out our cloud resources (and optionally completed the source migration to Gitlab), let's return to the Stack Details page and use the Resource Manager to tear it all down.

1. Start by clicking on  **Destroy** and enter the following information:

      - **Name:** HA LB App Destroy
      - Click **Destroy**

    ![](./../custom-stack/images/destroy01.png " ")

    ![](./../custom-stack/images/destroy02.png " ")

1. Once again, notice that the state change is reflected in the console:  

    ![](./../custom-stack/images/destroy03.png " ")
    Wait until the status shows **Succeeded** before proceeding.

1. The final step is to delete the stack by clicking on the More Actions button on Stack Details page. Click on **Delete Stack** and confirm it by clicking **Delete** on the modal window.

    ![](./../custom-stack/images/destroy04.png " ")
    ![](./../custom-stack/images/destroy05.png " ")


*Congratulations! You have successfully completed the lab.*

## Acknowledgements

- **Author** - Flavio Pereira, Larry Beausoleil, Eli Schilling
- **Adapted by** -  Yaisah Granillo, Cloud Solution Engineer
- **Contributors** - Arabella Yao, Kamryn Vinson, Orlando Gentil
- **Last Updated By/Date** - Orlando Gentil, Jun 2021

