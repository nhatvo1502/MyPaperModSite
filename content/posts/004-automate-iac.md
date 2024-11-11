---
title: 'build AWS network infrastructure and deploy resources using Terraform'
date: 2024-01-16T11:32:20-08:00
draft: true
author: Nhat Vo
categories: 
    - Terraform
    - AWS
    - GitHub Action
---

### 1. Source Code
```
https://github.com/nhatvo1502/Automate-IaC/tree/main
```

### 2. What the code does
#### 2.1. create-resource.yml
- configure aws credential
    - verify aws credential
        - verify aws s3 permission
            - create s3 bucket for terraform backend
                - copy file from GH to runner
                    - verify terraform file
                        - terraform init -> apply

#### 2.2 destroy-resource.yml
- configure aws credential
    - copy file from GH to runner
        - verify terraform file
            - terraform init -> destroy
                - empty s3 backend bucket
                    - delete-bucket

#### 2.3 main.tf
- configure s3 as backend
- create an s3 bucket
- create a keypair
    - using a public key from GitHub Secret
- AMI lookup
- create VPC
- create a subnet
- create a network interface
- create EC2 instance

### 3. Setup
#### 3.1 GH clone and repo set

You will need to clone my code
```
gh repo clone https://github.com/nhatvo1502/Automate-IaC/tree/main
```
When you clone a reposityory, it added as a remote of yours, under the name origin. What you need to do now (as you're not using the my source anymore) is to rename origin to upstream and add your own origin URL:
```
git remote rename origin upstream
git remote add origin http://github.com/YOU/YOUR_REPO
```
Whenever you want to changes from my branch (which is your upstream)
```
git fetch upstream
```
Whenever you want to commit and push to your repository
```
git add *.*
git commit -m 'YOUR_MESSAGE'
git push
```

#### 3.2 Github Secrets
- Create AWS CLI user with Admin Permission
```
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html
```
- Create GitHub Secrets
```
https://docs.github.com/en/codespaces/managing-codespaces-for-your-organization/managing-development-environment-secrets-for-your-repository-or-organization
```
Login to your Github Account > Go to your project Repository > Settings > Under Secrets and variables > Select Actions > Create an environment "Dev" > Create AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
![image](/images/004/ghsecret1.png)

### 4. Github Actions

![image](/images/004/ghsecret2.png)

### 5. GitHub Action runner results:

#### 5.1 Terraform apply
```
Run terraform apply -auto-approve
data.aws_ami.ubuntu: Reading...
data.aws_ami.ubuntu: Read complete after 0s [id=ami-027a754129abb5386]

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.new_instance will be created
  + resource "aws_instance" "new_instance" {
      + ami                                  = "ami-027a754129abb5386"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core              
         = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_lifecycle                   = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t3.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = "automate-iac-kp"
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + spot_instance_request_id             = (known after apply)
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "automate-iac"
        }
      + tags_all                             = {
          + "Name" = "automate-iac"
        }
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + network_interface {
          + delete_on_termination = false
          + device_index          = 0
          + network_card_index    = 0
          + network_interface_id  = (known after apply)
        }
    }

  # aws_key_pair.new_keypair will be created
  + resource "aws_key_pair" "new_keypair" {
      + arn             = (known after apply)
      + fingerprint     = (known after apply)
      + id              = (known after apply)
      + key_name        = "automate-iac-kp"
      + key_name_prefix = (known after apply)
      + key_pair_id     = (known after apply)
      + key_type        = (known after apply)
      + public_key      = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCJgKe/Ko/3pXldymY2/bwP70xPJzP6NpKS9iQP450sWFtc5VuG3fqaOvGMzhrS3VUseALddYALiWmgf+cgjdvASeW3eIl54y2253QTme4eb1WOZ2/MvcANBuiJFqItL7SR4xlaAuI48yS8lh7L2OIIpzy8PUV+EcPD9KGKsAicFwIDAQAB"
      + tags            = {
          + "Name" = "automate-iac"
        }
      + tags_all        = {
          + "Name" = "automate-iac"
        }
    }

  # aws_network_interface.automate-iac-aws_network_interface will be created
  + resource "aws_network_interface" "automate-iac-aws_network_interface" {
      + arn                       = (known after apply)
      + id                        = (known after apply)
      + interface_type            = (known after apply)
      + ipv4_prefix_count         = (known after apply)
      + ipv4_prefixes             = (known after apply)
      + ipv6_address_count        = (known after apply)
      + ipv6_address_list         = (known after apply)
      + ipv6_address_list_enabled = false
      + ipv6_addresses            = (known after apply)
      + ipv6_prefix_count         = (known after apply)
      + ipv6_prefixes             = (known after apply)
      + mac_address               = (known after apply)
      + outpost_arn               = (known after apply)
      + owner_id                  = (known after apply)
      + private_dns_name          = (known after apply)
      + private_ip                = (known after apply)
      + private_ip_list           = (known after apply)
      + private_ip_list_enabled   = false
      + private_ips               = (known after apply)
      + private_ips_count         = (known after apply)
      + security_groups           = (known after apply)
      + source_dest_check         = true
      + subnet_id                 = (known after apply)
      + tags                      = {
          + "Name" = "automate-iac"
        }
      + tags_all                  = {
          + "Name" = "automate-iac"
        }
    }

  # aws_s3_bucket.new_bucket will be created
  + resource "aws_s3_bucket" "new_bucket" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "automate-iac-bucket-123123123"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)
    }

  # aws_subnet.automate-iac-subnet will be created
  + resource "aws_subnet" "automate-iac-subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "172.16.10.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "automate-iac"
        }
      + tags_all                                       = {
          + "Name" = "automate-iac"
        }
      + vpc_id                                         = (known after apply)
    }

  # aws_vpc.automate-iac-vpc will be created
  + resource "aws_vpc" "automate-iac-vpc" {
      + arn                                  = (known after apply)
      + cidr_block                           = "172.16.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_dns_hostnames                 = (known after apply)
      + enable_dns_support                   = true
      + enable_network_address_usage_metrics = (known after apply)
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Name" = "automate-iac"
        }
      + tags_all                             = {
          + "Name" = "automate-iac"
        }
    }

Plan: 6 to add, 0 to change, 0 to destroy.
aws_key_pair.new_keypair: Creating...
aws_vpc.automate-iac-vpc: Creating...
aws_s3_bucket.new_bucket: Creating...
aws_key_pair.new_keypair: Creation complete after 0s [id=automate-iac-kp]
aws_vpc.automate-iac-vpc: Creation complete after 1s [id=vpc-02606838ac1e89cea]
aws_subnet.automate-iac-subnet: Creating...
aws_subnet.automate-iac-subnet: Creation complete after 1s [id=subnet-0dbfc00f69e201fa7]
aws_network_interface.automate-iac-aws_network_interface: Creating...
aws_network_interface.automate-iac-aws_network_interface: Creation complete after 0s [id=eni-0673ed65d965c194e]
aws_instance.new_instance: Creating...
aws_s3_bucket.new_bucket: Still creating... [10s elapsed]
aws_instance.new_instance: Still creating... [10s elapsed]
aws_instance.new_instance: Creation complete after 13s [id=i-08a28371557209b0b]
aws_s3_bucket.new_bucket: Creation complete after 16s [id=automate-iac-bucket-123123123]

Apply complete! Resources: 6 added, 0 changed, 0 destroyed.

```

#### 5.2 Terraform destroy
```
Run terraform destroy -auto-approve
data.aws_ami.ubuntu: Reading...
aws_vpc.automate-iac-vpc: Refreshing state... [id=vpc-02606838ac1e89cea]
aws_key_pair.new_keypair: Refreshing state... [id=automate-iac-kp]
aws_s3_bucket.new_bucket: Refreshing state... [id=automate-iac-bucket-123123123]
data.aws_ami.ubuntu: Read complete after 0s [id=ami-027a754129abb5386]
aws_subnet.automate-iac-subnet: Refreshing state... [id=subnet-0dbfc00f69e201fa7]
aws_network_interface.automate-iac-aws_network_interface: Refreshing state... [id=eni-0673ed65d965c194e]
aws_instance.new_instance: Refreshing state... [id=i-08a28371557209b0b]

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_instance.new_instance will be destroyed
  - resource "aws_instance" "new_instance" {
      - ami                                  = "ami-027a754129abb5386" -> null
      - arn                                  = "arn:aws:ec2:us-east-1:566027688242:instance/i-08a28371557209b0b" -> null
      - associate_public_ip_address          = false -> null
      - availability_zone                    = "us-east-1a" -> null
      - cpu_core_count                       = 1 -> null
      - cpu_threads_per_core                 = 2 -> null
      - disable_api_stop                     = false -> null
      - disable_api_termination              = false -> null
      - ebs_optimized                        = false -> null
      - get_password_data                    = false -> null
      - hibernation                          = false -> null
      - id                                   = "i-08a28371557209b0b" -> null
      - instance_initiated_shutdown_behavior = "stop" -> null
      - instance_state                       = "running" -> null
      - instance_type                        = "t3.micro" -> null
      - ipv6_address_count                   = 0 -> null
      - ipv6_addresses                       = [] -> null
      - key_name                             = "automate-iac-kp" -> null
      - monitoring                           = false -> null
      - placement_partition_number           = 0 -> null
      - primary_network_interface_id         = "eni-0673ed65d965c194e" -> null
      - private_dns                          = "ip-172-16-10-100.ec2.internal" -> null
      - private_ip                           = "172.16.10.100" -> null
      - secondary_private_ips                = [] -> null
      - security_groups                      = [] -> null
      - source_dest_check                    = true -> null
      - subnet_id                            = "subnet-0dbfc00f69e201fa7" -> null
      - tags                                 = {
          - "Name" = "automate-iac"
        } -> null
      - tags_all                             = {
          - "Name" = "automate-iac"
        } -> null
      - tenancy                              = "default" -> null
      - user_data_replace_on_change          = false -> null
      - vpc_security_group_ids               = [
          - "sg-0ce04508e0520a5f7",
        ] -> null

      - capacity_reservation_specification {
          - capacity_reservation_preference = "open" -> null
        }

      - cpu_options {
          - core_count       = 1 -> null
          - threads_per_core = 2 -> null
        }

      - credit_specification {
          - cpu_credits = "unlimited" -> null
        }

      - enclave_options {
          - enabled = false -> null
        }

      - maintenance_options {
          - auto_recovery = "default" -> null
        }

      - metadata_options {
          - http_endpoint               = "enabled" -> null
          - http_protocol_ipv6          = "disabled" -> null
          - http_put_response_hop_limit = 1 -> null
          - http_tokens                 = "optional" -> null
          - instance_metadata_tags      = "disabled" -> null
        }

      - network_interface {
          - delete_on_termination = false -> null
          - device_index          = 0 -> null
          - network_card_index    = 0 -> null
          - network_interface_id  = "eni-0673ed65d965c194e" -> null
        }

      - private_dns_name_options {
          - enable_resource_name_dns_a_record    = false -> null
          - enable_resource_name_dns_aaaa_record = false -> null
          - hostname_type                        = "ip-name" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - device_name           = "/dev/sda1" -> null
          - encrypted             = false -> null
          - iops                  = 100 -> null
          - tags                  = {} -> null
          - throughput            = 0 -> null
          - volume_id             = "vol-0a8b14289eb677afa" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp2" -> null
        }
    }

  # aws_key_pair.new_keypair will be destroyed
  - resource "aws_key_pair" "new_keypair" {
      - arn         = "arn:aws:ec2:us-east-1:566027688242:key-pair/automate-iac-kp" -> null
      - fingerprint = "6f:9d:e0:9f:e1:05:17:47:8c:e0:65:ca:dd:6b:7b:1e" -> null
      - id          = "automate-iac-kp" -> null
      - key_name    = "automate-iac-kp" -> null
      - key_pair_id = "key-0114ec655b0b831e7" -> null
      - key_type    = "rsa" -> null
      - public_key  = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCJgKe/Ko/3pXldymY2/bwP70xPJzP6NpKS9iQP450sWFtc5VuG3fqaOvGMzhrS3VUseALddYALiWmgf+cgjdvASeW3eIl54y2253QTme4eb1WOZ2/MvcANBuiJFqItL7SR4xlaAuI48yS8lh7L2OIIpzy8PUV+EcPD9KGKsAicFwIDAQAB" -> null
      - tags        = {
          - "Name" = "automate-iac"
        } -> null
      - tags_all    = {
          - "Name" = "automate-iac"
        } -> null
    }

  # aws_network_interface.automate-iac-aws_network_interface will be destroyed
  - resource "aws_network_interface" "automate-iac-aws_network_interface" {
      - arn                       = "arn:aws:ec2:us-east-1:566027688242:network-interface/eni-0673ed65d965c194e" -> null
      - id                        = "eni-0673ed65d965c194e" -> null
      - interface_type            = "interface" -> null
      - ipv4_prefix_count         = 0 -> null
      - ipv4_prefixes             = [] -> null
      - ipv6_address_count        = 0 -> null
      - ipv6_address_list         = [] -> null
      - ipv6_address_list_enabled = false -> null
      - ipv6_addresses            = [] -> null
      - ipv6_prefix_count         = 0 -> null
      - ipv6_prefixes             = [] -> null
      - mac_address               = "02:27:2b:64:3d:c3" -> null
      - owner_id                  = "566027688242" -> null
      - private_ip                = "172.16.10.100" -> null
      - private_ip_list           = [
          - "172.16.10.100",
        ] -> null
      - private_ip_list_enabled   = false -> null
      - private_ips               = [
          - "172.16.10.100",
        ] -> null
      - private_ips_count         = 0 -> null
      - security_groups           = [
          - "sg-0ce04508e0520a5f7",
        ] -> null
      - source_dest_check         = true -> null
      - subnet_id                 = "subnet-0dbfc00f69e201fa7" -> null
      - tags                      = {
          - "Name" = "automate-iac"
        } -> null
      - tags_all                  = {
          - "Name" = "automate-iac"
        } -> null

      - attachment {
          - attachment_id = "eni-attach-05750a0a2ee7869c5" -> null
          - device_index  = 0 -> null
          - instance      = "i-08a28371557209b0b" -> null
        }
    }

  # aws_s3_bucket.new_bucket will be destroyed
  - resource "aws_s3_bucket" "new_bucket" {
      - arn                         = "arn:aws:s3:::automate-iac-bucket-123123123" -> null
      - bucket                      = "automate-iac-bucket-123123123" -> null
      - bucket_domain_name          = "automate-iac-bucket-123123123.s3.amazonaws.com" -> null
      - bucket_regional_domain_name = "automate-iac-bucket-123123123.s3.us-east-1.amazonaws.com" -> null
      - force_destroy               = false -> null
      - hosted_zone_id              = "Z3AQBSTGFYJSTF" -> null
      - id                          = "automate-iac-bucket-123123123" -> null
      - object_lock_enabled         = false -> null
      - region                      = "us-east-1" -> null
      - request_payer               = "BucketOwner" -> null
      - tags                        = {} -> null
      - tags_all                    = {} -> null

      - grant {
          - id          = "479f57b44641303e6c9a544836dc9b6e50e73a3e558bcb6a2a08f13a3ec6b547" -> null
          - permissions = [
              - "FULL_CONTROL",
            ] -> null
          - type        = "CanonicalUser" -> null
        }

      - server_side_encryption_configuration {
          - rule {
              - bucket_key_enabled = false -> null

              - apply_server_side_encryption_by_default {
                  - sse_algorithm = "AES256" -> null
                }
            }
        }

      - versioning {
          - enabled    = false -> null
          - mfa_delete = false -> null
        }
    }

  # aws_subnet.automate-iac-subnet will be destroyed
  - resource "aws_subnet" "automate-iac-subnet" {
      - arn                                            = "arn:aws:ec2:us-east-1:566027688242:subnet/subnet-0dbfc00f69e201fa7" -> null
      - assign_ipv6_address_on_creation                = false -> null
      - availability_zone                              = "us-east-1a" -> null
      - availability_zone_id                           = "use1-az1" -> null
      - cidr_block                                     = "172.16.10.0/24" -> null
      - enable_dns64                                   = false -> null
      - enable_lni_at_device_index                     = 0 -> null
      - enable_resource_name_dns_a_record_on_launch    = false -> null
      - enable_resource_name_dns_aaaa_record_on_launch = false -> null
      - id                                             = "subnet-0dbfc00f69e201fa7" -> null
      - ipv6_native                                    = false -> null
      - map_customer_owned_ip_on_launch                = false -> null
      - map_public_ip_on_launch                        = false -> null
      - owner_id                                       = "566027688242" -> null
      - private_dns_hostname_type_on_launch            = "ip-name" -> null
      - tags                                           = {
          - "Name" = "automate-iac"
        } -> null
      - tags_all                                       = {
          - "Name" = "automate-iac"
        } -> null
      - vpc_id                                         = "vpc-02606838ac1e89cea" -> null
    }

  # aws_vpc.automate-iac-vpc will be destroyed
  - resource "aws_vpc" "automate-iac-vpc" {
      - arn                                  = "arn:aws:ec2:us-east-1:566027688242:vpc/vpc-02606838ac1e89cea" -> null
      - assign_generated_ipv6_cidr_block     = false -> null
      - cidr_block                           = "172.16.0.0/16" -> null
      - default_network_acl_id               = "acl-063165fef63bf9e2c" -> null
      - default_route_table_id               = "rtb-087a43ef011a55a02" -> null
      - default_security_group_id            = "sg-0ce04508e0520a5f7" -> null
      - dhcp_options_id                      = "dopt-6cefc816" -> null
      - enable_dns_hostnames                 = false -> null
      - enable_dns_support                   = true -> null
      - enable_network_address_usage_metrics = false -> null
      - id                                   = "vpc-02606838ac1e89cea" -> null
      - instance_tenancy                     = "default" -> null
      - ipv6_netmask_length                  = 0 -> null
      - main_route_table_id                  = "rtb-087a43ef011a55a02" -> null
      - owner_id                             = "566027688242" -> null
      - tags                                 = {
          - "Name" = "automate-iac"
        } -> null
      - tags_all                             = {
          - "Name" = "automate-iac"
        } -> null
    }

Plan: 0 to add, 0 to change, 6 to destroy.
aws_key_pair.new_keypair: Destroying... [id=automate-iac-kp]
aws_s3_bucket.new_bucket: Destroying... [id=automate-iac-bucket-123123123]
aws_instance.new_instance: Destroying... [id=i-08a28371557209b0b]
aws_key_pair.new_keypair: Destruction complete after 0s
aws_s3_bucket.new_bucket: Destruction complete after 0s
aws_instance.new_instance: Still destroying... [id=i-08a28371557209b0b, 10s elapsed]
aws_instance.new_instance: Still destroying... [id=i-08a28371557209b0b, 20s elapsed]
aws_instance.new_instance: Still destroying... [id=i-08a28371557209b0b, 30s elapsed]
aws_instance.new_instance: Still destroying... [id=i-08a28371557209b0b, 40s elapsed]
aws_instance.new_instance: Still destroying... [id=i-08a28371557209b0b, 50s elapsed]
aws_instance.new_instance: Still destroying... [id=i-08a28371557209b0b, 1m0s elapsed]
aws_instance.new_instance: Still destroying... [id=i-08a28371557209b0b, 1m10s elapsed]
aws_instance.new_instance: Destruction complete after 1m10s
aws_network_interface.automate-iac-aws_network_interface: Destroying... [id=eni-0673ed65d965c194e]
aws_network_interface.automate-iac-aws_network_interface: Destruction complete after 1s
aws_subnet.automate-iac-subnet: Destroying... [id=subnet-0dbfc00f69e201fa7]
aws_subnet.automate-iac-subnet: Destruction complete after 0s
aws_vpc.automate-iac-vpc: Destroying... [id=vpc-02606838ac1e89cea]
aws_vpc.automate-iac-vpc: Destruction complete after 0s

Destroy complete! Resources: 6 destroyed.
```