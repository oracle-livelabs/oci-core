# Provision the resources using OCI Resource manager

## Introduction

In this lab, we will provision all the required OCI resources for the Mushop application using OCI Resource Manager.

Estimated Time: 60 minutes

### Objectives

- Create custom image and prepare terraform files
- Provision OCI resources using OCI resource manager
- Verify the OCI resource manager job status

### Pre-requisites 

- You should use OCI tenancy with credits or a paid tenancy, **FREE TIER** account will not work!
- Preferably use OCI user with **administrator** role to provision the OCI resources as part of this lab. If not please check the OCI  documentation and create necessary IAM policies for the OCI user to create and manage OCI VCN,Load Balancer,Autonomous Database Serverless-ATP,Block Storage,Compute,Object Storage,Cloud Shell. 
- Provide necessary IAM policies to the OCI user for using Full Stack DR. Refer to this blog [Full Stack DR IAM policies](https://blogs.oracle.com/maa/post/iam-policies-fullstackdr).


## Task 1: Create custom image and prepare terraform files

1. We will use **Ashburn** as primary region and **Phoenix** as standby region. Both region details are used in the terraform files. If you want to use different regions, please modify the region (primary) and remote_region (standby) details in the variables.tf file.More details about the terraform files are there from step 9-12. If you are using different Primary and Standby region, make sure to complete the rest of lab 1 steps from your **Primary** region.

2. Login into OCI Console with your provided Credentials. Select **ASHBURN** region.

    ![ashburn console](./images/ashburn-region-new.png " ")

3. Create custom image in the Ashburn region.From the Hamburger menu,select Compute->Custom images

    ![custom image](./images/ashburn-custom-image.png " ")

4. Click Import image and provide the below details.

    - Select the compartment of your choice and use the same compartment for deploying all resources as part of this lab. 
    - Provide Name as **mushopimage**
    - Select Operating system as **Oracle Linux**
    - Select **Import from an Object Storage URL**
    - Paste the below URL inthe **Object Storage URL**
        ````
        <copy>https://objectstorage.us-ashburn-1.oraclecloud.com/p/N1Q_gWVqQ2ETQ6kf6S-WTqd3s3uv6vPs4SniklnUf7wrukkhAWPEFC1EuxE3wRqI/n/idfwhcj05ugj/b/fsdrs/o/mushop-new-image</copy>
        ````
    - Select Image type as **OCI**
    - Verify the details and click **Import image**

    ![create custom image](./images/ashburn-create-custom-image.png " ")

5. It will take approximately 10-15 minutes to show the custom image as **Available**.

    ![Custom image created](./images/ashburn-available-custom-image.png " ")

6. Copy the OCID of the custom image and keep it handy. 

    ![Custom image ocid](./images/ashburn-ocid-custom-image.png " ")

7. Click on the link below to download the Resource Manager zip file.

    [fullstackdr-mushop.zip](https://idfwhcj05ugj.objectstorage.us-ashburn-1.oci.customer-oci.com/p/h8n0TsiellDDWa-fODHkWs3mg4nPCirw7sSlU_5sMsqLIJUutEHFrdISSro5NE5I/n/idfwhcj05ugj/b/fsdrs/o/fullstackdr-mushop.zip) - Packaged terraform script for creating various OCI resources for Mushop application.

8. Save it in your downloads or any other preferred folder.

9. Unzip  **fullstackdr-mushop.zip** 

10. Navigate to **fullstackdr-mushop->terraform->scripts** folder

11. Open **compute.tf** file in your preferred editor, you should add the OCID of your compute image (taken in step 8) in the **`source_id`** which is under **`source_details`** section

    ![source-custom-id](./images/ashburn-source-custom-id.png " ")

12. Save the compute.tf file and zip or compress the folder as **fullstackdr-mushop-updated.zip**. This zip file should have the **updated compute.tf** file.

## Task 2: Provision OCI resources using OCI resource manager

1. Click the Navigation Menu in the upper left, navigate to Developer Services, and select Stacks. Make sure you are in the **Ashburn** region.

    ![resource manager](./images/ashburn-resource-manager.png " ")

2. Click **Create Stack** and select the compartment.

    ![create stack](./images/ashburn-create-stack.png " ")

3. In the **Stack information** section,provide the below details.

    - Choose **My configuration**
    - In the **Stack congiguration**, use **.Zip file** and upload **fullstackdr-mushop-updated.zip**. You should use the zip file from Task 1 -> Step 14.
    - In the **Name**,provide the name as **fullstackdr-mushop-stack**
        ![create stack](./images/ashburn-create-stack-1.png " ")
    - Select the **compartment** of your choice
    - Leave the other values as default.
    - Click Next
    ![create stack](./images/ashburn-create-stack-2.png " ")

4. In the **Configure variables** section, provide the below details  in the variables section.  
 
    - ociCompartmentOcid: Provide your compartment OCID
    - ociTenancyOcid: Provide your tenancy OCID
    - ociUserOcid: Provide your OCI user OCID
    - resId:Provide a random 5 digit number
    ![create stack](./images/ashburn-create-stack-3.png " ")
    ![create stack](./images/ashburn-create-stack-4.png " ")
    - Leave the rest of the variables section with default values.
    - Click Next

5. In the **Review** section
 
    - Review the **Stack Information** and **Variables**
    - Select the checkbox **Run apply** in the Run apply on selected stack
    - Click **Create**
    ![create stack](./images/ashburn-create-stack-5.png " ")

## Task 3: Verify the OCI resource manager job

1. Navigate to **Stacks**, select **fullstackdr-mushop-stack**

    ![mushop stack](./images/ashburn-mushop-stack.png " ")

2. You should be able a see oracle resource manager (orm) job which is in-progress state.Click the job details and monitor.

    ![mushop job](./images/ashburn-mushop-job-inprogress.png " ")

3. The job will take approximately 30 minutes to complete. Verify the status of job, it should show as **Succeeded**. In case if the job fails, verify the logs and take necessary action. Common causes for failures are missing IAM policies, resource quota not available.

    ![mushop job done](./images/ashburn-mushop-job-done.png " ")

You may now [Proceed to the next lab](#next)

## Acknowledgements

- **Author** - Suraj Ramesh,Principal Product Manager,Oracle Database High Availability (HA), Scalability and Maximum Availability Architecture (MAA)
- **Last Updated By/Date** - Suraj Ramesh,November 2024

