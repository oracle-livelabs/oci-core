# Prepare the environments for using the Full Stack DR Service

## Introduction

In this lab, you will download the **Autonomous Database Wallets** and prepare the **MuShop application virtual machines**, which are essential for working with the **Full Stack DR Service**.


**Estimated Time**: 15 Minutes

### Objectives

- Download Autonomous Database Wallets using Cloud Shell and transfer the wallets to application VM instances
- Connect to application VM instances and execute application change script

## Task 1: Download Autonomous Database Wallets using Cloud Shell and transfer the Wallets to application VM instances

1. Login into OCI Console with your provided Credentials. Select **ASHBURN** region.

    ![ashburn console](./images/ashburn-region-new.png " ")

2. Gather the OCID( Oracle Cloud Identifier) of the ATP database in Ashburn.

    From the Hamburger menu, select **Oracle Database**, then **Autonomous Transaction Processing**.

    ![ATP menu](./images/atp-menu-new.png)

     Make sure to change the compartment which was assigned to you.Select the compartment(**LLxxxxx-COMPARTMENT**) you were assigned. You can verify the compartment allocated to you using the **View Login Info** in the top left of the instructions page.**LLxxxxx** is the username which was used to login into the OCI console. 
     
     Expand the root compartment and then select Livelabs compartment. Under Livelabs compartment select the **correct compartment which was assigned to you without fail**.You should be able to see an ATP database, similar to below. If you cannot see, then you should be selecting different compartment. Retry by selecting the right compartment.

    ![ATP database](./images/atp-database-new.png)

    Click the ATP database which should have like **"MuShopDB-XXXXX"** and in the Autonomous Database Information tab, copy the OCID of the ATP database and keep it safe. This is required for downloading the wallet in the next step.

    ![ATP OCID](./images/atp-ocid-new.png)


3.  Open the **Cloud Shell** using the icon next to the **Ashburn** region.  

    ![open cloud shell](./images/cloud-shell-new.png)
    ![open cloud shell](./images/cloud-shell-1-new.png)

    The Cloud Shell will open after a few seconds and shows the **prompt**. Input **N** for quitting the tutorial.

4. Download the ATP database wallet running in Ashburn using the Cloud Shell.In case if the Cloud Shell got disconnected, reconnect it again.

    You can maximize the Cloud Shell view and restore it as your requirements. For better viewing, you can use maximize option.

    ![Cloud Shell Maximize](./images/cloud-max-new.png)

    **Make sure to modify the ATP database OCID for your database in the below command.You should replace the OCID after --autonomous-database-id  with your values which was captured in Step 2**

    ````
     <copy>oci db autonomous-database generate-wallet --generate-type ALL --file atpwallet_ashburn.zip --password Fsdrs@123 --autonomous-database-id ocid1.autonomousdatabase.oc1.iad.anuwcljt5h22avqazns5ncswf4jc7owrzb6nug53xxxxxxxxxxxx</copy>
    ````

    Copy the command and execute in Cloud Shell prompt.You should be able to see the Wallet file which was downloaded. Verify that using the list command ls -ltr atpwallet_ashburn.zip as provided in the screenshot.

    ![ATP Wallet](./images/atp-wallet-cs-new.png)

5. Open a new tab in the browser, login to OCI console and select **PHOENIX** region.

    ![phoenix console](./images/phoenix-region-new.png " ")

6. Gather the OCID( Oracle Cloud Identifier) of the ATP database in Phoenix.

    From the Hamburger menu, select **Oracle Database**, then **Autonomous Transaction Processing**.

    ![ATP menu](./images/atp-menu-new.png)

     Make sure to change the compartment which was assigned to you.Select the compartment(**LLxxxxx-COMPARTMENT**) you were assigned. You can verify the compartment allocated to you using the **View Login Info** in the top left of the instructions page.**LLxxxxx** is the username which was used to login into the OCI console. 
     
     Expand the root compartment and then select Livelabs compartment. Under Livelabs compartment select the **correct compartment which was assigned to you without fail**.You should be able to see an ATP database, similar to below. If you cannot see, then you should be selecting different compartment. Retry by selecting the right compartment.

    ![ATP database](./images/atp-database-phx-new.png)

    Click the ATP database which should have like **"MuShopDB-XXXXX"** and in the Autonomous Database Information tab, copy the OCID of the ATP database and keep it safe. This is required for downloading the wallet in the next step.

    ![ATP OCID](./images/atp-ocid-phx-new.png)

7.  Open the **Cloud Shell** using the icon next to the **Phoenix** region.  

    ![open cloud shell](./images/cloud-shell-phx-new.png)
    ![open cloud shell](./images/cloud-shell-1-phx-new.png)

    The Cloud Shell will open after a few seconds and shows the **prompt**. Input **N** for quitting the tutorial.

8. Download the ATP database wallet running in Phoenix using the Cloud Shell.In case if the Cloud Shell got disconnected, reconnect it again.

    You can maximize the Cloud Shell view and restore it as your requirements. For better viewing, you can use maximize option.

    ![Cloud Shell Maximize](./images/cloud-max-phx-new.png)

    **Make sure to modify the ATP database OCID for your database in the below command.You should replace the OCID after --autonomous-database-id  with your values which was captured in Step 6**

    ````
     <copy>oci db autonomous-database generate-wallet --generate-type ALL --file atpwallet_phoenix.zip --password Fsdrs@123 --autonomous-database-id ocid1.autonomousdatabase.oc1.phx.anyhqljt5h22avqaw32nmjoi7d5zhxbkt6txxxxxxxxxxxxx</copy>
    ````

    Copy the command and execute in Cloud Shell prompt.You should be able to see the Wallet which was downloaded. Verify that using the list command ls -ltr atpwallet_phoenix.zip as provided in the screenshot.

    ![ATP Wallet](./images/atp-wallet-cs-phx-new.png)

    In case if you are getting "NotAuthorizedOrNotFound" error, make sure you are running the command in the Phoenix region and Cloud Shell shows phoenix region in the prompt.  

9. **Switchback to the Ashburn region Cloud Shell tab and rest of the Lab 1 will be done in Ashburn region**.

    Download the private key for connecting to MuShop compute VM's using the below command in the Cloud Shell.

    ````
    <copy>wget https://c4u04.objectstorage.us-ashburn-1.oci.customer-oci.com/p/EcTjWk2IuZPZeNnD_fYMcgUhdNDIDA6rt9gaFj_WZMiL7VvxPBNMY60837hu5hga/n/c4u04/b/livelabsfiles/o/labfiles/mushoppk</copy>
    ````

    ![Mushop privatekey](./images/mushoppk-key.png)

10. Change the permission of the private key to `0600` in in the Cloud Shell

    ````
    <copy>chmod 0600 mushoppk</copy>
    ````

11. Use the other tab of the browser, From the Hamburger menu, select **Compute**, then **Instances**. Verify the region as **Ashburn**

    ![Compute navigation](./images/compute-navigate-new.png)

    Gather the Public IP from the two MuShop application instances. **mushop-xxxxx-0-->Node 0** and **mushop-xxxxx-1-->Node 1**. In case if you are not able to view the instances, select the correct compartment assigned to you. Refer Step-2 for more details.

    ![Copy publicip](./images/compute-publicip-new.png)

12. Navigate to the Cloud Shell browser tab and scp (Secure copy) the ATP wallets(Ashburn and Phoenix) to MuShop Application compute instances (both mushop-xxxxx-0 and mushop-xxxxx-1 )

    ````
    <copy>scp -i mushoppk atpwallet_ashburn.zip atpwallet_phoenix.zip opc@publicipnode0:/home/opc</copy>
    ````

    ````
    <copy>scp -i mushoppk atpwallet_ashburn.zip atpwallet_phoenix.zip opc@publicipnode1:/home/opc</copy>
    ````

    Replace `publicipnode0` and `publicipnode1` with the public IP address(step 11) of both mushop-xxxxx-0 and mushop-xxxxx-1 respectively in the scp commands.

    Execute the scp commands in the cloud shell and while it prompts for confirmation key in as yes. Make sure the ATP wallets for both **Ashburn and Phoenix** are transferred successfully to both muShop application compute instances.

    ![Wallet copytovm](./images/wallet-compute-new.png)

## Task 2: Connect to application VM instances and run the application script

1. **Use Ashburn region for all the steps**.From the existing Cloud Shell, Connect to MuShop App VM **mushop-xxxxx-0**, replace `publicipnode0` with the public IP address of mushop-xxxxx-0. Refer task 1.11 to get the public IP address of mushop-xxxxx-0.

    ````
    <copy>ssh -i mushoppk opc@publicipnode0</copy>
    ````

    Once its connected, use ls -ltr atpwa* command in the shell. You should be able to see the ATP wallet files from both Ashburn and phoenix region files which we transferred in previous step.

    ![list atp wallet](./images/compute-node0-new.png)

2. Mushopapp script has been created to unzip the wallet files,modify changes in the mushop env files,copy the wallet to oracle client and restart the mushoapp.

    Execute the mushopapp script

    ````
    <copy>./mushopapp</copy>

    ````
3. Verify for successful execution of script and exit from mushop-xxxxx-0 node

    ````
    <copy>exit</copy>
    ````
    ![disconnect node0](./images/disconnect-mushop-node0-new.png)

4. From the existing Cloud Shell,Connect to MuShop App VM **mushop-xxxxx-1**, replace `publicipnode1` with the public IP address of mushop-xxxxx-1.Refer task 1.11 to get the public IP address of mushop-xxxxx-1.

    ````
    <copy>ssh -i mushoppk opc@publicipnode1</copy>
    ````
    Once its connected, use ls -ltr atpwa* command in the shell. You should be able to see the ATP wallet files from both Ashburn and phoenix region files which we transferred in previous step.

    ![list atp wallet](./images/compute-node1-new.png)

5. Execute the mushopapp script

    ````
    <copy>./mushopapp</copy>
    ````

6.  Verify for successful execution of script and exit from mushop-xxxxx-1 node

    ````
    <copy>exit</copy>
    ````
    ![run mushop script](./images/exec-mushapp-node1-new.png)

You may now [Proceed to the next lab](#next)

## Acknowledgements

- **Author** - Suraj Ramesh,Principal Product Manager,Oracle Database High Availability (HA), Scalability and Maximum Availability Architecture (MAA)
- **Last Updated By/Date** - Suraj Ramesh,November July 2025

