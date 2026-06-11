
## Deep Dive into AWS EC2: Architecture, Lifecycle, Purchasing Models, Instance Metadata (IMDSv2), and Automated Provisioning for Enterprise Microservices

## 🎯 Learning Objectives

-   Understand EC2 core components, virtualization layer, and decoupling of storage/compute.
    
-   Analyze EC2 purchasing strategies to balance cost, availability, and fault tolerance.
    
-   Master instance bootstrapping using user-data and verify scripts via system logs.
    
-   Differentiate between Instance Metadata (IMDS) and User-Data for runtime configuration.
    
-   Connect securely to Linux instances using SSH/Key Pairs and troubleshoot common connection failures.
    

## 📝 Session Summary

This session establishes the foundational compute pillar of AWS: Amazon Elastic Compute Cloud (EC2). We covered the structural building blocks of an instance (AMIs, Instance Types, EBS, and Security Groups), evaluated the seven distinct purchasing models, and demonstrated how to automate package installation via User-Data scripts. Finally, we reviewed the mechanics of Instance Metadata (`169.254.169.254`) and secure terminal connectivity via SSH.

## Corrected & Enhanced Notes

### 1. Amazon EC2 Core Architecture

Amazon Elastic Compute Cloud (EC2) provides secure, resizable compute capacity in the cloud. In a traditional data center, spinning up a server requires hardware procurement, racking, provisioning, and hypervisor management. EC2 abstracts this into a multi-tenant or single-tenant virtualized environment managed via APIs.

-   **The Decoupled Compute Model**: An EC2 instance is inherently transient by default unless backed by persistent storage. CPU and RAM are allocated from the physical host hardware, while storage is typically attached over a dedicated storage network via Amazon Elastic Block Store (EBS).
    
-   **Operating System Support**: Highly flexible, supporting Amazon Linux 2023 (AL2023), Red Hat Enterprise Linux (RHEL), Ubuntu, CentOS, SUSE Linux, Microsoft Windows Server, and macOS (running on dedicated bare-metal instances).
    

### 2. Core Features Breakdown

-   **Instance**: The virtual computing environment itself, running on Nitro or Xen Hypervisors.
    
-   **Tags**: Key-value pairs assigned to AWS resources. They are critical for cost allocation (Cost Allocation Tags), automation scripts, and IAM attribute-based access control (ABAC).
    
-   **Amazon Machine Image (AMI)**: A pre-configured template containing the OS, application server, and initial configurations. You can use AWS-provided AMIs, Marketplace AMIs, or create custom AMIs (Golden Images) baked with your specific Java runtime environment (JRE) and monitoring agents.
    
-   **Instance Type**: Categorized families optimized for distinct workloads:
    
    -   **General Purpose (M, T)**: Balanced CPU, memory, and networking. _T-families use burstable performance credits._
        
    -   **Compute Optimized (C)**: High-performance processors. Ideal for batch processing, high-performance web servers, and dedicated modeling.
        
    -   **Memory Optimized (R, X, Z)**: Designed for fast performance for workloads that process large data sets in memory (e.g., distributed caching like Redis, or high-throughput Spring Boot applications).
        
    -   **Storage Optimized (I, D, H)**: High sequential read/write for local data sets (e.g., distributed databases, Kafka brokers).
        
    -   **Accelerated Computing (P, G, F)**: Hardware accelerators (GPUs, FPGAs) for ML and graphics rendering.
        
-   **Elastic Block Storage (EBS)**: Network-attached virtual hard disks providing persistent block-level storage. Unlike local instance store volumes, data on EBS survives instance stop/start operations.
    
-   **Security Groups**: Stateful, virtual firewalls operating at the **instance network interface (ENI) level** to control inbound and outbound traffic.
    
-   **Key Pairs**: Asymmetric cryptography used to authenticate your terminal session. AWS stores the public key inside the instance guest OS (`~/.ssh/authorized_keys`), and you maintain the private key (`.pem` or `.ppk`).
    

### 3. Deep Dive: EC2 Purchasing Models

Selecting the right purchasing model directly impacts your operational architecture and cost efficiency.
### EC2 Purchasing Models Comparison

| Feature | On-Demand | Savings Plans | Reserved Instances (RI) | Spot Instances | Dedicated Hosts | Dedicated Instances | Capacity Reservations |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Commitment** | None (Pay-by-the-second) | 1 or 3-year usage commit ($/hr) | 1 or 3-year tenure commit | None (Subject to AWS capacity) | None or 1/3 year commit | None or 1/3 year commit | None (Specify duration) |
| **Cost Profile** | Baseline / Highest standard rate | Up to 72% savings vs On-Demand | Up to 72% savings vs On-Demand | Up to 90% savings vs On-Demand | High cost (Billed per physical host) | Premium charge over standard multi-tenant | Standard On-Demand rates apply |
| **Flexibility** | Highest (Change size/type instantly) | High (Applies across family, region, Fargate) | Low/Medium (Convertible RIs allow shifts) | None (Instance can be reclaimed) | Low (Tied to physical hardware) | Medium (Isolated instance placement) | High (Reserve target AZ capacity) |
| **Interruption Risk** | Zero risk of AWS reclaiming capacity | Zero risk of AWS reclaiming capacity | Zero risk of AWS reclaiming capacity | High risk (**2-minute reclamation warning**) | Zero risk of AWS reclaiming capacity | Zero risk of AWS reclaiming capacity | Zero risk of AWS reclaiming capacity |
| **Primary Use Case** | Unpredictable, short-term dev/test workloads | Consistent, steady microservices across mixed compute | Fixed, unyielding long-term production systems | Stateless, fault-tolerant batch processors, EKS workers | Bring Your Own License (BYOL) compliance (Oracle/MS) | Strict physical tenant isolation mandates | Disaster recovery drills, high-traffic events |
### 4. Instance Bootstrapping & User Data

**Bootstrapping** means executing configuration scripts automatically during the very first boot lifecycle of the instance.

-   **Execution Lifecycle**: By default, User Data scripts run **exactly once** when the instance is created. If you stop and restart the instance, the script does _not_ re-run (unless explicitly configured via cloud-init directives).
    
-   **Privileges**: The script executes automatically as the `root` user. Therefore, using `sudo` within your user-data script is redundant but harmless.
    

#### Corrected & Production-Ready User Data Script

Bash

```
#!/bin/bash
# Update the package manager repositories safely
yum update -y

# Install Apache Web Server (httpd)
yum install httpd -y

# Start the web server daemon
systemctl start httpd

# Enable the web server daemon to start automatically on system reboots
systemctl enable httpd

# Inject a basic diagnostic landing page
echo "<h1>Welcome to Webserver from EC2 Bootstrapping!</h1>" > /var/www/html/index.html

```

#### Troubleshooting and Logs Verification

If your application or server does not spin up as expected, do not guess. Inspect the boot-cycle automation engine logs inside the guest OS:

-   `/var/log/cloud-init.log`: Logs the operational stages of the `cloud-init` initialization system.
    
-   `/var/log/cloud-init-output.log`: **Critical for debugging scripts.** Captures all `stdout` and `stderr` outputs generated by your User Data execution.
    

### 5. Instance Metadata Service (IMDS)

Instance Metadata represents internal endpoints providing configuration data about the running instance (e.g., its public/private IP addresses, instance ID, security groups, IAM role credentials).

> ⚠️ **Critical Security Warning**: The old method of fetching metadata via `IMDSv1` used a simple, unauthenticated HTTP GET request to the link-local IP `169.254.169.254`. This made systems vulnerable to Server-Side Request Forgery (SSRF) attacks. Modern production environments enforce **IMDSv2**, which requires a session token.

#### Fetching Metadata via IMDSv2 (Secure Way)

To query metadata under IMDSv2, you must first issue a `PUT` request to generate a cryptographically signed token, then pass that token inside an HTTP header in your subsequent `GET` request.

Bash

```
# Step 1: Generate a session token valid for 600 seconds (10 minutes)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 600")

# Step 2: Use that token to query the metadata hierarchy safely
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/

# Example: Fetching the specific unique Instance ID
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id

```

#### Fetching User Data via Meta-Endpoint

The user-data script passed during provisioning can also be fetched at runtime by the instance itself:

Bash

```
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/user-data/

```

### 6. Linux Connectivity & SSH Architecture

To establish an interactive terminal connection to a remote Linux instance via port 22, asymmetric key-pair cryptography is required.

#### Default Built-In OS Usernames

Using the incorrect username will result in a `Permission denied (publickey)` failure.

-   **Amazon Linux 2 / Amazon Linux 2023**: `ec2-user`
    
-   **Ubuntu Linux**: `ubuntu`
    
-   **CentOS**: `centos`
    
-   **Red Hat Enterprise Linux (RHEL)**: `ec2-user` or `root`
    
-   **SUSE Linux**: `ec2-user` or `root`
    

#### Connecting from Modern Terminals (PowerShell / Linux Terminal / macOS)

Ensure permissions on your private key file are locked down (on Unix systems, run `chmod 400 sample.pem` so that it is readable only by the owner).

Bash

```
ssh -i "sample.pem" ec2-user@35.172.219.8

```

#### Connecting from Legacy Systems (PuTTY Workflow)

-   **PuTTY** cannot native-read a `.pem` file directly.
    
-   **PuTTYgen** must be opened first to import the `.pem` file and convert/save it as a native `.ppk` (PuTTY Private Key) file format.
    
-   The `.ppk` file is then loaded into PuTTY's SSH Auth configurations (`Connection -> SSH -> Auth -> Credentials`) to establish a connection.
    

## Mistakes / Corrections Found

-   **Typo in Curl Tool Commands**: The raw notes explicitly typed `Scurl http://...`. The leading `S` was a typo or terminal artifact. Corrected to standard executable command `curl`.
    
-   **Outdated IMDS Reference**: The original note showcased an unauthenticated request structure (`curl http://169.254.169.254/latest/meta-data/`). This is an `IMDSv1` implementation. This has been updated to the current AWS best practice of using **IMDSv2 tokens** to align with AWS Certified Developer security criteria.
    
-   **Typo in Commands**: The note contained `$sudo-i`. The correct syntactic command requires a space: `sudo -i` or `sudo su -`.
    
-   **Typo**: "power shall" was corrected to the correct tool name: **PowerShell**.
    

## Discussion Points / Clarifications Needed

1.  **Stateful Security Groups vs Stateless NACLs**: A common point of confusion is how firewalls behave. Remember that if you allow inbound traffic on Port 22 in a Security Group, the outbound reply traffic is _automatically allowed_ regardless of outbound rules because Security Groups are **stateful**. Network Access Control Lists (NACLs), operating at the subnet level, are **stateless** and require explicit rules in both directions.
    
2.  **Instance Store vs EBS**: Remember that certain instance types come with local NVMe drive allocations called "Instance Store". Unlike EBS volumes, data on Instance Store is entirely lost if the instance is stopped or suffers a underlying hardware host failure. It is strictly temporary scratch space.
    
3.  **When does User Data execute?**: A common misconception is that user data executes every time an instance powers on. It only runs on the first boot. If an automation task needs to run on every boot cycle, alternative tools like crontabs, systemd services, or customized cloud-init configs must be configured.
    

## 🔥 High Probability Exam Topics Covered

-   **Spot Instance Interruptions**: Managing stateless microservices using Spot Instances and understanding that AWS provides a **2-minute warning** via Amazon EventBridge and Instance Metadata before reclaiming capacity.
    
-   **IMDSv2 Authentication Flow**: Differentiating between IMDSv1 and IMDSv2. Expect exam items where a security audit flags exposed metadata endpoints; the solution is always enforcing token-based `IMDSv2`.
    
-   **User Data Debugging Path**: Recognizing that script logs are written to `/var/log/cloud-init-output.log` when an EC2 instance fails to bootstrap correctly during standard deployment testing.
    

## 🎯 Key Interview Topics Covered

-   **EC2 Instance Sizing & Memory Bottlenecks**: High-throughput Spring Boot microservices heavily utilize heap space. For Java workloads, understanding that **Memory-Optimized instances (R family)** are often preferred over Compute-Optimized (C family) prevents runtime OutOfMemoryErrors (OOM).
    
-   **Automating CI/CD Golden Images**: Transitioning from slow runtime bootstrapping (installing JRE, updates, agents on boot) to baking custom **AMIs** using tools like HashiCorp Packer to speed up Auto Scaling scaling events.
    

## Service Comparisons

### On-Demand vs Spot vs Savings Plans

-   **Use On-Demand**: When deploying unproven applications, running ad-hoc profiling sessions, or executing short-term proof of concepts where workloads cannot be interrupted.
    
-   **Use Spot Instances**: For distributed microservice application architectures that are stateless (e.g., containerized application runners on EKS/ECS), background batch consumers pulling from SQS queues, or stateless Jenkins worker nodes.
    
-   **Use Savings Plans**: For core underlying stateful services, primary production APIs, and relational databases running 24/7 with zero expected downtime over a 1 to 3-year road map.
    

## Common Exam Traps

-   **Trap**: A question describes an application that can easily recover from sudden compute dropouts and asks for the most cost-effective EC2 billing framework.
    
    -   _Distractor_: Dedicated Instances or Reserved Instances.
        
    -   _Correct Strategy_: **Spot Instances** always yield the highest savings (up to 90%) for fault-tolerant architectures.
        
-   **Trap**: An EC2 instance fails to initialize its application environment on boot, and the developer cannot find the cause using CloudTrail logs.
    
    -   _Distractor_: Inspecting IAM execution roles or standard CloudWatch system metrics.
        
    -   _Correct Strategy_: Connect to the machine and inspect `/var/log/cloud-init-output.log`. User-data stdout is internal to the operating system and is not captured by generic AWS management-plane audit trails.
        

## Potential Certification Questions

1.  **A developer is provisioning an EC2 instance running Amazon Linux 2023 to host a core API layer. The security policy dictates that the metadata service must be fortified against SSRF (Server-Side Request Forgery) attacks. Which configuration choice fulfills this standard?**
    
    -   A. Use a custom IAM policy attached to the instance that blocks outbound communication to `169.254.169.254`.
        
    -   B. Configure the instance to strictly require Instance Metadata Service Version 2 (IMDSv2).
        
    -   C. Deploy the instance into a completely isolated private subnet with zero access to internet gateways.
        
    -   D. Utilize an encrypted Amazon EBS root volume using an AWS KMS customer managed key.
        
    -   _Correct Answer: B_
        
2.  **A batch-processing application executes stateless processing tasks that can be safely interrupted and restarted without data loss. What is the most cost-effective compute choice for this setup?**
    
    -   A. On-Demand EC2 Instances
        
    -   B. Dedicated Hosts
        
    -   C. Spot Instances
        
    -   D. Capacity Reservations
        
    -   _Correct Answer: C_
        

## Potential Interview Questions

1.  **How do you troubleshoot a scenario where a fresh EC2 instance launched successfully, but your automated shell script configured inside the User Data block failed to set up your Java runtime?**
    
    -   _Answer_: I would SSH into the instance using its key pair and verify the execution logs. The specific logs are stored at `/var/log/cloud-init-output.log`. This file captures the standard output and error output of the user-data script during execution. I would check for broken repository links, missing dependencies, or syntax errors inside that file.
        
2.  **If a Spring Boot microservice running inside an EC2 instance needs to discover its own private IP address or determine the Availability Zone it is running in without using external service registries, how can it do so?**
    
    -   _Answer_: The application can query the local Link-Local Address endpoint `http://169.254.169.254/latest/meta-data/`. By using IMDSv2, the application would first issue a PUT request to fetch a session token and then perform a GET request to `/latest/meta-data/local-ipv4` and `/latest/meta-data/placement/availability-zone`.
        

## Session & Revision Summary

Amazon EC2 provides scalable, decoupled virtual compute instances built on varying hardware families (General, Compute, Memory, Storage, Accelerated). Organizations optimize expenditure using On-Demand, Savings Plans, Reserved, Spot, or Dedicated host options. Bootstrapping automates setup at the OS layer via User Data, which logs internal output straight to `/var/log/cloud-init-output.log`. Safe data discovery is managed via secure token-based Instance Metadata (IMDSv2) at the specialized local address `169.254.169.254`.

## Key Takeaways

-   **IMDSv2**: Always remember the IP address **`169.254.169.254`**. For IMDSv2, it requires a two-step handshake: a `PUT` token request followed by a tokenized `GET` header request.
    
-   **User Data Lifecycle**: Runs **once** during the initial instance boot cycle as the root user.
    
-   **Default SSH Users**: `ec2-user` for Amazon Linux/RHEL, `ubuntu` for Ubuntu machines.
    
-   **Spot Instances**: Offers up to 90% cost reductions but can be terminated with a 2-minute notice. Excellent for stateless systems.
    

### 🛠️ Java Spring Boot Integration Context (For Reference)

When writing a Spring Boot service hosted on EC2, you can use the **AWS Java SDK v2** (`software.amazon.awssdk:imds`). The SDK natively abstracts the IMDSv2 token handshake:

Java

```
import software.amazon.awssdk.imds.Ec2MetadataClient;
import software.amazon.awssdk.imds.internal.DefaultEc2MetadataClient;

public class InstanceIdentityService {
    public void printInstanceDetails() {
        try (Ec2MetadataClient client = DefaultEc2MetadataClient.create()) {
            String instanceId = client.get("/latest/meta-data/instance-id").asString();
            System.out.println("Running on EC2 Instance ID: " + instanceId);
        }
    }
}

```

This avoids manual curl implementation inside your Java codebase while enforcing robust IMDSv2 patterns.
