# Add members to the DR Protection groups

## Introduction

In this lab, we will add members to the DR Protection groups created and associated in the previous lab. Ashburn is a primary region, and Phoenix is the standby region. This is an optional lab, in case if you want to destroy all the OCI resources.

**Members**- A resource type which can be added to a DRPG. Full Stack DR currently supports all the below member types. These resources can be added as member to a DRPG.

- Compute ( Standard Compute and Dedicated VM Host (DVH))
- Block Storage (Volume Groups)
- Oracle Autonomous Database Serverless (ADB-S)
- Oracle Autonomous Database on Dedicated Exadata Infrastructure (ADB-D)
- Autonomous Database on Exadata Cloud@Customer (ADB-C@C)
- Oracle Base Database Service (BaseDB/DBCS)
- Oracle Exadata Database Service on Dedicated Infrastructure (ExaDB-D/ExaCS)
- Oracle Exadata Database Service on Cloud@Customer (ExaCC)
- Oracle Exadata Database Service on Exascale Infrastructure (ExaDB-XS)
- Load Balancer and Network Load Balancer 
- File Storage Service
- Object Storage Bucket

Estimated Time: 15 Minutes

### Objectives

- Add members to Ashburn DRPG (Primary)- ATP(Primary DB),MuShop Compute VM's(node-0 and node-1),two volume groups( Boot volumes of MuShop Compute VM's), Load Balancer
- Add members to Phoenix DRPG (Standby)-ATP(Standby DB), Load Balancer

As part of the MuShop architecture,Virtual machines are deployed as Cold VM or Pilot light pattern.Full Stack DR will create the MuShop VM's in the phoenix region during the Switchover.

## Task 1: Add members to Ashburn DRPG (Primary)

1.  Login into OCI Console with your provided Credentials. The primary region should be **Ashburn**.

    ![oci console ashburn](./images/ashburn-region-new.png)

2.  Select **Migration and Disaster Recovery** from the Hamburger menu, then **Disaster Recovery** -> **DR Protection Groups**. Verify the region in **Ashburn**

    ![drpg navigation page](./images/ashburn-drpgpage-new.png)

3.  You will land on the Disaster Recovery Protection group home page; make sure you have selected *the Ashburn* region.

    ![drpg landing page](./images/drpg-status-ashburn-new.png)

4.  In the Ashburn region DRPG page, add the members required in the **mushop-ashburn** DRPG. *We will add ATP Primary Database, two mushop compute VMs,two-volume groups for the boot volumes of mushop compute VMs and Load Balancer*. Let's add those details.

5.  Add ATP Primary Database. 

    Select **mushop-ashburn** DRPG, navigate to **Members** in the *Resources* section, and hit **Add Member**

    ![drpg add member](./images/ashburn-add-member-new.png)

    It will show various resource types and select **Autonomous Database**
    ![drpg resource type](./images/ashburn-resource-new-members.png)

    Select the Database in your compartment; it will have MushopDB-XXXXX. 
    
    Default option for standby type of DR Drill is "Refreshable clone", so we will select this.

    Verify it and hit add. Make sure to check the box **"I understand that all existing plans will be deleted"**

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


## Task 2: Add members to Phoenix DRPG (Standby)

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

    Select the Database in your compartment; it will have **MushopDB-XXXXX**.
    
    Default option for standby type of DR Drill is "Refreshable clone", so we will select this.

    Verify it and hit add. Make sure to check the box **"I understand that all existing plans will be deleted"**

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
- **Last Updated By/Date** -  Suraj Ramesh,November 2024
