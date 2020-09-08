VPC peering connection between two VPCs in a single AWS account
===
Terraform 0.12+ module to create a peering connection between two VPCs two allow traffic between the VPCs.
Note that the two VPCs must belong to the same account.
Note that security groups will also be required (not configured here) to allow traffic to flow.
This module creates routes in the default route tables of both sides of the peering connection.

# Examples
```terraform
# Create Public VPC
module "public_vpc" {
  source           = "voquis/vpc-subnets-internet/aws"
  version          = "0.0.3"
  name             = "public-vpc"
  internet_gateway = true
  flowlogs         = true
  cidr_block       = "10.10.0.0/24"
  subnets = [
    # Public subnets
    {
      availability_zone = "euw2-az1"
      # 10.10.0.1 - 10.10.0.63 (62 ip addresses)
      cidr_block              = "10.10.0.0/26"
      map_public_ip_on_launch = true
      name                    = "public_subnet_1"
    },
    {
      availability_zone = "euw2-az2"
      # 10.10.0.64 - 10.10.0.127 (62 ip addresses)
      cidr_block              = "10.10.0.64/26"
      map_public_ip_on_launch = true
      name                    = "public_subnet_2"
    },
    {
      availability_zone = "euw2-az3"
      # 10.10.0.128 - 10.10.0.191 (62 ip addresses)
      cidr_block              = "10.10.0.128/26"
      map_public_ip_on_launch = true
      name                    = "public_subnet_3"
    }
  ]
}

# Create Private VPC (no internet gateway or route)
module "private_vpc" {
  source           = "voquis/vpc-subnets-internet/aws"
  version          = "0.0.3"
  name             = "private-vpc"
  internet_gateway = false
  flowlogs         = true
  cidr_block       = "10.10.1.0/24"
  subnets = [
    # Public subnets
    {
      availability_zone = "euw2-az1"
      # 10.10.1.1 - 10.10.1.63 (62 ip addresses)
      cidr_block              = "10.10.1.0/26"
      map_public_ip_on_launch = false
      name                    = "private_subnet_1"
    },
    {
      availability_zone = "euw2-az2"
      # 10.10.1.64 - 10.10.1.127 (62 ip addresses)
      cidr_block              = "10.10.1.64/26"
      map_public_ip_on_launch = false
      name                    = "private_subnet_2"
    },
    {
      availability_zone = "euw2-az3"
      # 10.10.1.128 - 10.10.1.191 (62 ip addresses)
      cidr_block              = "10.10.1.128/26"
      map_public_ip_on_launch = false
      name                    = "private_subnet_3"
    }
  ]
}

# Fetch current account id, assumes both VPC are in the same region
data "aws_caller_identity" "current" {}

# Main VPC Peering connection module
module "myvpcpeering" {
  source  = "voquis/vpc-peering-connection/aws"
  version = "0.0.2"

  peer_owner_id           = data.aws_caller_identity.current.account_id
  vpc_id                  = module.private_vpc.vpc.id
  vpc_route_table_id      = module.private_vpc.vpc.default_route_table_id
  vpc_cidr_block          = module.private_vpc.vpc.cidr_block
  peer_vpc_id             = module.public_vpc.vpc.id
  peer_vpc_route_table_id = module.public_vpc.vpc.default_route_table_id
  peer_vpc_cidr_block     = module.public_vpc.vpc.cidr_block
}
```
