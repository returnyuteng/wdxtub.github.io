title: 云计算 阅读材料 5 Cloud Management
categories:
- Technique
toc: true
date: 2016-01-20 11:37:56
tags:
- CMU
- 云计算
- 讲义
---

You have learned the origins of a data center, from its roots in mainframe computers to the newest trends, and about what goes inside a data center (cooling, power, servers, network, and more), as well as some of the design criteria for various components that data center.

<!-- more -->

---

When designing large data centers, it is not possible to follow the same practices as that of a small data center. To truly leverage the economies of scale, it is important that the data center have a software layer that allows resources to be governed and managed easily. The cloud software stack is a platform to run a cloud given a pool of physical resources. Since most Cloud Service Providers (CSPs) are extremely wary of revealing their techniques (since it is their Intellectual Property) we have to rely on reverse engineering, rumors and the contribution of open-source cloud software stacks like OpenStack to understand the components involved.

In this module, we will start by learning about how a software middleware layer enables all the benefits of the cloud. We will look at the simplest use case, that of resource provisioning, and understand that a long series of steps are involved in handling a simple resource request. Of course, cloud providers charge for every quantifiable resource that a user utilizes. We must understand the details of the billing and monitoring systems that enables CSPs to profit from their data center. Automation and orchestration are some important techniques that we will look at the enable the low staff-to-resource ratio at CSPs, and drive down their effective costs.

Finally, we will look at OpenStack, an increasingly popular software platform that allows anyone with physical resources to create a cloud environment.

## Cloud Middleware

Middleware is a general term for software that serves to "glue together" separate, often complex and already existing, programs. The term middleware is used in many contexts. For example, in the context of a single computer, middleware exists between the operating system kernel and application programs in the form of APIs, which manage access to system resources such as hardware devices.

![Figure 2.22: Cloud Middleware Features](/images/14545511086121.jpg)

Cloud Middleware is a software platform that controls and coordinates different cloud services and makes it possible for users to issue service requests, and cloud providers to manage their infrastructure. Cloud Middleware consists of multiple abstraction layers that hides system complexity and enables communication between various applications, services, and devices that are part of a cloud service.

**Cloud Middleware Features**

There are a number of distinct and important features that cloud middleware provides, which come with several benefits. Some of the most important responsibilities of a cloud middleware stack are as follows:

Interoperability: Middleware is designed to connect distinct application services with different APIs to one another. Cloud service APIs act as middleware for cloud services by taking instructions from a program (written in languages such as Java, Python etc.), and translating them into service calls that the cloud service can understand. These instructions are further passed down the middleware stack at the cloud service provider’s end to perform actions (such as create virtual machines, allocate disk space, create a database table, etc.). Thus cloud middleware is the proverbial “glue” that enables multiple distinct applications and services to connect and communicate to each other.

Virtualization Management: Cloud middleware is also responsible for the configuration, allocation, creation, management and destruction of virtualized resources from physical resources. As an example, when a cloud service provider gets a request from a client to provision a virtual machine, it handles that request through multiple middleware layers until it reaches a hypervisor layer, which handles the configuration and allocation of a virtual machine for the client.

Resource Allocation and Scheduling: As discussed above, an important aspect of cloud middleware is the management of resources. As part of this responsibility middleware must manage resource allocation and scheduling of multiple resource types in order to achieve multiple goals such as performance, isolation, utilization, etc.

Load Balancing and Fault Tolerance: Cloud service providers must utilize adequate load balancing mechanisms in their middleware in order to optimize the distribution of load on multiple back-end services and physical infrastructure. The middleware should also coordinate with back-end resources in order to provide end-to-end fault-tolerance so that the availability of services to the client meets required SLOs.

Resource Monitoring: A crucial responsibility of middleware is the monitoring of resources. Monitoring provides a source of data which is valuable for the internal middleware features such as allocation, scheduling, load balancing and fault tolerance as discussed above. In addition, data from monitoring systems can be made available to clients, which gives them an additional visibility into the state of their applications and provisioned resources.

User Management and Security: Cloud middleware also must provide support for access control of users, and use standard security practices for the management of various types of credentials to control access to individual resources. The user management system in the middleware should provide features that allow cloud clients to create and destroy entities such as users and groups, and configure the Access Control Lists (ACLs) that define the resources that individual users and groups have access to.

User Interface and APIs: Finally, cloud middleware must make available a client-facing set of APIs. It is also typical of cloud middleware to provide user-friendly interfaces (typically in the form of Web interfaces), where clients can log in and manage their provisioned resources and make service requests.

![](/images/14545512278768.jpg)

**References**

1. Amrani C., Filali K., Ahmed K., Diallo A., Telolahy S. (2012). "A Comparative Study of Cloud Computing Middleware." 12th IEEE/ACM International Symposium on Cluster, Cloud and Grid Computing.

## Resource Provisioning

The most basic role of a Cloud Service Provider is to package and isolate virtual parts of a physical data center and provide access for cloud users to provision resources on the cloud. Provisioning is the process of mapping abstract resource requests by cloud users to physical resources on servers within a data center.

In IAAS, this mainly involves launching virtual machines on top of hypervisors (these are special software applications that enable isolation of virtual machines on top of a physical machine- we will read more about them soon), as well as mounting storage volumes from a storage pool and creating a private network overlay for the user’s resources.

The techniques behind virtualizing compute, storage or networking resources is dealt with in future modules. Here, we focus on understanding some of the high level steps that a CSP must take to create and allocate a publicly accessible virtual machine with a fixed set of resources to an end user.

**Components of a Resource Provisioning System**

A resource provisioning system on the cloud generally has the following sub-parts:

1. Access to a physical pool of resources- generally thousands or millions of servers, interconnected by a network (generally using a fat-tree topology as discussed in a previous module) and also a large pool of disks.
2. An identity management subsystem that maintains and validates the end user’s credentials for accessing many different types of resources; it also provides for role based access control.
3. A metering and monitoring system to detect utilization of physical resources
4. A billing and charge management system to map the metered resources to physical costs and take appropriate actions based on the user’s allowed privileges.
5. A resource manager that works with a hypervisor to map physical resources to virtual abstractions.
6. Often, the provisioning system will have a web front-end or an API.

**Cloud End-to-End Service Provisioning Flow**

Figure 2.23 below shows the typical end-to-end steps for a customer provisioning a virtual machine from a Cloud Service Provider (CSP):

![Figure 2.23: Typical End-to-End IaaS provisioning steps](/images/14545514768762.jpg)

The steps illustrated in Figure 2.23 are explained as follows:

1. The customer logs on to the portal and is authenticated by the identity management system.
2. Based on the customer’s entitlement, the portal extracts a subset of services that the user can order from the service catalogue and constructs a ‘request catalog’.
3. The customer selects a service, e.g. a virtual server of a particular size. Associated to this service is a set of technical requirements such as the amount of vRAM, vCPU, etc. in addition to business requirements such as high availability or SLA requirements.
4. The orchestration tool extracts the technical service information from the service catalog and decomposes the service into individual parts, such as compute resource configuration, network configuration, and so on.
5. The provisioning process is initiated.
6. The virtual machine running on the server is provisioned using the server/compute domain manager.
7. The network, including firewalls and load balancers, as well the storage is provisioned by the network, network services and storage domain managers.
8. Charging is initiated for billing/chargeback and the change management case is closed and the customer is notified accordingly.

![](/images/14545515552417.jpg)

**Metering and Monitoring Cloud Services**

Cloud service providers (CSPs) charge their customers according to usage of their services. In order to do that, providers have to monitor the usage of their services carefully and have to include the cost of running these services in their infrastructure. This is mission critical for both CSPs and their customers.

The revenue stream of a CSP depends on the correctness of metering and monitoring their resources. Considering AWS revenues in 2014, losing metrics for a couple of hours can cost Amazon millions of dollars. On the other, overcharging customers for a couple of hours can highly affect their credibility.

From the customer’s point of view, the cost of cloud resources forms an important part of their expenses. In order for cloud clients to make budget plans, they must receive consistent bills every month. This poses important challenges to cloud providers for metering and monitoring.

**Challenges in Monitoring and Metering**

There are various costs included in cloud resources. Although fixed costs such as facilities, staff, servers are easy to calculate, variable costs require constant metering and monitoring. The advantage of using CSPs comes from paying only for the resources which are used. For example, provisioning an EC2 instance includes the cost of instance usage per hour, storage per GB-month for each storage type, and data transfer per GB-month. Even for this one resource, AWS has to keep track of these metrics for every instance and attached volume. In Figure 2.24, a possible break down of various cost for services can be found. If we imagine doing this for more than 1 million AWS customers for tens of different types of services, this will require the metering and monitoring of gigabytes of logs every minute and charging customers accordingly. The most popular model which is used to define such metrics is called the chargeback model.

![Figure 2.24: Metering in Different Types of Cloud Services](/images/14545516100780.jpg)

**Chargeback Model**

Basically the chargeback model is the ability of an IT organization to measure the usage of resources and chargeback their customers accordingly. Traditionally the chargeback model is easy to implement since an IT department can easily divide its budget for the business units that it serves like software licences, stand-alone servers etc. This is challenging in cloud because the CSP needs to consider the rate and time of consumption.

**Validation of Metering and Monitoring**

From the customer’s point of view, verifiable metering is an important issue. There are some costs which are easy to measure like EC2 usage (hourly usage * cost per hour) but it is hard for customers to measure other resources such as data transitions or I/O requests. In order to verify metering and monitoring, users can work with certified cloud providers. For instance, IBM’s “Resilient Cloud Validation” program allows businesses who collaborate with IBM to perform a consistent program of benchmarking and validation of cloud services.

**Case Study: Ceilometer**

Although the underlying architecture of metering and measuring is hidden by corporate CSPs, Ceilometer is designed for OpenStack metering, billing and rating. The high-level architecture of OpenStack Ceilometer can be summarized as follows (Figure 2.25):

![Figure 2.25: Ceilometer Architecture](/images/14545516558630.jpg)

Polling Agent: A daemon which polls OpenStack services for metering.

Notification Agent: A daemon which listens to notifications on the message queue and converts them into samples and events.

Collector: Daemon designed to gather metering data created by the notification and polling agents.

API: Service to query the data recorded by the collector.

Alarming: Daemons to evaluate and trigger notifications based on predefined rules.

Each of these services are designed to scale horizontally. Additional workers and nodes can be added according to expected load.

![](/images/14545516976128.jpg)

## Cloud Orchestration and Automation

Cloud Orchestration is the process by which all of the provisioning, middleware and other complex systems in a cloud can be automatically arranged and coordinated. Today, cloud orchestration supports the specification and management of the following resources including but not limited to:

+ Compute Servers (In the form of virtual machines)
+ Auto Scaling
+ Load Balancers
+ Databases
+ Block Storage
+ DNS and Virtual Network (VLANs)
+ Software configuration and setup (typically in the form of bootstrapping scripts)

**Benefits of Orchestration**

Cloud orchestration is a method to fully realize the dynamic potential of cloud infrastructure, by allowing users to specify and configure a complete application encompassing multiple resource types. One of the most important aspects of the cloud is the rapid service delivery which is made possible by cloud orchestration. It also saves cost by eliminating manual intervention and management of IT services. Benefits of cloud orchestration can be summarized as follows:

+ Integration, automation and optimization of service deployment across heterogeneous environments.
+ Self-service portals for selection of cloud services, including storage and networking, from a predefined menu of offerings.
+ Reduced need for intervention to allow lower ratio of administrators to physical and virtual servers.
+ Automated high-scale provisioning and de-provisioning of resources with policy-based tools to manage virtual machine sprawl by reclaiming resources automatically.
+ Ability to integrate workflows and approval chains across technology silos to improve collaboration and reduce delays.
+ Real-time monitoring of physical and virtual cloud resources, as well as usage and accounting chargeback capabilities to track and optimize system usage.
+ Making adoption of best practices easy by prepackaging automation templates and workflows for most common resource types.

**Orchestrator Tools**

There are variety of tools which provides orchestration, Puppet and Chef are popular examples.

**Puppet**

Puppet is a tool that can be used to issue service commands to multiple client machines from a master machine. This allows developers and systems administrators to manage client machines from a single master machine, providing commands to individual clients based on code which describes the configuration actions that are to be performed on each machine:

![Figure 2.26: Puppet](/images/14545518040764.jpg)

**Chef**

Chef uses the same concepts as Puppet, but differs in deployment. Chef operates using user-specified recipes, which describe the state of the resources in the system, such as the packages (and versions) to install, start up daemons or services to execute, or an data to be downloaded/created. This ensures an identical operating environment with the same resources and configurations across all systems. Using Chef, it is possible to automate the creation of a complex distributed system, stitching together various components and workflows.

![](/images/14545518601244.jpg)

## Case Study : OpenStack

Earlier, we described OpenStack as a popular solution that allows developers to build both public and private clouds. At the time of writing, OpenStack has mature offerings that cover a large part of the cloud software stack spectrum. We take a very quick conceptual overview of the various pieces of the OpenStack architecture, as the complete description of the OpenStack architecture is beyond the scope of this course. The purpose is to give you an overview of the different components that make up a cloud software stack.

OpenStack consists of multiple layers that can be used to configure, provision, manage, monitor and deprovision various types compute, storage and networking resources.

A high level view of the various services involved in the OpenStack middleware suite is represented in Figure 2.27 below:

![Figure 2.27: OpenStack Service Architecture](/images/14545518887840.jpg)

**User Authentication Service (Keystone)**

The primary authentication service in OpenStack is called Keystone. Keystone is an OpenStack project that provides Identity, Token, Catalog and Policy services for use specifically by individual services in the OpenStack family.

The identity service provides authentication credential validation as well as data about users and groups. The token service validates and manages session tokens used for authenticating requests once a user’s credentials have already been verified. The catalog service provides an endpoint registry used for endpoint discovery. The policy service provides a rule-based authorization engine and the associated rule management interface which can be used to dynamically grant or deny resource privileges to specific applications and services.

**Monitoring Service (Ceilometer)**

The Ceilometer project aims to deliver a unique point of contact for billing systems to acquire all of the measurements needed to establish customer billing, across all current OpenStack core components with work underway to support future OpenStack components. Celiometer is designed to provide efficient collection of metering data, while ensuring metering messages are digitally signed and non-repudiable. Ceilometer was introduced in “Metering and Monitoring Cloud Services”.

**Orchestration Service (Heat)**

Heat is the primary orchestration system in OpenStack. Heat provides orchestration services for higher-level systems such as Sahara, OpenStack’s cluster provisioning system that is similar to AWS EMR. Heat can orchestrate individual cloud middleware services such as the compute service (nova), block storage service (Cinder) and networking services (Neutron).

**Virtual Machine Image Service (Glance)**

Glance image services include discovering, registering, and retrieving virtual machine images. Glance has a RESTful API that allows querying of VM image metadata as well as retrieval of actual virtual machine images.

**Object Storage Service (Swift)**

Swift is a highly available, distributed, eventually consistent object store, similar to Amazon’s S3. You create, modify, and get objects and metadata by using Swift’s Object Storage API, which is implemented as a set of REST web services. We will take a closer look at OpenStack’s Swift service in future modules.

**Block Storage Volume Management Service (Cinder)**

Cinder is an OpenStack project to provide “block storage as a service”, similar to Amazon’s Elastic Block Storage (EBS) service. Cinder allows users to define block storage devices and attach them as volumes to individual virtual machines. Cinder virtualizes pools of block storage devices and provides end users with a self service API to request and consume those resources without requiring any knowledge of where their storage is actually deployed or on what type of device. Cinder is also used by the Glance service to store virtual machine images as volumes. Cinder is designed to work with a growing number of storage systems and devices including Storage-Area-Network (SAN) appliances and distributed file systems.

**Cluster Provisioning and Management Service (Sahara)**

The Sahara project aims to provide users with a simple means to provision data processing frameworks (such as Hadoop, Spark and Storm) on OpenStack, much like the Elastic MapReduce (EMR) service on AWS. Sahara can be used to specify cluster configuration parameters such as the framework version, cluster topology, node hardware details and more. Sahara uses Nova to provision individual cluster nodes using framework specific images supplied by Glance. Sahara then runs special scripts to complete the configuration of each of the individual cluster nodes so that they are ready to execute jobs.

**Compute Service (Nova)**

Nova is an OpenStack project designed to provide massively scalable, on demand, self service access to compute resources. Nova is designed to provision, manage different virtual machines deployed using different virtualization platforms including Xen, VMWare, Hyper-V and can even deploy virtual machines onto EC2, with an aim to support the notion of “cloud bursting”. The Nova service is also used by other services such as Trove (to provision virtual machines that hold databases), and Sahara (to provision virtual machines for analytics clusters).

**Database-as-a-Service (Trove)**

Trove is a Database as a Service for OpenStack. It’s designed to run entirely on OpenStack, with the goal of allowing users to quickly and easily utilize the features of a relational database without the burden of handling complex administrative tasks. Cloud users and database administrators can provision and manage multiple database instances as needed. The service focuses on providing resource isolation at high performance while automating complex administrative tasks including deployment, configuration, patching, backups, restores, and monitoring.

**User Interface (Horizon)**

Horizon is the canonical implementation of OpenStack’s Dashboard, which provides a web-based user interface to OpenStack services including Nova, Swift, Keystone, etc. Users can log into Horizon to get interactive dashboard views of the status of their resources, and issue service requests to individual OpenStack services.

**Physical Hardware Management (Ironic)**

Ironic is a bare-metal provisioning system that can be used to control physical serves. Ironic has the ability to power on and power off individual servers using industry-standard protocols such as Intelligent Platform Management Interface (IPMI). Once machines are powered on, it can coordinate the machine boot up process using the Preboot eXecution Environment (PXE) boot, using specific boot images stored in the image management service (Cinder).

One may wonder why bare metal provisioning makes sense in a cloud environment where virtualized resources are the norm. There are situations where bare metal provisioning could be advantageous: A client that has a higher SLO requirement for compute performance for say, a high performance computing application, may require dedicated hardware without the overheads of a hypervisor. In such a case, it would be advantageous to provision and dedicate an entire physical server without virtualization.

As an example of one of the scenarios that Ironic can be used in, consider the case where there happens to be a physical server capacity crunch due to a large number of virtual machines that have been provisioned. In this case, Ironic can power on additional servers that are powered off and configure them to boot with a specific hypervisor image. Once the additional capacity is made available, virtual machines can be provisioned on the new servers, or existing virtual machines can be migrated to the new servers. In case of the reverse (where the physical server capacity is not being fully utilized), virtual machines can be consolidated to a smaller number of physical servers and the servers that have been freed up can be powered off to save on electric costs.

## Cloud Software Stack Summary

+ The cloud software stack enables the proivisioning, monitoring and metering of virtual user "resources" on top of a Cloud Service Provider's infrastructure.
+ Cloud Middleware is the overarching platform that coordinates all of the different cloud services and allows them to be accessed as services by users.
+ Provisioning on the cloud refers to the creation of different resources on top of the physical infrastructure. A provisioning system deals with identity and cost management, scheduling resources.
+ Metering is an extremely challenging task on clouds. It requires tracking the utilization of network, storage IOPS, disk and compute capacity, by thousands or millions of concurrent users and mapping into different costs.
+ Orchestration and Automation are the processes that enable Cloud Service Providers to operate at scale, by running workflows and creating environments and configurations without manual intervention.
+ OpenStack is an open-source cloud stack implemenations that contains services for user authentication, provisioning, management, metering, compute, storage among others.


