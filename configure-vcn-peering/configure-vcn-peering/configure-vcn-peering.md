# Configure Virtual Cloud Network Peering

## Introduction

Local VCN peering is the process of connecting two VCNs in the same region and tenancy, so that their resources can communicate using private IP addresses without routing the traffic over the Internet or through your on-premises network. Without peering, a given VCN would need an Internet gateway and public IP addresses for instances that need to communicate with another VCN.

Estimated Time: 25 minutes

### Objectives
- Create Local Peering Gateways in each VCN
- Configure Route Tables and Security Lists to establish local peering connection between two VCNs
- Verify the peering connection

### Prerequisites
* An Oracle Cloud Account - please view this workshop's LiveLabs landing page to see which environments are supported
* SSH keys created in Oracle Cloud Shell
* Successful completion of the previous lab tasks

## Task 1: Configure Local Peering

>**Note:** Oracle Cloud Infrastructure (OCI) UI is being updated, thus some screenshots in the instructions may be different from the actual UI.

1. From the OCI Services menu, click **Networking** -> **Virtual Cloud Networks**. 

    ![Click Virtual Cloud Network](https://oracle-livelabs.github.io/common/images/console/networking-vcn.png)

2. Click the **Marketing-VCN** link on the Virtual Cloud Networks dashboard.

3. Click the **Gateways** tab and scroll down to **Local Peering Gateways**. Click **[Create Local Peering Gateway]**.

4. Fill out the dialog box:

    - **Name**: Provide a name like Marketing-to-Mgmt-lpg01
    - **Create in Compartment**: Select your compartment

    ![Create Local Peering Gateway for the First VCN](images/lpg01.png " ")

5. Click **Create Local Peering Gateway**.

6. Navigate to the **Routing** tab and click on the **Default Route Table for Marketing-VCN**.

7. Click the **Route Rules** tab and click **[Add Route Rules]**

8. Click **Add Route Rules** and add the following rule:

    - Target Type: **Local Peering Gateway**
    - Destination CIDR Block: **10.0.0.0/24**
    - Compartment:  Make sure the correct Compartment is selected
    - Target Local Peering Gateway: Select the Local Peering Gateway of the first VCN

    ![Add Local Peering Gateway Route Rule for the First VCN](images/route-rule1.png " ")

    >NOTE: You'll notice that the destination CIDR is the IP range that we assigned to the subnet rather than the IP range for the entire VCN. This is intentional to ensure that we only route traffic to the one, specific subnet. There may be other subnets with resources that we do not need to access via the Local Peering connection. We limit the scope of the connectivity for added security.

9. Click **[Add Route Rules]**

10. Click the **Security** tab, then click **Default Security List for `<FIRST_VCN_NAME>`**.

    ![Configure Security List for the First VCN](images/security-list.png " ")

11. Click the **Security rules** tab, then click **Add Ingress Rules**. Enter the following Ingress Rule. Ensure to leave **Stateless** flag un-checked.

    - Source CIDR: **172.16.0.0/24**
    - IP Protocol: **ICMP**

    ![Add Ingress Rule for the First VCN](images/ingress-rule1.png " ")

12. Click **Add Ingress Rules**.

13. Return to the **Virtual Cloud Networks** dashboard and repeat steps 2 through 12 to create the local peering gateway, new route rule, and Security Ingress Rule for the Management-VCN network.

    - **LPG Name**: Mgmt-to-Marketing-lpg02
    - **Route Rule - Destination CIDR Block**: 172.16.0.0/24
    - **Target Local Peering Gateway**: Mgmt-to-Marketing-lpg02
    - **Security Ingress Rules**: Protocol: ICMP || Source: 172.16.0.0/24

    >NOTE: What is ICMP? It is a network layer protocal, generally used by diagnostic tools like *ping* and *tracert* that verify connectivity between network resources.

14. Navigate to the Management-VCN configuration page. Click the **Gateways** tab and scroll down to Local Peering Gateways. Click the **...** (ellipses) to the right of the listed LPG. Click **Establish Peering Connection**.

    ![Establish Peering Connection for the First VCN](images/configure-lpg01.png " ")

15. Fill out the dialog box:

    - Specify the Local Peering Gateway: **Browse Below** (to browse the list of available gateways)
    - Virtual Cloud Network Compartment: Select your compartment
    - Virtual Cloud Network: Choose the **second VCN** (Gateway1 needs to pair with Gateway2 that is in the second VCN)
    - Local Peering Gateway Compartment: Select your compartment
    - Unpeered Peer Gateway: Choose the second local peering gateway

    Click **Establish Peering Connection**.

    ![Establish Peering Connection to the Second VCN](images/establish-peering-connection.png " ")

16. Verify the Local Peering Gateway shows Status as *Peered* and Peered information is correct.

    ![Verify Status of Local Peering Gateway](images/verify-peering.png " ")

**We now have two VCNs with one compute instance in each VCN. These VCNs have been connected using Local Peering Gateways with specific routes to enable traffic flow between the two subnets that were created. Any instance in the Management VCN subnet can reach any instance in the other Marketing VCN Subnet, and vice versa. Next we will test the connectivity.**

## Task 2: SSH to compute instance and test VCN Peering

1. Using Cloud Shell, enter the following command:

    ```bash
    <copy>
    cd ~/.ssh/
    </copy>
    ```

2. Enter **ls** and verify that your ssh key file exists.

3. Enter command:

    ```
    <copy>
    ssh -i <sshkeyname> opc@<PUBLIC_IP_OF_MGMT_INSTANCE>
    </copy>
    ```

    **We will SSH to the first compute instance**.

    >Note: User name is opc. This is required to log into the host along with your SSH private key.

    **HINT:** If *Permission denied error* is seen, ensure you are using '-i' in the ssh command.

4. Enter **yes** when prompted for security message.

5. Verify opc@`<COMPUTE_INSTANCE_NAME>` appears on the prompt.

    ![SSH to the First Compute Instance](images/ssh.png " ")

6. Enter command:

    ```
    <copy>
    ping <PRIVATE_IP_OF_SECOND_COMPUTE_INSTANCE>
    </copy>
    ```

    >**Note:** Use Private IP of the compute instance that you are not connected to.

    **Verify the ping is successful.**
    ![Verify Peering is Successful](images/peering-success.png " ")

    If ping is successful, then we have successfully created VCN peering across two different VCNs.

7. If the ping was not successful, review the instructions above to verify the following:

    - Both VCN Security Ingress rules have been configured properly. Mgmt VCN Ingress rule should have source 172.16.0.0/24. Marketing VCN Ingress rule should have source 10.0.0.0/24.
    - Both VCN Route Rules have a Local Peering route that points to the target CIDR block for the other VCN.

8. You may exit Cloud Shell.

*Congratulations! You have successfully completed the lab. If you would like assistance removing the resources created in this workshop, you may proceed to Lab 3.*

## Acknowledgements

- **Author** - Umair Siddiqui, Larry Beausoleil
- **Adapted by** -  Yaisah Granillo, Cloud Solution Engineer
- **Last Updated By/Date** - Eli Schilling, Cloud Architect, Lab Experience Team, February 2026