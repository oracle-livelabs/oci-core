# Identity and Access Management

## Introduction

Oracle Cloud Infrastructure (OCI) Identity and Access Management (IAM) Service lets you control who has access to your cloud resources. You control the types of access a group of users has and to which specific resources. Moreover, with the inclusion of Identity Domains, OCI IAM and Oracle IDCS were unified into a single cloud service.

The purpose of this section is to give you a recap of the content from the first Ignite session. You will encounter an overview of the IAM Service components with Identity domains and an example scenario to help you understand how they work together.

Estimated time: 5 minutes

### Objectives


In this section, you will revise the Ignite knowledge:

- Create a compartment
- Create a user
- Create a group
- Create a policy associated to the group
- Add user to the group

### Prerequisites

* This lab requires completion of the **Get Started** section in the Contents menu on the left.

* Your **<font color="red">Oracle Cloud Account</font>** - During this workshop we will create a basic environment for you to use on your tenancy.


## Task 1: Create a Compartment

A compartment is a collection of cloud assets, like compute instances, load balancers, databases, etc. By default, a root compartment was created for you when you created your tenancy (i.e. when you registered for the trial account). It is possible to create everything in the root compartment, but Oracle recommends that you create sub-compartments to help manage your resources more efficiently.

1. Click the **Navigation Menu** in the upper left, navigate to **Identity & Security**, and select **Compartments**.

 ![Navigation Menu](images/id-compartments.png " ")

1. Click **Create Compartment**.
   ![Create a compartment](images/create-compartment3.png)

1. Name the compartment **Demo** and provide a short description. Be sure your root compartment is shown as the parent compartment. Press the blue **Create Compartment** button when ready.

   ![Create compartment](images/compartment-details2.png)

1. You have just created a compartment for all of your work in this *Workshop*.

##  Task 2: Manage Users, Groups, and Policies to Control Access

A user's permission to access services comes from the _groups_ to which they belong. The permissions for a group are defined by policies. Policies define what actions members of a group can perform, and in which compartments. Users can access services and perform operations based on the policies set for the groups of which they are members.

We'll create a user, a group, and a security policy to understand the concept.

In 2022, OCI IAM introduced Identity Domains. An identity domain is a container for managing users and roles, federating and provisioning users, secure application integration through Oracle Single Sign-On (SSO) configuration, and OAuth administration.

1. Click the **Navigation Menu** in the upper left. Navigate to **Identity & Security** and select **Domains**

For IAM with Identity Domains, what was identified before as IAM users and groups, now is under the default domain.

   ![Navigate to Domains](images/id-domains.png)

1. Select the default domain

   ![Select the Default Domain](images/id-domains-default.png)

1. Select **Groups**

   ![Navigato to Groups](images/id-domains-groups.png)

1. Click **Create Group**.

   In the **Create Group** dialog box, enter the following:

     - **Name:** Enter a unique name for your group, such as **oci-group**   
     - **Description:** Enter a description, such as **New group for oci users**
     - Click **Create**

   ![Create Group](images/id-domains-create-group.png) 

1. Click your new group to display it. Your new group is displayed.

   ![New group is shown](images/id-domains-group-detail.png)

1. Create a New User

   a) In the breadcrumb, click on **Default Domain**

   ![Select Default domain in the bread crumb](images/id-domains-bc-default-domain.png)

   You can also click the **Navigation Menu** in the upper left, navigate to **Identity & Security**, and select **Domains**, select the default domain and then go to **Users**

   b) Select **Users**

   ![Select Users](images/id-domains-users.png)

   c) Click **Create User**.

   In the **Create User** dialog box, enter the following:

      - **First Name** - your first name
      - **Last NameName** - your last name
      - **Email:**  Preferably use a personal email address to which you have access (Gmail, Yahoo, etc) and different from any email already in use in the tenancy.
      - **Use the email address as the username:** Leave checked unless you want to use a username that is not the email. It can be used if you want to use the same email already in use in the tenancy.
      - **Assign cloud account administrator role:** Leave unchecked.
      - Check the box besides **oci-group**

    Click **Create**.

      ![New user form](images/id-domains-create-user.png)

   After creating the user, you will be directed to the user details.

   The newly created user will receive an email with an activation link like this:

      ![Activation email](images/id-domains-activation-message.png)

1. If the user did not receive the email, in the user details, you have a reset password button that will send a password reset link.

      ![Reset password](images/id-domains-user-resetpw.png)

   After clicking on the reset button, your will be prompted for confirmation before the reset link is sent.

1. Check the messages in the email account you used for the new user. Open the activation link (password reset will take you to a similar screen)

   ![Reset password](images/id-domains-resetpw.png)

1. Now, let’s create a security policy that gives your group permissions in your assigned compartment. For example, create a policy that gives permission to members of group **oci-group** in compartment **Workshop**:

   a) Click the **Navigation Menu** in the upper left. Navigate to **Identity & Security** and select **Policies**.

   ![IAM Policy](images/iam-policies.png)

   b) On the left side, select the **Workshop** compartment. After you have selected the **Workshop** compartment, click **Create Policy**.

   ![Click on Create Policy](images/id-domain-create-policy.png)

      >**Note:** You may need to click on the + sign next to your main compartment name to be able to see the sub-compartment ***Workshop***. If you do, and you still don't see the sub-compartment, ***refresh your browser***. Sometimes your browser caches the compartment information and does not update its internal cache.
    
   
   c) Enter a unique **Name** for your policy (for example, "Policy-for-oci-group").
      >**Note:** the name can NOT contain spaces.

   d) Enter a **Description** (for example, "Policy for OCI Group").

   e) Select **Workshop** for the compartment.

   f) Click **Show manual editor** and enter the following **Statement**:

     ```
     <copy> Allow group default/oci-group to manage all-resources in compartment Workshop</copy>

     ```

     >**Note:** If you do not include the *identity_domain_name* before the *group_name*, then the policy statement is evaluated as though the group belongs to the default identity domain.

   g) Click **Create**.

   ![Click on Create](images/create-policy.png)

8. Verify user permissions.

   a) Click the **Navigation Menu** in the upper left. Click **Compute** and then click **Instances**.

   ![Navigate to Compute Instances](images/compute-instances.png " ")

   b) Try to select any compartment from the left menu.

   c) The message “**You don’t have permission to view these resources**” appears. This is normal as you did not add the user to the group where you associated the policy.
      ![Error message can be ignored](images/no-permission.png)

   d) Sign out of the Console.

9. Add User to a Group.

      a) Sign back in with the ***admin*** account.

      b) Click the **Navigation Menu** in the upper left. Navigate to **Identity & Security** and select **Domains**. From the **Users** list, click the user account that you just created (for example, `Test User`)  to go to the User Details page.
         ![Navigate to Identity Domains](images/id-domains.png)

      C) Select the default domain.
         ![Select the Default](images/id-domains-default.png " ")

      d) Under the **Resources** menu on the left, click **Users**, if it's not already selected.
         ![Select Users](images/id-domains-users.png)

      e) Click **Assign User to groups**.
         ![Click on Assign User to groups](images/id-domains-users-assign-group.png)

      f) From the **Groups** list, click the group that you just created (for example, `oci-group`)  to go to the User Details page.

      g) Click **Add**.
         ![Press the Assign button](images/id-add-user-to-group.png)

      h) Sign out of the Oracle Cloud website.

10. Verify user permissions when a user belongs to a specific group.

      a) Sign in with the local **Test User** account you created. Remember to use the latest password you assigned to this user.

      b) Click the **Navigation Menu**. Click **Compute** and then click **Instances**.

      c) Select compartment **Workshop** from the list of compartments on the left.

      ![Select ***Workshop***](images/select-demo.png)

      d) There is no message related to permissions and you are allowed to create new instances.  

      e) Sign out.

</if>

Please *proceed to the next lab*.


## Acknowledgements

- **Author** - Cristian Manea
- **Last Updated By/Date** - Cristian Manea, March 2023
