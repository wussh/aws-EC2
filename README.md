# AWS VPC, Subnets, Security Group, Key Pair, and EC2 Instance with Terraform

## Overview
This project provisions a basic AWS network and compute environment using Terraform. It includes:
- A VPC
- A public subnet (with internet access)
- A private subnet
- An Internet Gateway
- A route table for the public subnet
- A security group allowing SSH and HTTP
- An EC2 instance in the public subnet
- An SSH key pair (with instructions to save the PEM file locally)

## Resources

### VPC
Creates a VPC with CIDR block `10.0.0.0/16`.

### Subnets
- **Public Subnet:** `10.0.1.0/24`, mapped to public IPs on launch, in `ap-southeast-1a`.
- **Private Subnet:** `10.0.2.0/24`, in `ap-southeast-1a`.

### Internet Gateway & Route Table
- **Internet Gateway:** Attached to the VPC for internet access.
- **Route Table:** Default route (`0.0.0.0/0`) to the Internet Gateway, associated with the public subnet.

### Security Group
Allows inbound SSH (22) and HTTP (80) from anywhere. All outbound traffic is allowed.

### Key Pair
A 4096-bit RSA key pair is generated. The public key is uploaded to AWS. The private key can be exported locally (see below).

### EC2 Instance
- Ubuntu 24.04, type `t2.micro`
- Placed in the public subnet
- Associated with the security group and key pair

## Usage

1. **Initialize Terraform:**
   ```sh
   terraform init
   ```
2. **Apply the configuration:**
   ```sh
   terraform apply
   ```
   Review and approve the plan.

3. **Export the PEM key (after apply):**
   - Add this output block to `main.tf` if not present:
     ```hcl
     output "key_pair_private_key_pem" {
       value     = tls_private_key.ec2_key.private_key_pem
       sensitive = true
     }
     ```
   - Run:
     ```sh
     terraform output -raw key_pair_private_key_pem > ec2-key.pem
     chmod 600 ec2-key.pem
     ```
   - Remove the output block from `main.tf` for security after saving the key.

4. **Get EC2 connection details:**
   ```sh
   terraform output
   ```
   This will show:
   - `ec2_public_ip`: Public IP of the instance
   - `ec2_private_ip`: Private IP of the instance
   - `ec2_name`: Name tag
   - `ec2_user`: Username ("ubuntu")

5. **SSH into the instance:**
   ```sh
   ssh -i ec2-key.pem ubuntu@<ec2_public_ip>
   ```

## Outputs
- `ec2_public_ip`: Public IP address
- `ec2_private_ip`: Private IP address
- `ec2_name`: Name tag
- `ec2_user`: Default username (ubuntu)

## Notes
- The private key is sensitive. Never commit it to version control.
- The public subnet is open to the internet. For production, restrict security group rules as needed. 