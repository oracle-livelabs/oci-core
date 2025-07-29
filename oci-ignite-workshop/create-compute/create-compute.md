# Create a Compute Service

## Introduction

Oracle Cloud Infrastructure Compute lets you provision and manage compute hosts, known as instances. You can launch instances as needed to meet your compute and application requirements. After you launch an instance, you can access it securely from your computer, restart it, attach and detach volumes, and terminate it when you're done with it. Any changes made to the instance's local drives are lost when you terminate it. Any saved changes to volumes attached to the instance are retained.

Be sure to review [Best Practices for Your Compute Instance](https://docs.cloud.oracle.com/iaas/Content/Compute/References/bestpracticescompute.htm) for important information about working with your Oracle Cloud Infrastructure Compute instance.

Estimated Time: 20 minutes

### Objectives
In this lab, you will:
- Create a compute instance
- Connect to the compute instance

### Prerequisites

Your **<font color="red">Oracle Cloud Account</font>** - During this workshop we will create a basic environment to get you started with Oracle Cloud.


## Task 1: Create a Compute Instance
Oracle Cloud Infrastructure  offers both Bare Metal and Virtual Machine instances:

- **Bare Metal**  - A bare metal compute instance gives you dedicated physical server access for highest performance and strong isolation.
- **Virtual Machine**  - A Virtual Machine (VM) is an independent computing environment that runs on top of physical bare metal hardware. The virtualization makes it possible to run multiple VMs that are isolated from each other. VMs are ideal for running applications that do not require the performance and resources (CPU, memory, network bandwidth, storage) of an entire physical machine.

An Oracle Cloud Infrastructure VM compute instance runs on the same hardware as a Bare Metal instance, leveraging the same cloud-optimized hardware, firmware, software stack, and networking infrastructure.

1. Click the **Navigation Menu** in the upper left. Navigate to **Compute**, and select **Instances**.

	![Navigate to Compute Instances](images/compute-instances.png)


2. Select the *Sandbox* Compartment that you created in *"Create a Compartment" Lab*. Then click **Create Instance**. We will launch a VM instance for this lab.

 
  ![Click on Create instance](images/create-compute1.png)



3. The *Create Compute Instance* wizard will launch.
    Enter **Sandbox-VM** as the name of the server. 
        
   
    ![Fill the Name of the Instance](images/create-compute2.png)
       
4. Click *Change Shape* to choose a VM shape.


    ![Change Shape of the instance](images/create-compute4.png)

5. Select *AMD Rome E4 Flex*, then select **1** as number of OCPUs, and **16 GB** as the amount of memory, and click **Select Shape**.

    ![Select AMD E4 Flex shape](images/create-compute-shapes.png)

    
    ![Select 1 OCPU and 16GB of memory](images/create-compute-shapes-1.png)

7. Click on **Next**. The Security section does not need any changes. Click **Next** again. In the Networking section, most of the defaults are perfect for our purposes. However, you will need to scroll down and select the **Assign a public IPv4 address** option.
 
    ![Assign a public IPv4 address ](images/create-compute4b.png)

    >**Note:** You need a public IP address, so that you can SSH into the running instance later in this lab.

8. Scroll down to the **Add SSH keys** area of the page. Select **Paste public keys** and paste the SSH key that you created earlier in ***Generate SSH Keys*** Lab. Click on **Next**. The Storage options remain unchanged. Click again on **Next**. Review the information and press the **Create** button to create your instance.

    ![Paste the content public key](images/ssh-keys.png)

    Launching an instance is simple and intuitive with few options to select. The provisioning of the compute instance will complete in less than a minute, and the instance state will change from *PROVISIONING* to *RUNNING*.

9. Once the instance state changes to *RUNNING*, you can SSH to the Public IP address of the instance. The Public IP address is noted under *Security* on the Instance information page. 

    
    ![Copy the Public IP address](images/public-ip.png " ")

## Task 2: Connect to the Instance 

>**Note**: You may need to log in as the *admin* user to use cloud shell.

1. To connect to the instance, use Cloud Shell and enter the following command:

    >**Note:** For Oracle Linux VMs, the default username is **opc**

    ```
    <copy>ssh -i <private_ssh_key> opc@<public_ip_address></copy>
    ```

    If you receive the message "Are you sure you want to continue connecting (yes/no)?", please write **yes**

    ![Connect to the machine using the command](images/ssh.png)



## Task 3 (Optional): Create a notification

The Observability and Management platform in Oracle Cloud Infrastructure (OCI) provides a suite of services that support cross stack visibility and rapid performance insights for any technology, deployed anywhere.
Oracle Cloud Infrastructure Monitoring for the Observability and Management platform is used to track the performance of the resources in your tenancy.

Today, we will create a **Notification** that will monitor our network activity.

1. Click the **Navigation Menu** in the upper left. Navigate to **Compute**, and select **Instances**.

	![Navigate to Compute Instances](images/compute-instances.png)


2. Select the *Sandbox-VM* instance that you created in previous steps. Then click **Monitoring**.

 
  ![Click on Monitoring](images/compute-monitoring.png)



3. Scroll until you see **Notifications** and click on **Create Notification** 
        
   
    ![Find Notification](images/compute-notification.png)

4. Click on **Template selection** and select **Average network transmit bytes are 0 over the past 5 minutes** 
        
   
    ![Select template](images/notification-template.png)

5. Provide a name for the **Alarm** and name it **Sandbox Alarm** 

    ![Provide a name for the Alarm](images/notification-alarm.png)

6. Create a new **topic** and name it **Sandbox Topic** 

    ![Create a new topic](images/notification-topic.png)

7. Add the email of the user that you want to be notified then click on **Create notification**. 

    ![Create Notification](images/notification-create.png)

_Congratulations! You have successfully completed the lab._

## Acknowledgements

- **Author** - Rajeshwari Rai, Prasenjit Sarkar 
- **Contributors** - Arabella Yao, Rajeshwari Rai, Prasenjit Sarkar, Cristian Manea, Radu Chiru, Alexandra Iancu, Radu Birsan
- **Last Updated By/Date** - Birsan Radu, Alexandra Iancu, Jully 2025