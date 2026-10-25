# AWS VPC Peering Project Report
## Private Communication Between Two Amazon Virtual Private Clouds

**Author:** Madhumitha  
**Platform:** Amazon Web Services (AWS)  
**Region:** Asia Pacific (Sydney) — `ap-southeast-2`  
**Project Type:** Cloud Computing / Networking Practical

---

# 1. Introduction

Amazon Web Services (AWS) provides cloud computing services that allow users to create virtual networks, launch servers, and configure secure communication between resources.

An Amazon Virtual Private Cloud (VPC) is a logically isolated virtual network in AWS. A VPC allows users to define IP address ranges, create subnets, configure route tables, and control network traffic.

VPC Peering is an AWS networking feature that allows two VPCs to communicate privately using their private IP addresses. The VPCs can be in the same AWS account or different accounts, provided the required configuration is completed.

In this project, two VPCs were created and connected using VPC Peering. An EC2 instance was launched in each VPC, and connectivity was tested using the Linux `ping` command.

The test was successful, with **0% packet loss** reported from Instance A to Instance B.

# 2. Objectives of the Project

The main objectives were:

1. To understand AWS VPC and subnet configuration.
2. To create two separate VPCs with different IPv4 CIDR ranges.
3. To configure public and private network components.
4. To establish a VPC Peering connection between the two VPCs.
5. To configure route tables for communication between the VPCs.
6. To launch EC2 instances in both VPCs.
7. To configure security groups to allow the required network traffic.
8. To test private communication using the `ping` command.
9. To understand basic AWS networking security and resource cleanup.

# 3. Technologies and Services Used

| Service or Tool | Purpose |
|---|---|
| Amazon VPC | Creates isolated virtual networks |
| VPC Peering | Connects the two VPCs privately |
| Amazon EC2 | Provides virtual Linux servers |
| Subnets | Divide a VPC into smaller network segments |
| Route Tables | Direct network traffic to the correct destination |
| Internet Gateway | Provides an internet route for a public subnet |
| Security Groups | Control permitted traffic to EC2 instances |
| Amazon Linux 2023 | Operating system used for the EC2 instances |
| Linux Terminal | Used to execute network testing commands |

# 4. Network Design

We created two VPCs with non-overlapping IP address ranges.

## VPC A

- **Name:** `ewb-vpc-a`
- **IPv4 CIDR:** `10.10.0.0/16`
- **Internet Gateway:** `ewb-igw-a`
- **Subnets:** Public and private
- **EC2 Instance:** `ewb-ec2-a`
- **Private IP:** `10.10.1.89`

## VPC B

- **Name:** `ewb-vpc-b`
- **IPv4 CIDR:** `10.20.0.0/16`
- **Private subnet:** `ewb-b-private`
- **Subnet CIDR:** `10.20.1.0/24`
- **Route table:** `ewb-rt-b-private`
- **EC2 Instance:** `ewb-ec2-b`
- **Private IP:** `10.20.1.75`

## VPC Peering Connection

- **Name:** `ewb-a-b-peer`
- **Requester:** `ewb-vpc-a`
- **Accepter:** `ewb-vpc-b`
- **Final status during testing:** Active

The two VPCs have different CIDR ranges, which prevents IP address overlap and allows their traffic to be routed correctly.

# 5. Step-by-Step Implementation

## Step 1: Create VPC A

We opened the AWS Management Console and navigated to:

**VPC → Your VPCs → Create VPC**

We selected the **VPC only** option and entered:

- Name: `ewb-vpc-a`
- IPv4 CIDR: `10.10.0.0/16`
- IPv6 CIDR: No IPv6 CIDR
- Tenancy: Default

We then created the VPC.

**Purpose:** This created the first isolated network for our project.

## Step 2: Configure VPC A's networking

We created public and private subnet infrastructure for VPC A and configured the Internet Gateway named `ewb-igw-a`.

The Internet Gateway was attached to VPC A, and the public route table was configured to direct internet-bound IPv4 traffic through the gateway.

A typical public route is:

`0.0.0.0/0 → Internet Gateway`

We also retained the local route for VPC A's own CIDR.

**Purpose:** The public subnet provided a location for Instance A, which we accessed through the AWS console. The private subnet provided additional network segmentation.

## Step 3: Create VPC B

We navigated to **VPC → Your VPCs → Create VPC** and selected **VPC only**.

The configuration was:

- Name: `ewb-vpc-b`
- IPv4 CIDR: `10.20.0.0/16`
- IPv6 CIDR: No IPv6 CIDR
- Tenancy: Default

We clicked **Create VPC**.

**Purpose:** This created the second isolated network that would communicate with VPC A through VPC Peering.

## Step 4: Create VPC B's private subnet

We navigated to **VPC → Subnets → Create subnet**.

We configured:

- VPC: `ewb-vpc-b`
- Subnet name: `ewb-b-private`
- IPv4 CIDR: `10.20.1.0/24`
- Availability Zone: An available Sydney Availability Zone

We created the subnet.

**Purpose:** The subnet provides a smaller network inside VPC B for the private EC2 instance.

## Step 5: Create VPC B's private route table

We opened **VPC → Route tables → Create route table**.

We entered:

- Name: `ewb-rt-b-private`
- VPC: `ewb-vpc-b`

We then opened **Subnet associations → Edit subnet associations**, selected `ewb-b-private`, and saved the association.

The route table retained its local route:

`10.20.0.0/16 → local`

**Purpose:** A route table determines where traffic originating from a subnet should be sent. We would later add a route to VPC A through the peering connection.

## Step 6: Create the VPC Peering connection

We navigated to **VPC → Peering connections → Create peering connection**.

We configured:

- Name: `ewb-a-b-peer`
- Requester VPC: `ewb-vpc-a`
- Account: My account
- Region: This Region
- Accepter VPC: `ewb-vpc-b`

We created the connection and accepted the peering request.

The connection became active.

**Purpose:** This established the logical connection between the two VPCs. However, creating the connection alone does not automatically configure routing.

## Step 7: Add the peering routes in VPC A

VPC A had two route tables associated with its public and private subnets.

We opened each route table and selected:

**Routes → Edit routes → Add route**

We added the following route to both route tables:

- Destination: `10.20.0.0/16`
- Target: `ewb-a-b-peer` (the VPC Peering connection)

We saved the changes.

The existing local routes and the public route table's Internet Gateway route were retained.

**Purpose:** This tells VPC A to send traffic destined for VPC B through the peering connection.

## Step 8: Add the peering route in VPC B

We selected `ewb-rt-b-private` and opened:

**Routes → Edit routes → Add route**

We added:

- Destination: `10.10.0.0/16`
- Target: `ewb-a-b-peer`

We saved the changes.

**Purpose:** This tells VPC B how to reach VPC A. Routes are needed in both directions for communication to work correctly.

## Step 9: Launch EC2 Instance A

We opened **EC2 → Instances → Launch instances**.

We configured:

- Name: `ewb-ec2-a`
- AMI: Amazon Linux 2023
- Instance type: A suitable micro instance
- VPC: `ewb-vpc-a`
- Subnet: VPC A's public subnet
- Auto-assign public IP: Enabled
- Security group: Created or selected with appropriate access rules

We launched the instance and waited until its status checks passed.

The instance's private IPv4 address was:

`10.10.1.89`

**Purpose:** Instance A served as the source for the connectivity test.

## Step 10: Launch EC2 Instance B

We launched a second EC2 instance with these settings:

- Name: `ewb-ec2-b`
- AMI: Amazon Linux 2023
- Instance type: Same or equivalent to Instance A
- VPC: `ewb-vpc-b`
- Subnet: `ewb-b-private`
- Auto-assign public IP: Disabled
- Security group: Configured for the required test traffic

The instance's private IPv4 address was:

`10.20.1.75`

**Purpose:** Instance B served as the destination for the connectivity test. It was placed in a private subnet without a public IPv4 address.

## Step 11: Configure the security group for ping

Initially, the ping test produced no replies.

We opened Instance B's security group and added an inbound rule:

- Type: All ICMP - IPv4
- Source: `10.10.0.0/16`
- Description: Allow ping from VPC A

We saved the rule.

**Purpose:** Security groups control which traffic is allowed to reach an instance. ICMP traffic must be allowed for the ping test to receive replies.

The source restriction allowed ICMP traffic from VPC A's CIDR rather than from the entire internet.

## Step 12: Connect to Instance A

We selected `ewb-ec2-a` and chose **Connect → EC2 Instance Connect**.

A Linux terminal opened in the browser.

We ran:

```bash
hostname
```

to check the host name, and:

```bash
hostname -I
```

to display the instance's IP addresses.

We confirmed that the terminal was connected to Instance A.

**Purpose:** This provided a command-line environment from which to test private network connectivity.

## Step 13: Test connectivity between the VPCs

From Instance A's terminal, we ran:

```bash
ping 10.20.1.75
```

Initially, no replies appeared because the required ICMP traffic was not allowed by Instance B's security group.

After adding the ICMP rule, we ran the command again and received replies similar to:

```text
64 bytes from 10.20.1.75: icmp_seq=3 ttl=127 time=0.204 ms
```

The ping command continued sending requests until we stopped it with:

**Ctrl + C**

The final result showed:

**Packet loss: 0%**

**Purpose:** This confirmed that Instance A could reach Instance B using its private IPv4 address through the configured network path.

The test verified connectivity from A to B. A separate test from B to A would be required to independently verify the reverse direction.

# 6. Route Table Summary

| Route table | Destination | Target |
|---|---|---|
| VPC A public route table | `10.20.0.0/16` | VPC Peering |
| VPC A private route table | `10.20.0.0/16` | VPC Peering |
| VPC B private route table | `10.10.0.0/16` | VPC Peering |

Each route table also retains its local route. The public route table in VPC A retains its Internet Gateway route.

These entries allow traffic destined for the other VPC to use the peering connection.

# 7. Results and Observations

The project was successfully configured and tested.

The main observations were:

1. Both VPCs were created with separate IPv4 CIDR ranges.
2. The VPC Peering connection was accepted and became active.
3. Routes were added to both VPC A route tables and VPC B's private route table.
4. Two EC2 instances were launched in their intended VPCs.
5. The initial ping test did not receive replies.
6. An inbound ICMP security-group rule was added to Instance B.
7. The ping test then returned successful replies.
8. The final ping summary reported 0% packet loss.

These results demonstrate successful ICMP connectivity from VPC A to VPC B over the private network.

# 8. Learning Outcomes

Through this practical, I learned the following concepts.

## 8.1 Amazon VPC

A VPC is a logically isolated virtual network in AWS. It allows users to define address ranges and control how cloud resources communicate.

## 8.2 CIDR and IP Addressing

CIDR notation defines network address ranges. For example, `10.10.0.0/16` and `10.20.0.0/16` represent separate network ranges.

Using non-overlapping ranges is important when connecting VPCs through peering.

## 8.3 Public and Private Subnets

A public subnet generally has a route to an Internet Gateway. A private subnet does not have a direct route to an Internet Gateway.

A subnet is not made public merely by its name; its routing and associated settings determine its network behavior.

## 8.4 Route Tables

Route tables determine where traffic is sent. VPC Peering requires routes for the remote VPC's CIDR in the relevant route tables.

## 8.5 VPC Peering

VPC Peering provides private IP connectivity between two VPCs. It does not automatically create routes or grant internet access.

## 8.6 Security Groups

Security groups act as virtual firewalls for EC2 instances. The ICMP rule was necessary to allow ping requests to reach Instance B and receive replies.

## 8.7 EC2 and Linux Commands

I learned how to launch EC2 instances, connect to a Linux terminal, inspect host information, and use the `ping` command to test network reachability.

## 8.8 Troubleshooting

The first ping test did not receive replies. By checking and updating Instance B's security group to allow ICMP traffic from VPC A, I resolved the issue and successfully completed the test.

## 8.9 Cloud Resource Management

AWS resources may incur charges. It is important to check billing implications, save project evidence, and remove resources that are no longer required.

# 9. Security and Cost Considerations

The project highlighted several important practices:

- Use the principle of least privilege when configuring security groups.
- Restrict SSH access to trusted IP addresses rather than allowing it from everywhere.
- Never upload AWS access keys, private key files, passwords, or credentials to GitHub.
- Remember that a successful ping confirms ICMP reachability, not that every application protocol is working.
- Check for resources that may incur charges, including EC2 instances, public IPv4 addresses, and any other billable networking services.
- Before deleting resources, save screenshots and documentation required for the project report.

# 10. Conclusion

This project demonstrated how to establish private communication between two Amazon Virtual Private Clouds using VPC Peering.

I created two VPCs with non-overlapping CIDR ranges, configured public and private subnet infrastructure, established a peering connection, and added the required routes to the route tables. I then launched two EC2 instances and configured a security-group rule to permit ICMP traffic for connectivity testing.

The initial ping test did not receive replies because the required ICMP traffic was not permitted. After updating the security group, Instance A successfully received replies from Instance B using its private IP address. The final test reported 0% packet loss.

The practical improved my understanding of AWS networking, VPCs, subnets, route tables, VPC Peering, EC2 instances, security groups, and basic Linux network troubleshooting.

Overall, the project achieved its primary objective of demonstrating private connectivity between two VPCs and provided useful hands-on experience with cloud networking and security.

---