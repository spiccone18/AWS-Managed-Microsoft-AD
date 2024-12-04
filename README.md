# AWS Lab: Setting up Domain-Joined EC2 Windows Instances with AWS Managed Microsoft Active Directory

## Overview
This guide explains how to set up an AWS lab environment where EC2 Windows instances are joined to an AWS Managed Microsoft Active Directory (AD). The lab is designed to simulate a corporate environment with redundancy, security, and scalability.

---

## Architecture Diagram
Below is the architecture of the lab setup:

![AWS Lab Architecture](docs/architecture-diagram.png)

---

## Step-by-Step Instructions

### 1. **Create a New VPC**
- Navigate to the **VPC Dashboard** in AWS Management Console.
- Click **Create VPC** > **VPC and More**.
- Configure the VPC:
  - **VPC Name:** `AD-VPC`
  - **IPv4 CIDR Block:** `10.0.0.0/16`
  - Enable DNS resolution and DNS hostname options.
- Add the following subnets:
  - **Public Subnet:** `Public-Subnet-1` with CIDR `10.0.1.0/24`.
  - **AD Subnet 1:** `AD-Subnet-1` with CIDR `10.0.2.0/24`.
  - **AD Subnet 2:** `AD-Subnet-2` with CIDR `10.0.3.0/24`.
  - **Management Subnet:** `Mgmt-Subnet` with CIDR `10.0.4.0/24`.

### 2. **Attach an Internet Gateway to the VPC**
- Go to the **Internet Gateway** section in the VPC Dashboard.
- Click **Create Internet Gateway** and name it `AD-VPC-IGW`.
- Attach it to the `AD-VPC` you just created.

### 3. **Create and Associate Route Tables**
- **Public Route Table:**
  - Navigate to the **Route Tables** section.
  - Create a route table named `Public-Route-Table`.
  - Add a route for `0.0.0.0/0` and associate it with `AD-VPC-IGW`.
  - Associate this route table with the `Public-Subnet-1`.
- **Private Route Table:**
  - Create a route table named `Private-Route-Table`.
  - Add no default routes.
  - Associate this route table with `AD-Subnet-1` and `AD-Subnet-2`.

### 4. **Launch AWS Managed Microsoft AD**
- Navigate to the **Directory Service** section in AWS Management Console.
- Click **Set up directory** and select **AWS Managed Microsoft AD**.
- Choose the standard edition.
- Use the following configuration:
  - **VPC:** Select `AD-VPC`.
  - **Subnets:** Assign `AD-Subnet-1` and `AD-Subnet-2`.

### 5. **Update Security Groups**
- Create a security group named `AD-SG` with the following rules:
  - **Inbound Rules:**
    - Allow **RDP (3389)** from your IP (for management).
    - Allow **LDAP (389)** and **LDAPS (636)** between subnets.
  - **Outbound Rules:**
    - Allow all traffic for now.

### 6. **Launch a Bastion Host**
- Launch a new EC2 instance in `Public-Subnet-1` with the following settings:
  - **Instance Type:** `t2.micro` (or larger, based on needs).
  - **AMI:** Amazon Linux 2.
  - **Key Pair:** Select or create one.
  - **Security Group:** Allow RDP (3389) from your IP.
  - Install the required tools for RDP and management.

### 7. **Launch an EC2 Windows Instance**
- Navigate to the **EC2 Dashboard** and launch a new instance:
  - **AMI:** Windows Server 2022.
  - **Instance Type:** `t2.medium` or larger.
  - **Subnet:** `Mgmt-Subnet`.
  - **IAM Role:** Attach a role with the `AmazonSSMManagedInstanceCore` policy.
  - **Storage:** Minimum 50GB.
  - **Security Group:** Use `AD-SG`.

### 8. **Join the Instance to the Domain**
- Connect to the instance using RDP.
- Open the **Server Manager** and select **Local Server** > **Workgroup** > **Change**.
- Select **Domain** and enter your AD domain name.
- Provide credentials for an AD user account with join permissions.
- Restart the instance to complete the domain join.

---

## Additional Notes
1. **High Availability**:
   - AWS Managed AD deploys two domain controllers (one in each availability zone) by default for redundancy.
2. **Expanding the Domain**:
   - To add a new forest for a new office, repeat the directory setup process in a different VPC.
   - Alternatively, create an Organizational Unit (OU) under the existing domain for logical separation.

