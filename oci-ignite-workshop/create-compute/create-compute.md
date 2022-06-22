# Create a Compute Service

## Introduction

Oracle Cloud Infrastructure Compute lets you provision and manage compute hosts, known as instances. You can launch instances as needed to meet your compute and application requirements. After you launch an instance, you can access it securely from your computer, restart it, attach and detach volumes, and terminate it when you're done with it. Any changes made to the instance's local drives are lost when you terminate it. Any saved changes to volumes attached to the instance are retained.

Be sure to review [Best Practices for Your Compute Instance](https://docs.cloud.oracle.com/iaas/Content/Compute/References/bestpracticescompute.htm) for important information about working with your Oracle Cloud Infrastructure Compute instance.
### **<font color="red">Prerequisites</font>**


Please view this workshop’s LiveLabs registration page to see which environments are supported during this event. 

If you prefer to use your own tenancy (**recommended**) please have your **username** and **password** ready to login for the event.

[](youtube:09kahbIF0Ew)

Estimated Time: 30 minutes

### Objectives
In this lab, you will:
- Create a compute instance
- Connect to the compute instance




## Task 1: Create a Compute Instance
Oracle Cloud Infrastructure  offers both Bare Metal and Virtual Machine instances:

- **Bare Metal**  - A bare metal compute instance gives you dedicated physical server access for highest performance and strong isolation.
- **Virtual Machine**  - A Virtual Machine (VM) is an independent computing environment that runs on top of physical bare metal hardware. The virtualization makes it possible to run multiple VMs that are isolated from each other. VMs are ideal for running applications that do not require the performance and resources (CPU, memory, network bandwidth, storage) of an entire physical machine.

An Oracle Cloud Infrastructure VM compute instance runs on the same hardware as a Bare Metal instance, leveraging the same cloud-optimized hardware, firmware, software stack, and networking infrastructure.

1. Click the **Navigation Menu** in the upper left. Navigate to **Compute**, and select **Instances**.

	![](images/compute-instances.png)


2. Select the *Workshop* Compartment that you created in *"Create a Compartment" Lab*. Then click **Create Instance**. We will launch a VM instance for this lab.

  ![](images/create-compute1.png)



3. The *Create Compute Instance* wizard will launch.
    Enter **workshop-instance** as the name of the server. 
        
    ![](images/create-compute2.png)
       
4. Click *Change Shape* to choose a VM shape.

    ![](images/create-compute-livelabs-3.png)

5. Select *AMD Rome*, then select **1** as number of OCPUs, and **16 GB** as the amount of memory, and click **Select Shape**.

    ![](images/create-compute4.png)

7. In the Networking section, most of the defaults are perfect for our purposes. However, you will need to scroll down and select the **Assign a public IPv4 address** option.
 
    ![](images/create-compute4b.png)

    >**Note:** You need a public IP address, so that you can SSH into the running instance later in this lab.

8. Scroll down to the **Add SSH keys** area of the page. Select **Paste public keys** and paste the SSH key that you created earlier in ***Generate SSH Keys*** Lab. Press the **Create** button to create your instance.

    ![](images/ssh-keys.png)

    Launching an instance is simple and intuitive with few options to select. The provisioning of the compute instance will complete in less than a minute, and the instance state will change from *PROVISIONING* to *RUNNING*.

9. Once the instance state changes to *RUNNING*, you can SSH to the Public IP address of the instance. The Public IP address is noted under *Instance Access*.

    
    ![](images/public-ip.png " ")

## Task 2: Connect to the Instance 

>**Note**: You may need to log in as the *admin* user to use cloud shell.

1. To connect to the instance, use Cloud Shell and enter the following command:

    >**Note:** For Oracle Linux VMs, the default username is **opc**

    ```
    <copy>ssh -i <private_ssh_key> opc@<public_ip_address></copy>
    ```

    ![](images/ssh.png)


_Congratulations! You have successfully completed the lab._

## Acknowledgements

- **Author** - Rajeshwari Rai, Prasenjit Sarkar 
- **Contributors** - Oracle LiveLabs QA Team (Kamryn Vinson, QA Intern, Arabella Yao, Product Manager, DB Product Management)
- **Last Updated By/Date** - Cristian Manea, Radu Chiru, March 2022