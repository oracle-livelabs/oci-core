# Deploy HA Application using Load Balancers

## Introduction

In this lab, you will deploy web servers on two compute instances in Oracle Cloud Infrastructure (OCI), configured in High Availability mode by using a Load Balancer.

Estimated time: 1 hour

### About OCI Load Balancing Service

The Load Balancing Service provides automated traffic distribution from one entry point to multiple servers within your Virtual Cloud Network (VCN). The service offers a Public load balancer with a public IP address, provisioned bandwidth, and high availability. The Load Balancing Service provisions the public IP address across two subnets within a VCN to ensure accessibility even during an Availability Domain outage.

### Objectives

- Create a Virtual Cloud Network (VCN)
- Create an OCI Compute Instance
- Modify VCN
- Create Load Balancer

### Prerequisites

- Have successfully logged into Oracle Cloud console and created SSH keys in cloud shell

## Task 1: Create a Virtual Cloud Network

1. From the OCI services menu, click **Virtual Cloud Networks** under **Networking**.

	![](./images/task1.1.png " ")

2.	Select the **Compartment** assigned to you from drop-down menu on the left part of the screen under List Scope and click **Start VCN Wizard**.

	> **Note:** Ensure the correct compartment is selected under COMPARTMENT list.

	![](./images/task1.2.png " ")

3. Choose **VCN with Internet Connectivity** and click **Start VCN Wizard**.

	![](./images/task1.3.png " ")

4. Fill out the dialog box:

    - **VCN NAME**: Provide a name
    - **COMPARTMENT**: Ensure your compartment is selected
    - **VCN CIDR BLOCK**: Provide a CIDR block (10.0.0.0/16)
    - **PUBLIC SUBNET CIDR BLOCK**: Provide a CIDR block (10.0.1.0/24)
    - **PRIVATE SUBNET CIDR BLOCK**: Provide a CIDR block (10.0.2.0/24)
    - Click **Next**

	![](./images/task1.4.png " ")

5. Verify all the information and click **Create**.

	![](./images/task1.5.png " ")

	![](./images/task1.5.1.png " ")

6. This will create a VCN with following components.

    *VCN, Public subnet, Private subnet, Internet gateway (IG), NAT gateway (NAT), Service gateway (SG)*

7. Click **View Virtual Cloud Network** to display your VCN details.

	![](./images/task1.7.png " ")

	![](./images/task1.7.1.png " ")

## Task 2: Create Two Compute Instances and Install the Web Server

1. Switch to the OCI console. From the OCI services menu, click **Instances** under **Compute**.

	![](./images/task2.1.png " ")

2. Make sure you are in the same compartment as the VCN you just created and click **Create Instance**.

	![](./images/task2.2.png " ")

3. Fill out the Create compute instance dialog box:

    - **Name**: Enter a name for you instance.
	- **Create in compartment**: Choose the same compartment as the VCN you just created.
    - **Placement**: Leave the default or click on **Edit** to select an availability domain.
    - **Image and shape**: For the image, we recommend using the latest Oracle Linux available. It is the default selection. For the shape, use the default one. If you want to use a different shape, click on **Edit** to change shape.
	- **Networking:** Leave the default choices
    - **Add SSH Keys:** Choose **Paste SSH Keys** and paste the Public Key you created in Cloud Shell earlier. *Ensure when you are pasting the key in one line.*
	- **Boot Volume:** Leave the default choices
	- Click **Create**

	> **Note:**
	1. If *Service limit* error is displayed, choose a different shape from VM.Standard2.1, VM.Standard.E2.1, VM.Standard1.1, VM.Standard.B1.1 OR choose a different AD.
	2. The lab instruction places the instances on a public subnets to simplify SSH access to them. In a more secure environment, they should be placed on private subnets and accessed through a bastion server or VPN connection.

	![](./images/task2.3.png " ")

	![](./images/task2.3.1.png " ")

<!--
	**Under Configure Networking:**
    - **Virtual cloud network compartment**: Select your compartment
    - **Virtual cloud network**: Choose the VCN
    - **Subnet Compartment:** Choose your compartment.
    - **Subnet:** Choose the Public Subnet
    - **Use network security groups to control traffic** : Leave un-checked
    - **Assign a public IP address**: Check this option
-->

4.  The instance starts Provisioning and turns to green from yellow once it is Running. Repeat task 2 steps 1 - 3 to launch a **second** compute instance. Wait for instances to be in **Running** state and note down its public IP address.

	![](./images/task2.4.png " ")

	![](./images/task2.4.1.png " ")

5.  Launch the Cloud Shell if it is not running.  When running, enter the command below and enter **ls** to verify your key file exists.


    ```
    <copy>
    cd .ssh
	ls
    </copy>
    ```

	![](./images/task2.5.png " ")

	![](./images/task2.5.1.png " ")

6.  Ssh into the **first** compute instance. Enter command:

    ```
    <copy>
    bash
    ssh -i <<sshkeyname>> opc@<PUBLIC_IP_OF_COMPUTE_1>
    </copy>
    ```

    > **Note:**  User name is *opc* if you used the Oracle Linux image.

    **HINT:** If *Permission denied error* is seen, ensure you use '-i' in the ssh command. You MUST type the command, do NOT copy and paste ssh command.

	Enter **Yes** when prompted for a security message.

    ```
    $ ssh opc@<PUBLIC_IP_OF_COMPUTE_1>
    The authenticity of host '158.101.17.73 (158.101.17.73)' can't be established.
    ECDSA key fingerprint is SHA256:RN5tRSWvMoBIhK3VfTJ/EWNmU1rqWoGsdRn0QKN9hRI.
    Are you sure you want to continue connecting (yes/no/[fingerprint])?
    ```

	Verify opc@`<COMPUTE_INSTANCE_NAME>` appears on the prompt.

	![](./images/task2.6.png " ")

7. Duplicate the Oracle cloud Infrastructure tab in the browser. In the second tab of Oracle Cloud, repeat task 2 steps 5 - 6 to launch a second cloud shell window and connect via SSH into the **second** Compute instance.

    *HINT: Ensure to use the IP address of the second Compute instance in the SSH command.*

	![](./images/task2.7.png " ")

8. Go back to the first tab cloud shell for the first Compute instance and install a Web server, using the commands below:

    Install Apache HTTP Server.

    ```
    <copy>
    sudo yum -y install httpd
    </copy>
    ```

	![](./images/task2.8.png " ")

	![](./images/task2.8.1.png " ")

    Once the installation is complete, open TCP port 80 in the local firewall.

    ```
    <copy>
    sudo firewall-cmd --permanent  --add-port=80/tcp
    </copy>
    ```

    > **Note:** --add-port flag has no spaces.

    Reload the firewall to activate the rules.

    ```
    <copy>
    sudo firewall-cmd --reload
    </copy>
    ```

    Start the web server.

    ```
    <copy>
    sudo systemctl start httpd
    </copy>
    ```

    Change user privileges (become root).

    ```
    <copy>
    sudo su
    </copy>
    ```

    Create the index.html file. The content of the file will be displayed when the web server is accessed.

    ```
    <copy>
    echo 'WebServer1' >>/var/www/html/index.html
    </copy>
    ```

	![](./images/task2.8.2.png " ")

9. Bring up the SSH session for the second Compute instance and repeat commands:

    Install Apache HTTP Server.

    ```
    <copy>
    sudo yum -y install httpd
    </copy>
    ```

    Once the installation is complete, open TCP port 80 in the local firewall.

    ```
    <copy>
    sudo firewall-cmd --permanent  --add-port=80/tcp
    </copy>
    ```

    > **Note:** --add-port flag has no spaces.

    Reload the firewall to activate the rules.

    ```
    <copy>
    sudo firewall-cmd --reload
    </copy>
    ```

    Start the web server.

    ```
    <copy>
    sudo systemctl start httpd
    </copy>
    ```

    Change user privileges (become root).

    ```
    <copy>
    sudo su
    </copy>
    ```

    Create the index.html file. The content of the file will be displayed when the web server is accessed.

    ```
    <copy>
    echo 'WebServer2' >>/var/www/html/index.html
    </copy>
    ```

10. Switch back to the first OCI console window.

We now have two Compute instances with Web servers installed and a basis index.html file. Before we create the load balancer, we will need to create a new security list, route table, and subnet that the load balancer will use.

Load balancers should always reside in different subnets than your application instances. This allows you to keep your application instances secured in private subnets while allowing public Internet traffic to the load balancers in the public subnets.

## Task 3: Create Security List, Route table, and Additional Subnet

In this section, we will create a new security list. This security list will be used by the load balancer (which will be created later on). This will ensure all traffic to the two web servers is allowed.

1. From the OCI Services menu, Click **Virtual Cloud Networks** under **Networking**. This displays the list of VCNs in the current compartment.

    **HINT:** If there are multiple Networks, scroll down to locate the one you created for this lab.

	![](./images/task1.1.png " ")

2. Click on your VCN name.

	![](./images/task3.2.png " ")

3.	On the VCN Details page, click **Security Lists** under Resources, then click on **Create Security List**.

	![](./images/task3.3.png " ")

4. In the Create Security List dialog box, provide the following:

    - **Name:** Specify a name for the security list (for example, LB Security List).
	- **Create In Compartment:** Select the compartment assigned to you (if not already selected).
    - Click **Create Security List**

	![](./images/task3.4.png " ")

5.  Verify the new Security List is created.

    **We now have a Security List that will be used by the load balancer. Next, we will create a Route table that will be used by two new subnets (that will be used by the load balancer, once created).**

	![](./images/task3.5.png " ")

6. Click on **Route Tables** under Resources and click on **Create Route Table**.

	![](./images/task3.6.png " ")

7. In the Create Route Table dialog box, fill the following details:

    - **Name:** Enter a name (for example, LB Route Table).
    - **Create In Compartment:** This field defaults to your current compartment. Make sure the correct Compartment is selected.
	- Click **+ Another Route Rules**
    - **Target Type:** Click on the drop-down to select **Internet Gateway**.
    - **Destination CIDR Block:** 0.0.0.0/0
    - **Compartment:** Make sure the correct Compartment is selected
    - **Target Internet Gateway:** Click on the drop-down to select the Internet Gateway for your VCN.
	- Click **Create**

	![](./images/task3.7.png " ")

    ![](./images/task3.7.1.png " ")

8. Ensure the new route table appears in the list (Under Create Route Table).

    **We now have a route table that allows all traffic. Next, we will attach this route table to two new subnets that we will create (This subnet will be used by the Load Balancer).**

	![](./images/task3.8.png " ")

9. Assuming you are still on your VCN details page,
create Load Balancer subnet. Click **Subnets** under Resources and click **Create Subnet**.

	![](./images/task3.9.png " ")

10. In the Create Subnet dialog box, fill the following details:

    - **Name:** Enter a subnet name (for example, LB-Subnet).
	- **Create In Compartment:** This field defaults to your current compartment. Make sure the correct Compartment is selected.
    - **Subnet Type:** Select Regional
    - **CIDR Block:** Enter 10.0.4.0/24
    - **Route Table:** From the drop-down, select the Route Table you created earlier.
    - **Subnet Access**: select Public Subnet.
    - **Dhcp Options Compartment:** From the drop-down, select the **Default DHCP Options for your vcn**.
    - **Security List Complartment:** From the drop-down, select the Security List you created earlier.
	- Leave all other options as default and click **Create Subnet**.

    ![](./images/task3.10.png " ")

	![](./images/task3.10.1.png " ")

11. Ensure the new subnet appears in the list (Under Create Subnet).

	![](./images/task3.11.png " ")

## Task 4: Create Load Balancer and Update Security List

When you create a load balancer, you choose its dynamic shape (maximum throughput) or flexible shape (user-defined range with minimum and maximum limits). If you assign it to use a regional subnet, it will set the fail-over load balancer on a different Availability Domain (if available), improving the high availability.

1. From the OCI Services menu, click **Load Balancers** under **Networking**.

	![](./images/task4.1.png " ")

2. Make sure you are in the same compartment as the VCN you created, click **Create Load Balancer**.

	![](./images/task4.2.png " ")

3. Choose **Load Balancer** and click Create Load Balancer.

	![](./images/task4.3.png " ")

4. In the Create Load Balancer dialog box, provide the following details:

    Under **Add Details** section:

    - **Load Balancer Name:** Enter a name for your load balancer.
    - **Choose visibility type:** Public
	- **Assign a public IP address:** Choose **Ephemeral IP Address**
    - **Shapes:** Choose **Flexible Shapes**
    - **Choose the minimum bandwidth**: 10Mbps
    - **Virtual Cloud Network:** Choose your Virtual Cloud Network
    - **Subnet:** Choose the Regional Subnet we created (10.0.4.0 in this lab)
	- Click **Next** or **Choose Backends**

    ![](./images/task4.4.png " ")

	![](./images/task4.4.1.png " ")

5. In the Create Load Balancer dialog box, Choose Backends section, provide the following details:

    - **Specify a Load Balancing Policy:** Choose **Weighted Round Robin**
    - Click **Add Backend**, choose the two compute instances created earlier and click **Add Selected Backends**

    **Specify Health Check Policy**

    - **Protocol:** HTTP
    - **Port:** Enter 80
    - **URL PATH (URI):** /
	- Leave other options with the default values
	- Click **Next** or **Configure Listener**

	![](./images/task4.5.png " ")

	![](./images/task4.5.1.png " ")

	![](./images/task4.5.2.png " ")

6. In the Create Load Balancer dialog box, Configure Listener section, provide the following details:

    - **Specify the type of traffic your listener handles:** HTTP
    - **Specify the port your listener monitors for ingress traffic:** 80
	- Leave other options with the default values
	- Click **Next** or **Manage Logging**

	![](./images/task4.6.png " ")

7. In the Create Load Balancer dialog box,  Manage Logging section, leave the defaults and click **Submit**.

	![](./images/task4.7.png " ")

8. Wait for the load balancer to become active and then note down it’s Public IP address.

    **We now have a load balancer that will manage the subnet we created earlier.**

	![](./images/task4.8.png " ")

	![](./images/task4.8.1tas.png " ")

9. From the OCI Services menu, Click **Virtual Cloud Networks** under **Networking**.

	![](./images/task1.1.png " ")

10. Locate the VCN you created and click on the VCN name to display the VCN detail page.

	![](./images/task3.2.png " ")

11. Click **Security Lists** and locate the Load Balancer Security List created earlier and click on it.

	![](./images/task4.11.png " ")

12. In the Security List Details page, click **Add Ingress Rule**.

	![](./images/task4.12.png " ")

13. In the Add Ingress Rules dialog box,enter the following ingress rule. Ensure to leave STATELESS flag un-checked.

	- **Source Type**: CIDR
	- **Source CIDR**: Enter 0.0.0.0/0.
	- **IP Protocol**: Select TCP.
	- **Source Port Range**: All.
	- **Destination Port Range**: Enter 80 (the listener port).
	- Click **Add Ingress Rule**.

	![](./images/task4.13.png " ")

14. Click **Egress Rule** under Resources. Click **Add Egress Rule**.

	![](./images/task4.14.png " ")

15. In the Add Egress Rules dialog box, enter the following Egress rule; Ensure to leave STATELESS flag un-checked.

    - **Destination Type**: CIDR
    - **Destination CIDR**: 0.0.0.0/0
    - **IP Protocol**: Select TCP.
    - **Destination Port Range**: All.
	- Click **Add Egress Rule**.

	![](./images/task4.15.png " ")

16. Navigate back to you VCN details page. Click **Security Lists** under Resources and locate the **Default Security List** of your VCN.

	![](./images/task4.16.png " ")

17. In the Default Security List details page, click **Add Ingress Rule**.

	![](./images/task4.17.png " ")

18. In the Add Ingress Rules dialog box, add the below 2 Rules for Ingress. Ensure to leave STATELESS flag un-checked for both the Ingress rules.

    **First Rule**

    - Source Type: CIDR
    - Source CIDR: 10.0.4.0/24
    - IP Protocol: Select TCP.
    - Source Port Range: All
    - Destination Port Range: 80
	- Click **+Additional Ingress Rule**

    **Second Rule**

    - Source Type: CIDR
    - Source CIDR: 10.0.5.0/24
    - IP Protocol: Select TCP
    - Destination Port Range: 80
	- Click **Add Ingress Rule**

	![](./images/task4.18.png " ")

	![](./images/task4.18.1.png " ")

We now have the set-up configured with 2 Compute instances running http server with an index.html file, Load Balancer with all relevant policies and components.

We will now test the Load Balancer functionality (load balance using round-robin). In case one of the http server in the High Availability configuration is un-available, Load Balancer will automatically route the traffic to the available http server.

> **Note:** Be sure to take note of the "Health" field in the Networking > Load Balancers dashboard. If the health is "Critical," the load balancer may not work as intended, and the best course of action may be to create a new one. This is likely the result of something being misconfigured, and it should only happen rarely.

## Task 5: Verify High Availability of HTTP Servers

In this section, we will access the two Web servers configured earlier using Load Balancer’s Public IP address and demonstrate Load Balancer’s ability to route traffic on round-robin basis (per the policy configured). In case one of the web server becomes unavailable the web content will be available via the second server (High Availability)

1. Open a web browser and enter load balancer's public IP address.

	![](./images/task5.1.png " ")

2. Verify the text in index.html file on the two servers (either WebServer1 or WebServer2) is displayed.

	![](./images/task5.2.png " ")

3. Refresh the browser multiple times and Observer Load Balancer Balancing traffic between the 2 web servers.

   ![](./images/task5.3.png " ")

    > **Note:** In case one of the servers goes down the Application will be accessible via Load Balancer’s Public IP address.

This lab is not intended to test failover and recovery of backend servers. Users can test that functionality at their own discretion. Any troubleshooting in case any issue is encountered it is out of the scope of this lab.

## Task 6: Delete the Resources

Delete Load Balancer and associated components:

1. From the OCI Services menu, click **Load Balancers**, under **Networking**.

	![](./images/task4.1.png " ")

2. Make sure you are in the same compartment as the VCN you created, click on your Load Balancer Name.

	![](./images/task6.2.png " ")

3. Click **Terminate** and click **Terminate** in the Confirm Window. Wait for the termination to be completed.

	![](./images/task6.3.png " ")

	![](./images/task6.3.1.png " ")

4. From OCI services menu, click **Instances** under Compute.

	![](./images/task2.1.png " ")

5. Make sure you are in the same compartment as the VCN you created, locate the first compute instance and click on its name.

	![](./images/task6.5.png " ")

6. Click on the **More Actions** and then select **Terminate**. Make sure check the **Permanently delete the attached Boot Volume** and then click **Terminate instance**.

	![](./images/task6.6.png " ")

	![](./images/task6.6.1.png " ")

7. Repeat steps **5** and **6** to terminate the second compute instance. Once the instances are terminated, the color changes to grey from yellow. Wait for both instances termination to be completed.

	![](./images/task6.7.png " ")

	![](./images/task6.7.1.png " ")

8. From OCI services menu, click **Virtual Cloud Networks** under **Networking**.

	![](./images/task1.1.png " ")

9. Make sure you are in correct compartment. From the list of all VCNs, locate your VCN and click on its name.

	![](./images/task3.2.png " ")

10. Click **Terminate** and the click **Terminate All** in the Confirmation window. Click **Close** once VCN is deleted.

   ![](./images/task6.10.png " ")

   ![](./images/task6.10.1.png " ")

   ![](./images/task6.10.2.png " ")

*Congratulations! You have successfully completed the lab.*

## Learn More

1. [OCI Training](https://cloud.oracle.com/en_US/iaas/training)
2. [Familiarity with OCI console](https://docs.cloud.oracle.com/en-us/iaas/Content/GSG/Concepts/console.htm)
3. [Overview of Networking](https://docs.cloud.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
4. [Familiarity with Compartments](https://docs.cloud.oracle.com/en-us/iaas/Content/GSG/Concepts/concepts.htm)
5. [Connecting to a compute instance](https://docs.cloud.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm)

## Acknowledgements

- **Authors** - Flavio Pereira, Larry Beausoleil
- **Adapted by** -  Yaisah Granillo, Cloud Solution Engineer
- **Contributors** - Anoosha Pilli, Product Manager, Oracle Database
- **Last Updated By/Date** - Anoosha Pilli, November 2021

