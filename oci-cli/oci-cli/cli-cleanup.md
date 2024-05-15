#  Cleaning up resources made with  OCI Command Line Interface (CLI)

## Task 1: Delete the resources
1. Switch to  OCI console window.

2. If your Compute instance is not displayed, From OCI services menu Click **Compute** -> **Instances**.

     ![](images/compute_instances.png " ")

3. Locate your compute instance, Click the Action icon and then **Terminate**.

     ![](images/RESERVEDIP_HOL0016.PNG " ")

4. Make sure Permanently delete the attached Boot Volume is checked, Click **Terminate Instance**. Wait for instance to fully Terminate.

     ![](images/RESERVEDIP_HOL0017.PNG " ")

5. From OCI services menu Click **Networking** -> **Virtual Cloud Networks**, a list of all VCNs will
appear. Locate your VCN, and click into the details page.

     ![](images/vcn.png " ")

     ![](images/findvcn.png " ")

6. Locate **Route Tables** on the left under **Resources**. Click into the Default Route Table and check the rule you created, click **remove** and then confirm when prompted.

     ![](images/findroutetable.png " ")

     ![](images/removerouterule.png " ")

     ![](images/confirmremoveroute.png " ")

6. Go back to details page of your VCN. Locate **Internet Gateways** on the left under **Resources**. Click Terminate on your Internet Gateway that you created. It will prompt if you are sure to delete, click **Terminate**.

     ![](images/findigw.png " ")

     ![](images/deleteigw.png " ")



6. Go back to **Virtual Cloud Networks** under **Networking**. Find your VCN in the list and click the action icon and then **Terminate**. It will ask you to scan resources in all compartments or a specific one. Choose **specific compartments** and make sure you have selected the compartment you created your resources in. Click **scan**.

     ![](images/RESERVEDIP_HOL0018.PNG " ")

     ![](images/scanvcn.PNG " ")

7. Once the scan is complete, click **Delete All** in the confirmation window. Click **Close** once VCN is deleted.

     ![](images/deletevcn.png " ")

*Congratulations! You have successfully completed the lab.*

## Learn More
* [OCI CLI Reference](https://docs.cloud.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/index.html)
* [OCI CLI GitHub repo](https://github.com/oracle/oci-cli)

## Acknowledgements
- **Author** - Flavio Pereira, Larry Beausoleil
- **Adapted by** -  Yaisah Granillo, Cloud Solution Engineer
- **Contributors** - Jaden McElvey, Technical Lead - Oracle LiveLabs Intern
- **Last Updated By/Date** - Uma Kumar, May 2024

