# Create Full Stack DR Protection groups and add members

## Introduction

In this lab, we will Create and Associate Disaster Recovery Protection Groups (DRPG), Add required members in Primary and Standby DR protection groups. Ashburn is the primary region and Phoenix is the standby region.

**DR Protection group (DRPG)** – A resource type used by Full Stack DR.  A DR Protection Group represents a consistency grouping defined for the purposes of disaster recovery.  It is a collection of different OCI resources that comprise an application and must be treated as a combined group when performing disaster recovery operations.  For example, a DR Protection Group may consist of application servers (compute instances), associated block storage (grouped as volume groups), and databases.

**Members**- A resource type which can be added to DRPG.Full Stack Disaster Recovery Service currently supports disaster recovery for the following OCI resource types

- Compute Instances ( Moving and Non-moving)
- Boot and Block Volumes (Volume Groups)
- Oracle Exadata Database Service
- Oracle Base Database Service
- Oracle Autonomous Database on Shared Exadata Infrastructure (Serverless)
- Load Balancer
- File Storage Service

**DR Plans**- DR plan is an automated DR workflow (a DR runbook) created by Full Stack Disaster Recovery to perform disaster recovery for all the resources in the primary DR protection group. 

There are four types of DR plans which you can create in Full Stack DR.

- Switchover Plan - Used for planned and controlled transition of services from a primary to a standby region. It involves shutting down services in the primary region and bringing them up in the standby region. 
- Failover Plan - Used for unplanned recovery during disasters or outages. It immediately activates services in the standby region whenever the primary is entirely inaccessible.
- Drill Plans - Performing DR drills is an essential DR service capability that allows users to exercise and validate their business continuity configuration and plans without disrupting their production stack.
    - Start Drill - It creates a replica of your production stack without impacting production.
    - Stop Drill - Removes the replica of your production stack created earlier by Start drill.
 
**Plan Group** – A group of steps in a DR Plan. A DR Plan consists of one or more Plan Groups that execute sequentially. All steps in a Plan Group execute in parallel. 

There are two types of plan groups

- Built-In Groups or Steps – A type of Plan Group or Step that is generated automatically by FSDR when a DR Plan is created. Examples of Built-in Plan Steps are: Launch Compute Instance, Switchover Database, etc.

- User-Defined Groups or Steps– A type of Plan Group or Step that is added by the user to a DR Plan after the DR plan is created by FSDR. 

Estimated Time: 25 Minutes

Watch the video below for a quick walk-through of the lab.
[Creation of DR protection group and switchover plan](videohub:)

### Objectives

- Create DRPG in Ashburn and Phoenix regions.
- Associate Ashburn DRPG as primary and Phoenix DRPG as Standby
- Add members to Ashburn DRPG (Primary)- ATP(Primary DB),MuShop Compute VM's(node-0 and node-1),two volume groups( Boot volumes of MuShop Compute VM's), Load Balancer
- Add members to Phoenix DRPG (Standby)-ATP(Standby DB), Load Balancer

**As part of the MuShop architecture,Virtual machines are deployed as Cold VM or Pilot light pattern.Full Stack DR will create the MuShop VM's in the phoenix region during the DR plan execution**

## Task 1: Create DRPG in Ashburn and Phoenix regions

1.  Login into OCI Console with your provided Credentials. The primary region should be **Ashburn**.

    ![Ashburn OCI Console](./images/ashburn-region-new.png)

    Open another browser tab and then select region as **Phoenix** (Standby Region)

    ![Phoenix OCI Console](./images/phoenix-region-new.png)

2.  In the first browser tab,Select **Migration and Disaster Recovery** from the Hamburger menu, then **Disaster Recovery** -> **DR Protection Groups**. Verify the region is **Ashburn**

    ![DRPG from ashburn](./images/ashburn-drpgpage-new.png)

3.  In the second browser tab,Select **Migration and Disaster Recovery** from the Hamburger menu, then **Disaster Recovery** -> **DR Protection Groups** Verify the region is **Phoenix**

    ![DRPG from phoenix](./images/phoenix-drpgpage-new.png)

4.  You will land up at the Disaster Recovery Protection group home page; make sure to have two tabs opened for Ashburn and Phoenix region.Make sure to select the right compartment.

    ![DRPG landing page for IAD](./images/ashburn-drpg-new.png)
    ![DRPG landing page for PHX](./images/phoenix-drpg-new.png)

5.  Create DRPG in the Ashburn region. Select **Create DR Protection group** in the Ashburn region browser tab and follow the below instructions.

    - Enter name as **mushop-ashburn**
    - Select the compartment assigned to you
    - In the object storage bucket, use the drop-down option and select **mushop-xxxxx** (mushop-12345)  ( Don't select mushop-media-xxxxx bucket)
    - In role, leave it as **Not Configured**
    - Ignore add member
    - Verify and hit create

    ![create drpg](./images/ashburn-drpgcreate-new.png)

    Navigate back to the DR Protection group page and refresh; the state of DRPG will change from creating to active in a few seconds.

    ![drpg status](./images/ashburn-drpgactive-new.png)

6.  Create DRPG in the Phoenix region. Select **Create DR Protection group** in the Phoenix region browser tab and follow the below instructions.

    - Enter name as **mushop-phoenix**
    - Select the compartment assigned to you
    - In the object storage bucket, use the drop-down option and select **mushop-xxxxx** (mushop-12345) ( Don't select mushop-media-xxxxx bucket)
    - In role, leave it as **Not Configured**
    - Ignore add member
    - Verify and hit create

    ![create drpg](./images/phoenix-drpgcreate-new.png)

    Navigate back to the DR Protection group page and refresh; the state of DRPG will change from creating to active in a few seconds.

    ![drpg status](./images/phoenix-drpgactive-new.png)

## Task 2: Associate Ashburn DRPG as primary and Phoenix DRPG as Standby

1. From the Ashburn region OCI console, select **mushop-Ashburn** DRPG. Select the **Associate** button

    ![drpg associate](./images/drpg-associate-new.png)

    - Select Role as **Primary**
    - Select Peer Region as **US West (Phoenix)**,
    - Select Peer DR Protection group in the compartment (change assigned compartment if required); you should select **mushop-phoenix**
    - Verify and associate

    ![drpg associate confirm](./images/drpg-associate-1-new.png)

    **mushop-ashburn** DRPG will change to *Updating* State

    ![drpg associate update](./images/drpg-associate-updating-new.png)

    Navigate back to the DR Protection group home page. You should be able to see DRPG **mushop-ashburn** state as *Active*, role as *Primary*, peer region as *US West (Phoenix)*

    ![drpg associate done](./images/drpg-status-ashburn-new.png)

2.  From the Phoenix region OCI console, navigate to the DR Protection group home page. You should be able to see DRPG **mushop-phoenix** state as *Active*, role as *Standby*, peer region as *US East (Ashburn)*

    ![drpg associate status](./images/drpg-status-phoenix-new.png)

    Now, we have associated **mushop-Ashburn** as *Primary DRPG* and **mushop-phoenix**  as *Standby DRPG*

## Task 3: Add members to Ashburn DRPG (Primary)

1.  In the Ashburn region DRPG page, add the members required in the **mushop-ashburn** DRPG. *We will add ATP Primary Database, two mushop compute VMs,two-volume groups for the boot volumes of mushop compute VMs and Load Balancer*. Let's add those details.

2.  Add ATP Primary Database. 

    Select **mushop-ashburn** DRPG, navigate to **Members** in the *Resources* section, and hit **Add Member**

    ![drpg add member](./images/ashburn-add-member-new.png)

    It will show various resource types and select **Autonomous Database**
    ![drpg resource type](./images/ashburn-resource-new-members.png)

    Select the Database in your compartment; it will have MushopDB-XXXXX. Verify it and hit add. Make sure to check the box **"I understand that all existing plans will be deleted"**

    ![drpg add atp](./images/ashburn-atp-add-new.png)

    **mushop-ashburn** DRPG status will change to updating; wait for a few seconds. You should see that the ATP database is added as a member. Refresh the DRPG page if required. You can monitor the request's status in the **Work requests** section under Resources.

    ![drpg atp added](./images/ashburn-atp-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active. In case if you don't see the ATP DB member, add it again.

6.  Add first Compute instance **mushop-xxxxx-0** as member,select **mushop-ashburn** DRPG, navigate to **Members** in the *Resources* section, and hit **Add Member**

    It will show various resource types and select **Compute**
    ![drpg resource type](./images/ashburn-resource-new-members.png)

    - Resource Type as **Compute**
    - Make sure to check the box **"I understand that all existing plans will be deleted"**
    - Instances in Compartment, select **mushop-xxxxx-0**
    - Compute instance type, select **Moving instance**
    - Click Add VNIC mapping. This will pop up inputs for Add VNIC mapping
    - Select VNIC as *primaryvnic*
    - Destination subnet as *mushop-main-xxxxx (regional)*
    - No need to input any values for *Destination primary private IP address* and *Destination primary private IP hostname label*
    - Ignore Network security groups
    - Click Add

    ![drpg compute vnic](./images/ashburn-vnic-node0-new.png)

    -You should be able to able to add VNIC details, verify and click Add

    ![drpg vnic added](./images/ashburn-compute-node0-new.png)

    **mushop-ashburn** DRPG status will change to updating; wait for a few seconds. DRPG status will change to active.You should be able to see that compute instance **mushop-xxxxx-0** has been added as a member. Refresh the DRPG page if required. You can monitor the status in the *Work requests* section under Resources.

    ![drpg compute added](./images/ashburn-node0-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active.

7.  Add second Compute instance **mushop-xxxxx-1** as member,select **mushop-ashburn** DRPG, navigate to **Members** in the *Resources* section, and hit **Add Member**

    It will show various resource types and select **Compute**
    ![drpg resource type](./images/ashburn-resource-new-members.png)

    - Resource Type as **Compute**
    - Make sure to check the box **"I understand that all existing plans will be deleted"**
    - Instances in Compartment, select **mushop-xxxxx-1**
    - Compute instance type, select **Moving instance**
    - Click Add VNIC mapping. This will pop up inputs for Add VNIC mapping
    - Select VNIC as *primaryvnic*
    - Destination subnet as *mushop-main-xxxxx (regional)*
    - No need to input any values for *Destination primary private IP address* and *Destination primary private IP hostname label*
    - Ignore Network security groups
    - Click Add

    ![drpg compute vnic](./images/ashburn-vnic-node1-new.png)

    - You should be able to able to add VNIC details, verify and click Add

    ![drpg vnic added](./images/ashburn-compute-node1-new.png)

    **mushop-ashburn** DRPG status will change to updating; wait for a few seconds. DRPG status will change to active.You should be able to see that compute instance **mushop-xxxxx-1** has been added as a member. Refresh the DRPG page if required. You can monitor the status in the *Work requests* section under Resources.

    ![drpg compute added](./images/ashburn-node1-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active.

8.  Add the first volume group  **mushop-volume-group-0**. This volume group consists of the boot volume of mushop-xxxx-0 VM and has cross-region replication configured to the phoenix region.

    Select **mushop-ashburn** DRPG, navigate to **Members** in the *Resources* section, and hit **Add Member**

    It will show various resource types and select **Volume group**
    ![drpg resource type](./images/ashburn-resource-new-members.png)

    - Resource Type as Volume Group
    - Make sure to check the box **"I understand that all existing plans will be deleted"**
    - Select volume group **mushop-volume-group-0**
    - Verify and add

    ![drpg add volume group](./images/ashburn-add-vg0-new.png)

    **mushop-ashburn** DRPG status will change to updating; wait for a few seconds. DRPG status will change to active.You should be able to see that volume group **mushop-volume-group-0** has been added as a member. Refresh the DRPG page if required. You can monitor the status in the *Work requests* section under Resources.

    ![drpg volume group added](./images/ashburn-vg0-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active.

9. Add the second volume group **mushop-volume-group-1**. This volume group consists of the boot volume of mushop-xxxx-1 VM and has cross-region replication configured to the phoenix region.

    Select **mushop-ashburn** DRPG, navigate to **Members** in the *Resources* section, and hit **Add Member**

    It will show various resource types and select **Volume group**
    ![drpg resource type](./images/ashburn-resource-new-members.png)

    - Resource Type as Volume Group
    - Make sure to check the box **"I understand that all existing plans will be deleted"**
    - Select volume group **mushop-volume-group-1**
    - Verify and add

    ![drpg add volume group](./images/ashburn-add-vg1-new.png)

    **mushop-ashburn** DRPG status will change to updating; wait for a few seconds. DRPG status will change to active.You should be able to see that volume group **mushop-volume-group-1** has been added as a member. Refresh the DRPG page if required. You can monitor the status in the *Work requests* section under Resources.

    ![drpg volume group added](./images/ashburn-vg1-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active.

10. Add Load Balancer as member.

    It will show various resource types and select **Load Balancer**
    ![drpg resource type](./images/ashburn-resource-new-members.png)
     
    - Resource Type as Load Balancer
    - Make sure to check the box **"I understand that all existing plans will be deleted"**
    - Select Load balancer **mushop-xxxxx**
    - Select Destination load balancer **mushop-xxxxx**
    - Select Source backend set **mushop-xxxxx**
    - Select Destination backend set **mushop-xxxxx**
    - Verify and add

    ![drpg add lbr](./images/ashburn-lbr-add-new.png)

    **mushop-ashburn** DRPG status will change to updating; wait for a few seconds. You should see that the Load Balancer is added as a member. Refresh the DRPG page if required. You can monitor the request's status in the **Work requests** section under Resources.

    ![drpg lbr added](./images/ashburn-lbr-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active. In case if you don't see the Load Balancer member, add it again.

10. We have added all the required members in the **mushop-ashburn** DRPG. It should show a ATP Database, two Compute Instances, two Volume groups and a Load Balancer.DRPG status should show as active.

    ![drpg members ashburn](./images/ashburn-allmembers-new.png)


## Task 4: Add members to Phoenix DRPG (Standby)

1.  Login into OCI Console with your provided Credentials. The Standby region should be **Phoenix**.

    ![oci console phoenix](./images/phoenix-region-new.png)

2.  Select **Migration and Disaster Recovery** from the Hamburger menu, then **Disaster Recovery** -> **DR Protection Groups** Verify the region in **Phoenix**

    ![drpg navigation page](./images/phoenix-drpgpage-new.png)

3.  You will land on the Disaster Recovery Protection group home page; make sure you have selected the Phoenix region.

    ![drpg landing page](./images/drpg-status-phoenix-new.png)

4.  In the Phoenix region DRPG page, add the members required in the **mushop-phoenix** DRPG. *We will be adding ATP Standby Database and Load Balancer*. Let's add those details.  **We don't need to add compute and volume groups as we VM's are designed in cold VM DR pattern and those VM's will be created automatically during the DR switchover process by Full Stack DR**

6.  Add ATP Standby Database. Select **mushop-phoenix** DRPG, navigate to **Members** in the *Resources* section, and hit **Add Member**

    ![drpg add member](./images/phoenix-add-member-new.png)

    It will show various resource types and select **Autonomous Database**
    ![drpg resource type](./images/phoenix-resource-new-members.png)

    Select the Database in your compartment; it will have **MushopDB-XXXXX**. Verify it and hit add. Make sure to check the box **"I understand that all existing plans will be deleted"**

    ![drpg add atp](./images/phoenix-atp-add-new.png)

    **mushop-phoenix** DRPG status will change to updating; wait for a few seconds. You should be able to see ATP database has been added as Member. Refresh the DRPG page if required. You can monitor the status in the *Work requests* section under Resources.

    ![drpg atp added](./images/phoenix-atp-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active.

7. Add Load Balancer as member.

    It will show various resource types and select **Load Balancer**
    ![drpg resource type](./images/phoenix-resource-new-members.png)
     
    - Resource Type as Load Balancer
    - Make sure to check the box **"I understand that all existing plans will be deleted"**
    - Select Load balancer **mushop-xxxxx**
    - Select Destination load balancer **mushop-xxxxx**
    - Select Source backend set **mushop-xxxxx**
    - Select Destination backend set **mushop-xxxxx**
    - Verify and add

    ![drpg add lbr](./images/phoenix-lbr-add-new.png)

    **phoenix-ashburn** DRPG status will change to updating; wait for a few seconds. You should see that the Load Balancer is added as a member. Refresh the DRPG page if required. You can monitor the request's status in the **Work requests** section under Resources.

    ![drpg lbr added](./images/phoenix-lbr-added-new.png)

    Navigate back to the DR Protection group page; the status of DRPG should be active. In case if you don't see the Load Balancer member, add it again.

8.  Now, we have added all the required members in the **mushop-phoenix** DRPG. It should show ATP Database and Load Balancer. DRPG status will show as active.

    ![drpg members phoenix](./images/phoenix-allmembers-new1.png)

    You may now [Proceed to the next lab](#next)

## Troubleshooting tips

1. After adding the member, in case if the member is not showing up. Re-add the member again.

## Acknowledgements

- **Author** - Suraj Ramesh, Principal Product Manager,Oracle Database High Availability (HA), Scalability and Maximum Availability Architecture (MAA)
- **Last Updated By/Date** -  Suraj Ramesh,June 2024
