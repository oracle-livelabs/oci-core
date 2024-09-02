# Perform pre-checks for the DR Failover Plan and break the MuShop application

## Introduction

In this lab, we will execute prechecks for the **mushop-app-failover-iad-phx** failover plan, which we created in lab 3 and break the Mushop app. Run Prechecks will execute all the **Built-in Prechecks** in parallel. Running **Run Prechecks** will only perform the pre-checks and not the actual execution. Since we are running a failover plan, depending on the number of affected services, during actual failover area some prechecks might fail. In this lab, since we are mimicking a failover, prechecks should work fine. 

Estimated Time: 10 Minutes

Watch the video below for a quick walk-through of the lab.
[Failover Prechecks](videohub:1_sbpe8nrp)

### Objectives

- Perform Run prechecks for the Mushop application failover plan
- Monitor the executed prechecks plan
- Verify the executed prechecks plan
- Verify the MuShop application and break the application

## Task 1: Perform Run prechecks for the Mushop application failover plan

1.  Login into OCI Console with your provided Credentials. Select region as **Pheonix**.

    ![phoenix oci console](./images/phoenix-region-new.png)

2.  Select Migration and Disaster Recovery from the Hamburger menu, then **Disaster Recovery** -> **DR Protection Groups**. Verify the region is **Phoenix**

    ![phoenix navigate drpg](./images/phoenix-drpgpage-new.png)

3.  You will land on the Disaster Recovery Protection group home page; make sure you have selected the Phoenix region.

    ![drpg landing page](./images/drpg-status-phoenix-new.png)

4.  Select the **mushop-phoenix** DRPG and select **mushop-app-failover-iad-phx** plan

    ![drpg switchover plan](./images/phoenix-fodrplan-created-new.png)

5.  Navigate to the **Run prechecks** section, which will be right below the **mushop-app-failover-iad-phx** plan

    ![navigate drpg prechecks](./images/phoenix-forun-prechecks-new.png)

6.  In the **Run prechecks** window, provide the Plan execution name as **mushop-app-failover-iad-phx-prechecks** and hit **Run prechecks**

    ![drpg execute prechecks](./images/phoenix-foexecute-prechecks-new.png)

## Task 2: Monitor the executed prechecks plan

1.  Navigate to **Plan executions** section under **Resources** and select the **mushop-app-failover-iad-phx-prechecks** plan execution.Initially, it will show all the **Built-in Prechecks** as *Queued*

    ![prechecks status](./images/phoenix-foexecute-queued-new.png)

2.  Refresh the page; within a few seconds, the **State** will change from *Queued* to *In Progress*.

    ![prechecks in-progress](./images/phoenix-foexecute-inprogress-new.png)

3.  All the **Built-in Prechecks**  steps will execute in parallel; you can monitor the various steps log. Navigate to the three dots section for the respective built-in step and click. You get the option to view the log and download the log. These logs are stored in the object storage bucket provided during the DRPG creation. You can monitor the Progress and download the log if necessary for troubleshooting.

    ![prechecks monitor](./images/phoenix-foexecute-monitor-new.png)

## Task 3: Verify the executed prechecks plan

1. After 2-3 mins, **Built-in Prechecks**  will be completed successfully. You can verify the duration of each step, status, duration of entire prechecks, etc.

    ![prechecks completed](./images/phoenix-foexecute-done-new.png)

Refer to the **Troubleshooting tips** section for known failures and correction actions.

## Task 4: Verify the MuShop application and break the application

1.  Login into OCI Console with your provided Credentials. Select region as **Ashburn**.

    ![oci console ashburn](./images/ashburn-region-new.png)
  
2.  Gather the Load Balancer public IP.From the Hamburger menu, select **Networking**, then **Load Balancers**
  
    ![navigate loadbalancer](./images/ashburn-loadbalancer-navigate-new.png)

    Gather the Public IP address of the Load Balancer

     ![get loadbalancer IP](./images/ashburn-loadbalancer-ip-new.png)

    Verify the mushop application using **http://ashburn loadbalancer ip**
    replace with the actual value from Step 2. Navigate around the site and verify the cat products.

    ![mushop app](./images/mushop-app-new.png)

3.  Let us break the MuShop Application to create an outage.Click the mushop-xxxxx load balancer details; in the resources section, select Listeners.

    ![load balancer landing page](./images/loadbalancer-ash-new.png)

    ![listeners load balancer](./images/loadbalancer-listeners-new.png)

4.  Select the edit option using the three dots symbol at the right end of the port 80 listeners, mushop-xxxxx-80

    ![modify port 80](./images/port80-edit-new.png)

    Modify the listener port from 80 to 81 to break the application, and save changes. You will get work requests submitted and select close requests.

    ![modify port 80](./images/port80-edit1-new.png)
    ![modify port 81](./images/port81-edit-new.png)
    ![work request](./images/workrequest-listener-new.png)

    Wait for a few seconds and refresh the browser tab of the MuShop application. You should see that the MuShop site can't be reached or Error: Request failed with status code 502. 

    ![reverify mushop app](./images/mushop-broken-new.png)
    ![reverify mushop app](./images/mushop-broken-new1.png)

    We have created an outage to a working MuShop application; in the next lab, we will use the Full Stack DR service to initiate the failover plan of the Full Stack (App VM's/Database/Load Balancer/Application Customization) from Ashburn to the Phoenix region.

    You may now [Proceed to the next lab](#next)

## Troubleshooting tips

1.  During the pre-check execution logs, if you messages like "Requests are being throttled for instance ocid1", retry the **Run prechecks** again

    [ocid1.instance.oc1.iad.anuwcljt5h22avqcjswxs6tublhrz2qonyjmucpxlyjxplybldbpxz2jqkaq] -- Error returned by PutObject operation in ObjectStorage service.(409, Conflict, false) Server is busy. Requests are being throttled for instance ocid1.instance.oc1.iad.anuwcljt5h22avqcjswxs6tublhrz2qonyjmucpxlyjxplybldbpxz2jqkaq (opc-request-id: B7721980580148749787FE758C9440FC/3D8758622598943932D50842DCD7EAA1/143166BA790ECD35695B548600382010)

## Acknowledgements

- **Author** - Suraj Ramesh,Principal Product Manager,Oracle Database High Availability (HA), Scalability and Maximum Availability Architecture (MAA)
- **Last Updated By/Date** - Suraj Ramesh, September 2024
