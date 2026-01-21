# Deploy APEX service
## Introduction

In this lab, we will learn how to deploy Oracle APEX service.

Estimated Time: 5 minutes

### Objectives

In this lab, you will:
- Deploy the APEX service on the Autonomous AI Database.

### Prerequisites

- This lab requires completion of the **Create an Autonomous AI Database with Oracle AI Database 26ai** section in the Contents menu on the left.

- Have an Autonomous AI Database with its admin password (i.e. *Oracle123!!!*).

## Task 1: Configure APEX 

1. Click the navigation menu on the upper left to show top-level navigation choices and then click **Oracle AI Database > Autonomous AI Database**.

    ![Autonomous AI Database](./images/navigate-adb.png)

2. Click on *KO2_ADB* database name.

3. Click on the button **Database Actions / View All Database Actions**
    ![Database Actions](./images/database_actions_adb.png)

4. Choose **APEX**. Click **Open**
    ![APEX](./images/ai_agent_db_action_apex.png)

5. If you have already an existing APEX workspace, go to next step. If not, create one with **APEX Administration Services**.
    - Log into the **Administration Service**, enter your database admin password (i.e. *Oracle123!!!*).

    ![APEX Administration Sign In](./images/apex_administration_adb.png)

6. In Welcome to Oracle APEX!, click **Create Workspace**    
    ![Create APEX Workspace](./images/apex_workspace_adb.png)

    - Choose **New Schema**

    ![Create APEX Schema](./images/apex_schema_adb.png)
    
    - In **Create Workspace** dialog add the following: (Keep note of it!)
         - Workspace Name: **AGENT**
         - Workspace Username: **AGENT**
         - Workspave Password: **##YOUR_PASSWORD##**
         - Click **Create Workspace**

    ![Create APEX Workspace](./images/apex_workspace_adb.png)    

7. Your workspace is created. Click the top-right icon **Admin**. Then **Sign Out** and then **Return to Sign In Page**.
    ![Sign out of Administration Services](./images/apex_signout_adb.png)
    ![Sign out of Administration Services](./images/apex_signout2_adb.png) 
 
8. Log into your **APEX workspace**:
    - Workspace: **AGENT**
    - USER: **AGENT**
    - Password: **##YOUR_PASSWORD##**
    - Click **Sign In**

    ![Sign into your workspace](./images/apex_signin_workspace_adb.png)
    ![Sign into your workspace](./images/apex_signin_workspace2_adb.png) 

_Congratulations! You have successfully completed the lab._

## Learn More
- [Getting Started with Oracle APEX in Oracle Cloud](https://docs.oracle.com/en/cloud/paas/apex/gsadd/index.html)

## Acknowledgements

* **Author** - Cristian Manea - May 2025

* **Last Updated By/Date** - Cristian Manea - December 2025
