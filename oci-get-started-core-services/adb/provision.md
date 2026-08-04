# Create an Oracle Autonomous AI Database

## Introduction

Learn how to provision an Autonomous AI Database instance using the OCI console.

Estimated Time: 5 minutes

### Objectives

In this lab, you will:
- Learn how to provision a new Oracle Autonomous AI Database instance.

### Prerequisites

-   This lab requires the completion of the **Get Started** lab in the **Contents** menu on the left.

## Task 1: Choose Autonomous AI Database from the Services Menu
1. Open the **Navigation** menu.

    ![Open the Navigation menu.](images/open-navigation-menu.png =65%x*)

2. Under **Oracle AI Database**, click **Autonomous AI Database**.

    <if type="livelabs">

     The **Autonomous AI Databases** page is displayed.
    
    >**Note:** The **Couldn't load data** error on the page is due to being in the wrong compartment. You will learn how to navigate to your assigned compartment next. 

    ![The new Autonomous Databases page.](images/new-adb-page.png =65%x*)

    </if>

    <if type="freetier">
    The **Autonomous AI Databases** page is displayed.

    ![The Autonomous Databases page is displayed.](images/autonomous-ai-databases-page.png =65%x*)

    OCI resources are organized into compartments. Click the **Compartment** field to select the compartment where you want to create your Autonomous Database.
    </if>
    
    <if type="livelabs">
    OCI resources are organized into compartments. To navigate to your assigned sandbox reservation compartment, click the **Compartment** field. Next, enter your assigned compartment name (or partial name) from the **Reservation Information** page in the **Compartment** text box. Once your assigned compartment is displayed in the drop-down list under the **`Livelabs`** node, click it.
    
    ![Select your assigned compartment.](images/ll-select-compartment.png =65%x*)

    The **Compartment** field shows your selected compartment.

    >**Note:** For more details on finding your assigned resources in your reservation such as the username, password, compartment and so on, review the **Get Started** lab in the Navigation menu on the left.
    </if>

    <if type="freetier">
    > **Note:** Avoid the use of the `ManagedCompartmentforPaaS` compartment as this is an Oracle default used for Oracle Platform Services.
    </if>

3. You can use the **Search and Filter** field to control the list of Autonomous Databases that are displayed on the page. This is useful when you are managing many Autonomous Databases. For example, you can use **Workload type** to filter the Autonomous Database list by selecting your workload type. You can also use the **State** filter to view databases that are `available`, `stopped`, `terminated` and much more.

    ![Database list.](images/search-and-filter.png =65%x*)

<if type="freetier">
4. If you are using a Free Trial or Always Free account, and you want to use Always Free Resources, you need to be in a region where Always Free Resources are available. You can see your current default **region** in the top, right hand corner of the page.


## Task 2: Create the Autonomous AI Database Instance
1. On the **Autonomous AI Databases** page, select your desired **region** and **compartment**. Click **Create Autonomous AI Database** to start the instance creation process. The **Create Autonomous AI Database Serverless** page is displayed.

    <if type="livelabs">
    ![Click Create Autonomous AI Database.](images/click-create-new-adb.png =65%x*)
    </if>

    <if type="freetier">
    ![Click Create Autonomous AI Database.](images/click-create-new-adb.png =65%x*)
    </if>

2. Specify the following:

<if type="freetier">
    - **Display name**: Enter a memorable name for the database for display purposes. For this lab, use **[](var:db_display_name)**.
    - **Database Name**: Use letters and numbers only, starting with a letter. Maximum length is 14 characters. _Underscores are not supported_. For this lab, use **[](var:db_name)**.
    - **Compartment**: Select your compartment from the drop-down list.

        ![Enter the required details.](./images/adb-create-screen-names.png =65%x*)

    >**Note:** Ensure that you use the suggested database names as instructed in this step, and not those shown in the screenshots.
</if>

<if type="livelabs">
    - **Display Name**: Enter a memorable name for the database for display purposes. For this lab, use **[](var:db_display_name)**.
    - **Database Name**: Use letters and numbers only, starting with a letter. Maximum length is 14 characters. _Hyphens and Underscores are not supported_. For this lab, use **[](var:db_name_livelabs)**. For example, if your assigned compartment is `LL166140-COMPARTMENT`, append **`LL166140`** to the database name. In this example, the database name will be **ADWFINANCELL166140**.
    - **Compartment**: Use the default compartment created for your reservation.

    ![Enter the required details.](./images/adb-create-screen-names.png =65%x*)

    > **Note:** Ensure that you use the suggested database names as instructed in this step, and not those shown in the screenshots.
</if>

3. In the **Workload type** section, choose a workload type. Select the workload type for your database from the following choices:

    - **Lakehouse**: Built for analytics and AI. Fast insights from a single lakehouse for all your data.
    - **Transaction Processing**: Provides all of the performance of the market-leading Oracle Database in an environment that is tuned and optimized to meet the demands of a variety of applications, including: mission-critical transaction processing, mixed transactions and analytics, IoT, and JSON document store.
    - **JSON**: It is Oracle Autonomous Transaction Processing, but designed for developing NoSQL-style applications that use JavaScript Object Notation (JSON) documents. You can store up to 20 GB of data other than JSON document collections. There is no storage limit for JSON collections.
    - **APEX**: It is a low cost, Oracle Cloud service offering convenient access to the Oracle APEX platform for rapidly building and deploying low-code applications

        For this workshop, accept the **Lakehouse** default selection.

        ![Choose a workload type.](images/adb-create-screen-workload.png =75%x*)

4. In the **Database configuration** section, specify the following:

    - **Always Free**: An Always Free databases are especially useful for development and trying new features. You can deploy an Always Free instance in an Always Free account or paid account. However, it must be deployed in the home region of your tenancy. The only option you specify in an Always Free database is the database version. For this lab, we recommend you leave the **Always Free** slider disabled unless you are in an Always Free account.
    - **Developer**: Developer databases provide a great low cost option for developing apps with Autonomous AI Database. You have similar features to Always Free - but are not limited in terms of region deployments or the number of databases in your tenancy. You can upgrade your Developer Database to a full paid version later and benefit from greater control over resources, backups and more. For this lab, leave the **Developer** slider disabled.
    - **Choose database version**: Select **26ai** for the database version from this drop-down list.
    - **ECPU count**: Choose the number of ECPUs for your service. For this lab, specify **[](var:db_ocpu)**. If you choose an Always Free database, you do not need to specify this option.
    - **Compute auto scaling**: Accept the default which is enabled. This enables the system to automatically use up to three times more compute and IO resources to meet workload demand.
    - **Storage (TB)**: Select your storage capacity in terabytes. For this lab, specify **[](var:db_storage)** of storage. Or, if you choose an Always Free database, it comes with 20 GB of storage.
    - **Storage auto scaling**: For this lab, there is no need to enable storage auto scaling, which would allow the system to expand up to three times the reserved storage. Accept the default which is disabled.

        > **Note:** You cannot scale up/down an Always Free Autonomous AI Database.

        ![Choose the remaining parameters.](./images/adb-create-database-configuration.png =75%x*)

        >**Note:** You can drill down on the **Advanced options** option to take advantage of database consolidation savings with **elastic pools** or to use your organization's on-premise licenses with **bring your own license**. 

5. In the **Backup** section, specify the following:
    - **Automatic backup retention period in days:** You can either accept the default value or specify your own preferred backup retention days value. For this lab, accept the default `60` days default value.
    - **Immutable backup retention:** Accept the disabled default selection.

     ![Choose backup retention.](./images/choose-backup-retention.png =75%x*)

6. In the **Administrator credentials creation** section, specify the following:

    - **Username:** This read-only field displays the default administrator username, **`ADMIN`**. _**Important:** Make a note of this **username** as you will need it to perform later tasks._
    - **Password:** Enter a password for the **`ADMIN`** user of the service instance choice such as **`Training4ADW`**. _**Important:** Make a note of this **password** as you will need it to perform later tasks._
    - **Confirm password:** Confirm your password.

        > **Note:** The password must meet the following requirements:    
            - Must be between 12 and 30 characters long and must include at least one uppercase letter, one lowercase letter, and one numeric character.    
            - Cannot contain the username.    
            - Cannot contain the double quote (") character.    
            - Must be different from the last 4 passwords used.    
            - Must not be the same password that you set less than 24 hours ago.

        ![Enter password and confirm password.](./images/adb-create-screen-password.png =75%x*)

7. In the **Network access** section, select one of the following options:
    - For this lab, accept the default selection, **Secure access from everywhere**.
    - If you want to allow traffic only from the IP addresses and VCNs you specify - where access to the database from all public IPs or VCNs is blocked, select **Secure access from allowed IPs and VCNs only** in the Choose network access area.
    - If you want to restrict access to a private endpoint within an OCI VCN, select **Private endpoint access only** in the Choose network access area.
    - If the **Require mutual TLS (mTLS) authentication** option is selected, mTLS will be required to authenticate connections to your Autonomous AI Database. TLS connections allow you to connect to your Autonomous AI Database without a wallet, if you use a JDBC thin driver with JDK8 or above. See the [documentation for network options](https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/support-tls-mtls-authentication.html#GUID-3F3F1FA4-DD7D-4211-A1D3-A74ED35C0AF5) for options to allow TLS, or to require only mutual TLS (mTLS) authentication.

        ![Choose the network access.](./images/adb-create-network-access.png =65%x*)

8. In the **Contacts for operational notifications and announcements** section, provide a contact email address. The **Contact email** field allows you to list contacts to receive operational notices and announcements as well as unplanned maintenance notifications.

    ![Provide a contact email address.](images/adb-create-contact-email.png =65%x*)

9. Click **Create**.

10. The **Autonomous AI Database details** page is displayed. The status of your ADB instance is **`Provisioning`**.

    ![Database Provisioning message.](./images/adb-create-provisioning-message-new.png =75%x*)

    A **Check database lifecycle state** informational box is displayed. You can navigate through this tour or choose to skip it. Click **Skip tour**. A **Skip guided tour** dialog box is displayed. Click **Skip**.

    In a few minutes, the instance status changes to **`Available`**. At this point, your Autonomous Data Warehouse database instance is ready to use! Review your instance's details including its name, database version, ECPU count, and storage size.
       
    >**Note:** In the following screen sample, the database display name is **`Training-Database`** and the database name is **`TrainingDatabase`**. _In your example, your **database display name** and **database name** might be different_.

    ![Database complete message.](./images/adb-created.png =75%x*)

11. Click the **Autonomous AI Databases** link in the top left of the page. The **Autonomous AI Database** page is displayed.

    ![Click left arrow.](./images/click-left-arrow.png =75%x*)

    Your new Autonomous AI Database instance is displayed. 

    In the following sample screen capture, the instance display name is **`Training-Database`**.

    ![Click left arrow.](./images/adb-home-page.png =75%x*)  

12. An email message is sent to the contact email that you provided. The email contains useful links that you can use to launch Database Actions, view the Get Started with Autonomous AI Database Web page, and access the online forums to post a question and collaborate with other Autonomous AI Database experts. 

    ![provisioning email sent.](./images/provisioning-email-generic.png =65%x*)

## Learn more

- [Using Oracle Autonomous AI Database Serverless
](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/autonomous-intro-adb.html)
- [Data Warehousing Insider blog](https://blogs.oracle.com/datawarehousing/)

## Acknowledgements

- **Author:** Lauran K. Serhal, Consulting User Assistance Developer
- **Last Updated By/Date:** Uma Kumar April
