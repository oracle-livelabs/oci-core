# Summary

* The first thing you need in your Tenancy is a Compartment.
* You will need at least one Virtual Cloud Network with a public subnet to connect any instance to.
* When creating Compute instances, you can choose various Images, allowing you to select the correct operating system you desire
* When creating Compute instances, you can configure the Instance size based on the shapes available
* When creating a Linux based instance, you will need a SSH Key set
    * The public SSH key is used inside the created instance to validate access
    * The private SSH key is used by the SSH Client to connect to the instance
    * The private key should be stored securely! This is what gives you access to your instance. 
* By default, only port 22 / SSH is allows to any instance. To configure any other access, you have 2 choices:
    * Network Security list: This sets the permissions for all instances inside a single subnet
    * Network Security group: This set the permissions for individual instances assigned to the group
* The Public IP Address and default username for login are displayed on the Instance’s main overview page
