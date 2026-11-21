# Cloud — Module 1: Cloud Concepts

Part of my data engineering study trail. This module covers the core cloud computing concepts that also open the AWS Certified Cloud Practitioner (CLF-C02) exam: service models, deployment models, the shared responsibility model, and how AWS organizes its global infrastructure.

## Key concepts

1. **Cloud computing** is the on-demand delivery of IT resources (servers, storage, databases, networking) over the internet, with pay-as-you-go pricing.
2. **Service models** go from more control to more convenience:
   - **IaaS:** the provider gives the infrastructure (virtual machine, network, disk) and the customer manages the operating system and everything above it. Example: Amazon EC2.
   - **PaaS:** the provider manages the platform, and the customer manages the code and the data. Example: AWS Elastic Beanstalk.
   - **SaaS:** the software is ready to use, and the customer only uses it. Example: Gmail.
3. **Deployment models:** **public** cloud (resources at the provider), **private** cloud (own infrastructure) and **hybrid** cloud (a mix of both).
4. **Shared responsibility model** (the most tested concept): AWS is responsible for security **OF** the cloud (hardware, data centers, global network, and the software of managed services). The customer is responsible for security **IN** the cloud (their data, permissions, configuration and encryption).
5. **The boundary changes with the service.** On an EC2 instance, the customer patches the guest operating system. On a managed service such as RDS or S3, AWS takes care of the operating system and the engine, but the customer is still responsible for who can access the data and how it is configured.
6. **Global infrastructure:**
   - **Region:** a geographic area with several Availability Zones, isolated from other regions.
   - **Availability Zone (AZ):** one or more data centers with independent power, cooling and networking, connected to the other AZs of the same region through low-latency links.
   - **Edge locations:** cache points used by CloudFront, outside the regions.
7. **How to choose a region:** legal and data residency requirements (for example LGPD in Brazil), latency to users, cost, and which services are available in that region. AWS has a region in São Paulo (`sa-east-1`).
8. **High availability:** to survive the failure of an entire data center, distribute the application across **multiple AZs of the same region**. A single AZ is a single point of failure.
9. **Six advantages of cloud computing:**
   1. Trade fixed expense for variable expense.
   2. Benefit from massive economies of scale.
   3. Stop guessing capacity.
   4. Increase speed and agility.
   5. Stop spending money on running and maintaining data centers.
   6. Go global in minutes.

## Quick reference

| Concept | One-line summary |
|---|---|
| IaaS / PaaS / SaaS | You manage OS and up / only code and data / nothing, just use it |
| Public / private / hybrid | Provider's resources / own infrastructure / both together |
| Security OF the cloud | AWS: physical security, hardware, global infrastructure, managed service software |
| Security IN the cloud | Customer: data, IAM permissions, security groups, encryption settings, guest OS patches on EC2 |
| Region | Geographic area with multiple AZs |
| AZ | One or more data centers with independent power and network |
| Edge location | CloudFront cache point outside regions |

## Practice questions

**1. A company runs its application on an EC2 instance. Who is responsible for applying security patches to the instance's operating system?**
The customer. On EC2 (IaaS), AWS handles the hardware and virtualization, but the guest operating system (patches, configuration, updates) belongs to the customer. On managed services such as RDS, that task moves to AWS.

**2. A team wants to deliver only the code and data of the application, leaving the provider to manage servers, operating system and runtime. Which service model is this?**
PaaS. In IaaS the customer manages the OS and above, and in SaaS the software already comes ready. Hybrid cloud is a deployment model, not a service model.

**3. What is an Availability Zone (AZ)?**
One or more data centers with independent power and networking inside a region. CloudFront's cache points are edge locations, not AZs.

**4. An application must keep working even if an entire data center fails. What is the correct approach?**
Distribute the application across multiple AZs of the same region. Each AZ has independent power and networking, while a single AZ is a single point of failure. The price of a region has no relation to resilience.

**5. In the shared responsibility model, which item is AWS's responsibility (security OF the cloud)?**
The physical security of the data centers. Security group rules, IAM permissions and the encryption configuration are the customer's responsibility (security IN the cloud).

## Takeaways

- Always ask "who manages this layer?" The answer depends on the service model: the more the provider manages (SaaS), the less the customer is responsible for.
- Shared responsibility is the most tested topic. Remember: AWS secures the cloud itself, the customer secures what they put and configure in it.
- Hybrid cloud is a deployment model. IaaS, PaaS and SaaS are service models. Do not mix the two groups.
- For high availability, think multi-AZ inside one region. For disaster recovery or lower latency in other countries, think multi-region.
- Choosing a region is a mix of compliance, latency, cost and service availability, and price alone should not decide it.

## Next

Module 2: core AWS services (EC2, S3, IAM, RDS).
