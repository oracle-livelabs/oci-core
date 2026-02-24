# Introduction

In this workshop, you will create two virtual cloud networks (VCNs), then deploy one compute instance to each VCN. Next, you will configure a local peering connection between the two VCNs, then test connectivity between the two compute instances.

Estimated time: 60 minutes

## About Virtual Cloud Network Peering

VCN peering is the process of connecting multiple virtual cloud networks (VCNs). There are two types of VCN peering:

- Local VCN peering (within region)
- Remote VCN peering (across regions)

You can use VCN peering to divide your network into multiple VCNs (for example, based on departments or lines of business), with each VCN having direct, private access to the others. There's no need for traffic to flow over the Internet or through your on-premises network by way of an IPSec VPN or FastConnect. You can also place shared resources into a single VCN that all the other VCNs can access privately.

Because remote VCN peering crosses regions, you can use it to create private connections between different regions. Traffic that travels between regions using a Remote Peering connection utilized the Oracle private backbone instead of the public internet. Common use cases for remote peering include multi-region application architecture or disaster recovery solution design.

Watch the video below for a quick introduction to Virtual Cloud Network Peering.
[](youtube:u3BssXJzRJk)

### Objectives
- Create two virtual cloud networks
- Create two compute instances, one in each VCN
- Configure Route Tables and Security Lists to establish local peering connection between two VCNs
- Verify the peering connection

### Prerequisites
* An Oracle Cloud Account - please view this workshop's LiveLabs landing page to see which environments are supported

>**Note:** If you have a **Free Trial** account, when your Free Trial expires, your account will be converted to an **Always Free** account. You will not be able to conduct Free Tier workshops unless the Always Free environment is available. 

**[Click here for the Free Tier FAQ page.](https://www.oracle.com/cloud/free/faq.html)**

You may now **proceed to the next lab**.

## Additional Recommended Resources

1. [Oracle Cloud Infrastructure (OCI) Training](https://cloud.oracle.com/en_US/iaas/training)
2. [Familiarity with OCI console](https://docs.us-phoenix-1.oraclecloud.com/Content/GSG/Concepts/console.htm)
3. [Overview of Networking](https://docs.us-phoenix-1.oraclecloud.com/Content/Network/Concepts/overview.htm)
4. [Familiarity with Compartments](https://docs.us-phoenix-1.oraclecloud.com/Content/GSG/Concepts/concepts.htm)

## Acknowledgements

- **Author** - Kay Malcolm, Director, Product Management
- **Adapted by** -  Yaisah Granillo, Cloud Solution Engineer, NA Cloud
- **Contributors** - LiveLabs QA Team (Arabella Yao, Product Manager | Isa Kessinger, QA Intern)
- **Last Updated By/Date** - Eli Schilling, Cloud Architect, Lab Experience Team, February 2026