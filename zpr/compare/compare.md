# Save time and make settings clear

## Introduction

Just to highlight the benefit of using ZPR over the standard networking configurations:
We can see that we only need to define policies once to give instances access to database or application servers and then we only need to ZPR tag those instances to become part of that policy use. We no longer need to deal with sets of IP addresses for those cloud instances. For enabling your employees to jump box onto specific cloud instances we can give specific VNC access to those instances and never deal with IP addresses there as well. If you really do need to limit access to just a few people you still have the option to limit to a few very specific laptop IP addresses.

Estimated Lab Time: 2 minutes

### In this workshop, we did the following

* Created a new ZPR namespace
* Created ZPR attributes
* Protected a compute instance with ZPR tagging
* Protected a autonomous database with ZPR tagging
* Tagged our VNCs, compute instances and autonomous database with ZPR tagging
* Created policy to allow limit access to the instance and database resources
* Saw how we had to use the private endpoint of the database with ZPR and how that blocks access from outside of the OCI network


## Task 1: Create a ZPR Namespace

 ![Namespaces](images/zpr_attribute_namespace.png)
 ![Namespace creation](images/zpr_create_namespace.png =40%x*)

## Task 2: Create ZPR Attributes

 ![ZPR Attributes](images/sec_attrs.png =50%x*)

## Task 3: Tagging Resources to protect them

 ![Assign an attribute to a resource](images/zpr_protected.png =55%x*)
 ![Select resource to protect](images/protect_vm.png =55%x*)


## Task 4: Creating Policies

 ![Assign an attribute to a resource](images/db_connection_policy.png =55%x*)
 ![Select resource to protect](images/policy_for_pe.png =55%x*)


## Task 5: Private Endpoint use

 ![Assign an attribute to a resource](images/db_pe_fqdn.png =55%x*) 


## Learn More

*(optional - include links to docs, white papers, blogs, etc)*

* [OCI Zero Trust Packet Routing](https://www.oracle.com/security/cloud-security/zero-trust-packet-routing/)
* [ZPR Help documents](https://docs.oracle.com/en-us/iaas/Content/zero-trust-packet-routing/overview.htm)

## Acknowledgements

* **Author** - Jim Smith, Principle Product Manager OCI
* **Contributors** - Dmitry Erastov, Consulting Member of Technical Staff OCI
* **Last Updated By/Date** - Jim Smith, January 2025