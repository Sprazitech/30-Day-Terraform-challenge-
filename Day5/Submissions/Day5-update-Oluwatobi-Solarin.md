mananging high traffic applications with aws elastic load balanacer and terraform

main.tf



  min_size            = var.min_size
  max_size            = var.max_size
  desired_capacity    = var.desired_capacity
  vpc_zone_identifier = aws_subnet.private[*].id

  target_group_arns = [aws_lb_target_group.web_tg.arn] # Attach target group here

  // Updated tagging format
  tag {
    key                 = "Name"
    value               = "Web Server Instance"
    propagate_at_launch = true
  }
}


resource "aws_security_group" "ec2_sg" {
  name        = "ec2-security-group"
  description = "Security group for EC2 instances"
  vpc_id      = aws_vpc.main_vpc.id

  # Allow traffic from the Load Balancer's security group
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    security_groups = [aws_security_group.alb_sg.id] # Reference ALB SG
  }

  # Allow SSH access (optional, for management purposes)
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Restrict this to your IP for better security
  }

  # Allow all outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "EC2 Security Group"
  }
}



variable.tf


# AWS Region
variable "aws_region" {
  description = "The AWS region where resources will be deployed"
  type        = string
  default     = "us-east-1"
}

# VPC CIDR
variable "vpc_cidr" {
  description = "The CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

# Public Subnet CIDR Block Prefix
variable "public_subnet_cidr_prefix" {
  description = "CIDR block prefix for public subnets"
  type        = number
  default     = 24
}

# Private Subnet CIDR Block Prefix
variable "private_subnet_cidr_prefix" {
  description = "CIDR block prefix for private subnets"
  type        = number
  default     = 24
}

# Key Pair Name
variable "key_name" {
  description = "Key pair name for SSH access"
  type        = string
  default     = "my-key-pair"
}

# Instance Type
variable "instance_type" {
  description = "Instance type for Auto Scaling group"
  type        = string
  default     = "t2.micro"
}

# Auto Scaling Configuration
variable "min_size" {
  description = "Minimum number of instances in the Auto Scaling group"
  type        = number
  default     = 1
}

variable "max_size" {
  description = "Maximum number of instances in the Auto Scaling group"
  type        = number
  default     = 3
}

variable "desired_capacity" {
  description = "Desired number of instances in the Auto Scaling group"
  type        = number
  default     = 2
}



output.tf


# Output VPC ID
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.main_vpc.id
}

# Public Subnet IDs
output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

# Private Subnet IDs
output "private_subnet_ids" {
  description = "List of private subnet IDs"
  value       = aws_subnet.private[*].id
}

# Internet Gateway ID
output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.rr_igw.id
}

# Load Balancer DNS Name
output "load_balancer_dns_name" {
  description = "DNS name of the Application Load Balancer"
  value       = aws_lb.app_lb.dns_name
}

output "aws_region" {
  description = "AWS region used"
  value       = var.aws_region
}

# Output the private key (note that it is sensitive, so be cautious)
output "private_key" {
  value     = tls_private_key.example.private_key_pem
  sensitive = true # Mark it as sensitive to prevent exposure
}

# Output the public key
output "public_key" {
  value = tls_private_key.example.public_key_openssh
}



provider.tf



terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0" # Adjust to the latest version you're using
    }
  }

  required_version = ">= 1.5.0"
}

provider "aws" {
  region = var.aws_region
}
