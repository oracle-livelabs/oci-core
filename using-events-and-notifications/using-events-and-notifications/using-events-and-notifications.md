# Using Events and Notification

## Introduction

The Oracle Cloud Infrastructure Notifications service broadcasts messages  to distributed components through a publish-subscribe pattern, delivering secure, highly reliable, low latency and durable messages for applications hosted on Oracle Cloud Infrastructure and externally.

The Notifications service enables you to set up communication channels for publishing messages using topics  and subscriptions . When a message is published to a topic, the Notifications service sends the message to all of the topic's subscriptions.
In this lab we will verify notifications when a compute instance is launched and deleted.

Estimated time: 30 minutes

## Task 1: Sign in to OCI Console and configure Notification and Event

1. First, we will create a Notification topic and subscribe to this topic. Click the **Navigation Menu** in the upper left, navigate to **Developer Services**, and select **Notifications**.

	![Navigate to Notifications](https://raw.githubusercontent.com/oracle/learning-library/master/common/images/console/developer-application-notification.png " ")

2. Click **Create Topic** and fill out the dialog box:

      - **Name**: Provide a name
      - **Description** (Optional): Provide a description

    ![Click create topic](images/click_create_topic.png " ")  

3. Click **Create**.

    ![Create topic](images/create_topic.png " ")  

4. Once the topic state changes to **Active**, Click the topic Name. 

    ![Click the topic name](images/click_topic.png " ")

5. Click **Create Subscription** and fill out the dialog box:

      - **PROTOCOL**: Email
      - **EMAIL**: Provide your email id

    ![Click create subscription](images/create_subscription.png " ")

6. Click **Create**.

    ![Create subscription](images/create_subscription2.png " ")

7. Click the Subscription OCID.

    ![Click the Subscription OCID](images/create_subscription3.png " ")

8. The subscription details screen will be displayed with subscription status showing **Pending**.

    ![Subscription pending](images/create_subscription4.png " ")

9. Check the email account you specified and click the verification link for this subscription. Switch back to OCI console window and verify the subscription status changed to **Active**. You may need to refresh your browser.

    ![Subscription active](images/create_subscription5.png " ")

10. You are now subscribed to a Notification topic. Next we will configure Events that will publish messages to this Notification topic.

11. From the OCI Services menu, click **Observability & Management** > **Rules**.

    ![Navigate to rules](images/create_rule.png " ")

12. Click **Create Rule**, Fill out the dialog box:

    - **DISPLAY NAME** : Provide a name
    - **DESCRIPTION** : Provide a description

    Under **Rule Conditions**

    - Ensure **Event Type** is selected
    - **SERVICE NAME**: Compute
    - **EVENT TYPE** : Choose these 4 Types from the drop down menu. **Instance-Launch Begin**, **Instance-Launch End**, **Instance-Terminate Begin**, **Instance-Terminate End**

    Under **Actions**

    - **ACTION TYPE**: Notifications
    - **NOTIFICATIONS COMPARTMENT**: Choose your compartment
    - **TOPIC**: Choose the topic created earlier

13. Click **Create Rule**.

    ![Create rule](images/create_rule2.png " ")


We have now configured Notification service and tied events to it with a specific compartment. When a new compute instance is launched or terminated an email notification will be sent to the email address specified.

## Task 2: Create VCN

1. From the OCI Services menu, click **Networking** > **Virtual Cloud Networks**. 

    ![Navigate to Virtual Cloud Networks](images/vcn.png " ")

2. Select the compartment assigned to you from the drop down menu on the left part of the screen under Networking and click **Start VCN Wizard**.

    **NOTE:** Ensure the correct Compartment is selected under COMPARTMENT list.

    ![Click Start VCN Wizard](images/vcn2.png " ")    

3. Click **Create VCN with Internet Connectivity** and click **Start VCN Wizard**.

    ![Start VCN Wizard](images/vcn3.png " ")

4. Fill out the dialog box:

       - **VCN NAME**: Provide a name
       - **COMPARTMENT**: Ensure your compartment is selected
       - **VCN CIDR BLOCK**: Provide a CIDR block (10.0.0.0/16)
       - **PUBLIC SUBNET CIDR BLOCK**: Provide a CIDR block (10.0.1.0/24)
       - **PRIVATE SUBNET CIDR BLOCK**: Provide a CIDR block (10.0.2.0/24)
       - Click **Next**

    ![Create VCN with Internet Connectivity](images/vcn4.png " ")       

5. Verify all the information and click **Create**.

    ![Click create](images/vcn5.png " ")

6. This will create a VCN with following components.

    *VCN, Public subnet, Private subnet, Internet gateway (IG), NAT gateway (NAT), Service gateway (SG)*

7. Click **View Virtual Cloud Network** to display your VCN details.

    ![Click View Virtual Cloud Network](images/vcn6.png " ")

## Task 3: Create compute instance and verify notification

1. From the OCI services menu, click **Compute** > **Instances**.

   ![Navigate to Instances](images/compute.png " ")

2. Click **Create Instance**. Fill out the dialog box:

      - **Name your instance**: Enter a name
      - **Create in compartment**: Select your compartment
      - **Image and shape**: For the image, we recommend using the Latest Oracle Linux available. For the shape, select a VM shape

      **In the Networking section**
      - **Virtual cloud network**: Choose the VCN
      - **Subnet:** Choose the Public Subnet under **Public Subnets**
      - **Use network security groups to control traffic** : No
      - **Assign a public IPv4 address**: Check this option

   ![Fill out the dialog box](images/compute2.png " ")

    - **Boot Volume:** Leave the default
    - **Add SSH Keys:** Leave empty

3. Click **Create**.

    **NOTE:** If 'Service limit' error is displayed choose a different shape from VM.Standard2.1, VM.Standard.E2.1, VM.Standard1.1, VM.Standard.B1.1  OR choose a different AD.

   ![Create compute instance](images/compute3.png " ")

4. Switch to your email account and verify an event indicating compute instance launch was received.

5. Wait for Instance to be in **Running** state.

   ![Instance in Running state](images/compute4.png " ")

6. Switch to your email account and verify an event indicating compute instance create was received.

## Task 4: Delete the resources

1. From the OCI services menu, click **Compute** > **Instances**.

   ![Navigate to Instances](images/compute.png " ")

2. Locate your compute instance, click the Action icon and then **Terminate**.

   ![Terminate compute instance](images/terminate_compute.png " ")

3. Make sure Permanently delete the attached Boot Volume is checked, Click **Terminate Instance**. Wait for instance to fully Terminate.

   ![Click Terminate Instance](images/terminate_compute2.png " ")

4. Switch to your email account and verify an event indicating compute instance terminate was received.

5. Once the compute instance is fully terminated another email notification will arrive.

6. From the OCI services menu, click **Networking** > **Virtual Cloud Networks**. A list of all VCNs will appear.

   ![Navigate to Virtual Cloud Networks](images/vcn.png " ")

7. Locate your VCN , Click Action icon and then **Terminate**. Click **Terminate All** in the Confirmation window. Click **Close** once VCN is deleted.

   ![Terminate Virtual Cloud Network](images/terminate_vcn.png " ")

8. Click the **Navigation Menu** in the upper left, navigate to **Developer Services**, and select **Notifications**.

	![Navigate to Notifications](https://raw.githubusercontent.com/oracle/learning-library/master/common/images/console/developer-application-notification.png " ")

9. Click your Topic name. 

   ![Click Topic name](images/delete_topic.png " ")

10. Click **Delete**.

   ![Delete Topic](images/delete_topic2.png " ")

11. From the OCI Services menu, click **Observability & Management** > **Rules**.

   ![Navigate to rules](images/create_rule.png " ")

12. Click the action icon and click **Delete**. 

   ![Delete rule](images/delete_rule.png " ")

13. In the dialog box type **DELETE** and click **Delete**.

   ![Type DELETE and click Delete](images/delete_rule2.png " ")

**Congratulations! You have successfully completed the lab.**

## Acknowledgements

- **Author** - Flavio Pereira, Larry Beausoleil
- **Adapted by** -  Yaisah Granillo, Cloud Solution Engineer
- **Contributors** - Arabella Yao, Product Manager, DB Product Management
- **Last Updated By/Date** - Kamryn Vinson, December 2021

