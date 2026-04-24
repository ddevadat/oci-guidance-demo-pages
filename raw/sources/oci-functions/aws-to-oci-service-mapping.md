---
title: "Oracle Cloud Infrastructure for Amazon Web Services professionals"
description: "To make more informed decisions regarding which cloud services to adopt, solution architects and CloudOps administrators considering popular cloud offerings need to compare our competitors' services with Oracle Cloud Infrastructure's similar services."
metadata:
  type: guide
  tags:
    - oci-functions
    - aws-service-mapping
  updated_at: 2026-04-24
---
To make more informed decisions regarding which cloud services to adopt, solution architects and CloudOps administrators considering popular cloud offerings need to compare our competitors' services with Oracle Cloud Infrastructure's similar services.

This guide introduces Amazon Web Services (AWS) professionals to the core capabilities of Oracle Cloud Infrastructure. It is designed for AWS Solution Architects and SysOps Administrators familiar with AWS features and setup and want to gain experience configuring OCI products immediately. Like AWS, Oracle Cloud Infrastructure is built around a core set of compute, storage, database, and networking services and over the top offer a broad and deep set of capabilities with global coverage.

This article provides comparisons of these general concepts:
- Regions & Availability Domains
- Domains Accounts, Tagging & Organizing
- Service Mapping

### Regions and Availability Domains

Amazon Web Services and OCI products are both deployed in similar variations of regions and availability domains.

Nearly all AWS products are deployed within regions located around the world. Each region comprises a group of data centers that are in relatively close proximity to each other. Amazon divides each region into two or more availability zones. By design, each AWS availability zone is isolated and independent from other AWS zones. This design helps ensure that the availability of one zone doesn't affect the availability of other zones, and that services within zones remain independent of each other.

Similarly, OCI is hosted in regions and availability domains. A region is a localized geographic area, and an availability domain is one or more data centers located within a region. A region is composed of one or more availability domains. OCI availability domains are isolated from each other, fault tolerant, and very unlikely to fail simultaneously or be impacted by the failure of another availability domain. When you configure your cloud services, use multiple availability domains to ensure high availability and to protect against resource failure.

For a full mapping of OCI 's global regions and availability domains, see OCI's [Cloud Regions—Infrastructure and Platform Services](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-comp-data-regions-na).

Each availability domain contains three fault domains. A fault domain is a grouping of hardware and infrastructure within an availability domain. This lets you distribute your instances so that they are not on the same physical hardware within a single availability domain. A hardware failure or Compute hardware maintenance event that affects one fault domain does not affect instances in other fault domains.

The physical hardware in a fault domain has independent and redundant power supplies, which prevents a failure in the power supply hardware within one fault domain from affecting other fault domains.

AWS's location terms and concepts map to those of OCI as follows:

| Concept | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Cluster of data centers and services | Region | [Region](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-region) |
| Abstracted data center | Availability Zone | [Availability Domain](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-region) |
| Hardware Grouping | Placement groups | [Fault domains](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-fd) |

### Accounts, Tagging, and Organizing

Here, we compare what happens when you sign up for an AWS account and for an OCI account and how these services organize those accounts.

To use an AWS service, you must sign up for an AWS account. After you have completed this process, you can launch any service under your account within Amazon's stated limits, and these services are billed to your specific account. Now to manage these AWS resources, you can optionally assign your own metadata to each resource in the form of tags.

A tag is a label that you assign to an AWS resource. Each tag consists of a key and an optional value. Tags enable you to categorize your resources in different ways, for example, by purpose, owner, or environment. You can further group together and organize resources with AWS Resource Groups. Moreover, you can create several different accounts and centrally manage them using AWS Organizations.

Similarly, OCI requires you to sign up for the service. When your request is processed, you will be provisioned a tenancy in OCI. By default, any OCI tenancy has a default root compartment, named after the tenancy itself. The tenancy administrator (default root compartment administrator) is any user who is a member of the default Administrators group.

Compartments help to organize and isolate cloud resources in a way that they can be accessed only by certain groups that have been given permission by an administrator in your organization. Once compartments are created, they can be assigned their own administrators who can then create sub-compartments and assign delegated administrators to each of them. OCI supports up to a 6-level deep compartment hierarchy and the administrator of a parent compartment has full powers over its children compartments. Compartments are global, they stretch out to all OCI regions within a given tenancy.

OCI Tagging enables you to attach arbitrary, free-form metadata to cloud resources, like compute instances. The labels that tags provide can help you organize and control resources. For example, you can add tags to describe the business organizations that are responsible for a resource, or operational metadata needed to manage your resources effectively. While other public cloud tagging implementations support free-form tags, that approach provides no structure. OCI supports free-from tags, but our solution goes further. We recommend the use of our Defined Tags, which eliminate many of the drawbacks of free-form approaches. Defined Tags support a schema to help you control tagging, ensure consistency, and prevent tag spam. You can even use tags to script bulk actions on your resources, to automate and simplify tasks.

AWS and OCI both have default soft limits on their services for new accounts. The service limit is the allowance set on a resource. For example, your tenancy is allowed a maximum number of compute instances per availability domain. These soft limits are not tied to technical limitations for a given service—instead, they are in place to help prevent fraudulent accounts from using excessive resources, and to limit risk for new users, keeping them from spending more than intended as they explore the platform. If you find that your application has outgrown these limits, you can also [request a service limit increase](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-comp-req-serv-limit). Sometimes these limits may be increased for you automatically based on your OCI resource usage and account standing.

| Concept | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Account | Account | [Tenancy](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-tenancy) |
| Organizing resources | Resource Groups | [Compartments](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-compartments) |
| Metadata to resources | Tags | [OCI Tagging](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-tagging) (Free-form & Defined Tags) |
| Multiple accounts management | AWS Organizations | [Organization Management](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=org-man-ovu) |

### Service Mapping

The following tables provide a side-by-side comparison of the various services available on AWS and OCI.

#### Compute Service Mapping

This table maps AWS compute services to comparable OCI compute services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Multi-tenant Virtual Machines | Amazon Elastic Compute Cloud (EC2) | [OCI Virtual Machine Instances](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-vm-instances) |
| Single tenant Virtual Machines | Amazon EC2 Dedicated Instances | [OCI Dedicated Virtual Machine Hosts](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-ded-vm-hosts) |
| Bare Metal hosts | Amazon EC2 Bare Metal Instances | [OCI Bare Metal Instances](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-bmi) |
| Managed Kubernetes Service and Registry | Amazon Elastic Kubernetes Service (EKS)  Amazon Elastic Container Registry | [Oracle Container Engine for Kubernetes](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oce-kub%20)  [OCI Registry](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-registry) |
| Serverless | Lambda | [Oracle Functions](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-functions%20) |

#### Storage Service Mapping

This table maps AWS storage services to comparable OCI storage services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Object Storage | Amazon Simple Storage Service (S3) | [Object Storage](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-object-storage) |
| Archival Storage | Amazon S3 Glacier | [Archive Storage](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-arch-stor) |
| Block Storage | Amazon Elastic Block Store (EBS) | [Block Volumes](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-block-vol) |
| Shared File System | Amazon Elastic File System | [File Storage](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-file-stor) |
| Bulk Data Transfer | AWS Snowball | [Data Transfer Appliance](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-data-xfer) |
| Hybrid Data Migration | AWS Storage Gateway | [rclone](https://blogs.oracle.com/linux/post/using-rclone-to-copy-data-in-and-out-of-oracle-cloud-object-storage)  [OCIFS Utility](https://docs.oracle.com/en-us/iaas/oracle-linux/ocifs/index.htm) (Linux) |

#### Networking and Edge Service Mapping

This table maps AWS networking and edge services to comparable OCI networking and edge services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Virtual Network | Amazon Virtual Private Cloud | [Virtual Cloud Network (VCN)](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-vcn) |
| Dedicated Private Connectivity | AWS Direct Connect | [FastConnect](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-fc) |
| Site-to-Site Connectivity | AWS VPN | [VPN Connect](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-vpn-conx) |
| DNS and Query Management | Amazon Route 53 | [OCI Domain Name System (DNS)](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-dns)  [OCI Traffic Management](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-tm) |
| Traffic Load Balancer | AWS Elastic Load Balancing | [OCI Load Balancing](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-lb) |
| Managed Email Delivery Service | Amazon Simple Email Service | [OCI Email Delivery](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-email) |
| Firewall | Web Application Firewall | [Web Application Firewall](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-app-firewall) |
| DDoS Protection | Shield | [DDoS Protection](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-ddos) |

#### Database Service Mapping

This table maps AWS database and cache services to comparable OCI services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Managed Relational Database systems | Amazon Relational Database Service (RDS)  Amazon Aurora | [Oracle Autonomous Transaction Processing (ATP)](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-atp)  [Oracle MySQL Database Service](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-mysql-product)  [OCI PostgreSQL](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=post-gres-home%20) |
| Cache Services | AWS ElastiCache | [Oracle Cloud Infrastructure Cache](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-cache-home)  [OCI Search with OpenSearch](https://docs.oracle.com/en-us/iaas/Content/search-opensearch/home.htm) |
| NoSQL | Amazon DynamoDB | [Oracle NoSQL Database Cloud Service](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-nosql-dbs)  [Oracle Autonomous JSON Database (AJD)](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-ajd) |
| Data warehousing | Amazon Redshift | [Oracle Autonomous Data Warehouse (ADW)](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=autonomous-data-warehouse)  [Oracle MySQL HeatWave](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-heatwave-product) |

#### Big Data and Analytics Service Mapping

This table maps AWS big data and analytics services to comparable OCI services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Batch Data Processing | Batch  Amazon Elastic MapReduce | [OCI Data Flow](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-data-flow)  [Oracle Big Data Service](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=big-data-service) |
| Streaming Data Ingest | Amazon Kinesis | [OCI Streaming](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=cloud-native-streaming) |
| Data Analytics and Visualization | Amazon Quicksight | [Oracle Analytics Cloud](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-analytics-cloud) |
| Metadata Management | Glue | [OCI Data Catalog](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-data-catalog) |
| Query | Athena | Query Service (limited availability) |

#### Messaging and Notifications Service Mapping

This table maps AWS messaging and notifications services to comparable OCI services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Tracking changes to resources | Amazon Cloudwatch Events | [OCI Events](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-events) |
| Messaging Queue | Amazon Simple Queuing Service | [OCI Queue](https://docs.oracle.com/en-us/iaas/Content/queue/home.htm) |
| Publish/Subscribe | Amazon Simple Notification Service | [OCI Notifications](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-notify) |

#### Management Services Service Mapping

This table maps AWS management services to comparable OCI services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Monitoring | Amazon CloudWatch | [OCI Monitoring](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-monitor) |
| Logging | Amazon CloudWatch Logs | [OCI Logging](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-log) |
| Deployment | CloudFormation | [OCI Resource Manager](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-resource-manager-iac) |

#### Security and Identity Service Mapping

This table maps AWS security and identity services to comparable OCI services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Identity and Access Management | IAM | [OCI IAM](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-iam%20) |
| Key Management | Key Management Service | [OCI Vault](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-vault) |
| Audit | CloudTrail | [OCI Audit](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-audit-overview) |
| Security Monitoring | Inspector | [OCI Cloud Guard](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=cloud-guard) |

#### Machine Learning/AI/GenAI Service Mapping

This table maps AWS services for Machine Learning, AI, and GenAI services to comparable OCI services.

| Services | Amazon Web Services | Oracle Cloud Infrastructure |
| --- | --- | --- |
| Model Building, Training, Tuning | Amazon SageMaker  Amazon Glue | [OCI Data Science](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-data-science)  [OCI Data Labeling](https://docs.oracle.com/pls/topic/lookup?ctx=en/solutions/oci-for-aws-professionals&id=oci-data-labeling) |