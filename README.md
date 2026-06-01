# # 🏗️ Terraform-AWS-lb

[![OpsStation](https://img.shields.io/badge/Made%20by-OpsStation-blue?style=flat-square&logo=terraform)](https://www.opsstation.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Terraform](https://img.shields.io/badge/Terraform-1.13%2B-purple.svg?logo=terraform)](#)
[![CI](https://github.com/OpsStation/terraform-aws-ec2/actions/workflows/ci.yml/badge.svg)](https://github.com/OpsStation/terraform-aws-ec2/actions/workflows/ci.yml)

> 🌩️ **A production-grade, reusable AWS Ec2 module by [OpsStation](https://www.opsstation.com)**
> Designed for reliability, performance, and security — following AWS networking best practices.
---

## 🏢 About OpsStation

**OpsStation** delivers **Cloud & DevOps excellence** for modern teams:
- 🚀 **Infrastructure Automation** with Terraform, Ansible & Kubernetes
- 💰 **Cost Optimization** via scaling & right-sizing
- 🛡️ **Security & Compliance** baked into CI/CD pipelines
- ⚙️ **Fully Managed Operations** across AWS, Azure, and GCP

> 💡 Need enterprise-grade DevOps automation?
> 👉 Visit [**www.opsstation.com**](https://www.opsstation.com) or email **hello@opsstation.com**

---
## 🌟 Features

- ✅ Creates and manages **AWS Load Balancers** (Application, Network, or Gateway LB)
- ✅ Supports both **internal** and **internet-facing** load balancer configurations
- ✅ Automatically configures **listeners**, **target groups**, and **listener rules**
- ✅ Supports **HTTP**, **HTTPS**, and **TCP/UDP** protocols with custom port mappings
- ✅ Integrates seamlessly with **Auto Scaling Groups**, **EC2 Instances**, and **ECS Services**
- ✅ Optional **SSL/TLS certificate** integration via **AWS ACM** for secure traffic
- ✅ Supports **cross-zone load balancing**, **access logs**, and **connection draining**
- ✅ Enables tagging and naming conventions through the **Labels module**
- ✅ Follows AWS best practices for **high availability** and **fault tolerance**
- ✅ Fully compatible with other **OpsStation Terraform modules**
---

# Example : alb
```hcl
module "alb" {
  source                     = "git::https://github.com/opsstation/terraform-aws-lb.git?ref=v1.0.0"

  name                       = local.name
  environment = local.environment
  enable                     = true
  internal                   = true
  load_balancer_type         = "application"
  instance_count             = 2
  subnets                    = module.subnet.public_subnet_id
  target_id                  = module.ec2.instance_id
  vpc_id                     = module.vpc.id
  allowed_ip                 = [module.vpc.vpc_cidr_block]
  allowed_ports              = [3306]
  enable_deletion_protection = false
  with_target_group          = true
  https_enabled              = false
  http_enabled               = true
  https_port                 = 443
  listener_type              = "forward"
  target_group_port          = 80

  http_tcp_listeners = [
    {
      port               = 80
      protocol           = "TCP"
      target_group_index = 0
    },
    {
      port               = 81
      protocol           = "TCP"
      target_group_index = 0
    },
  ]
  https_listeners = [
    {
      port               = 443
      protocol           = "TLS"
      target_group_index = 0
      certificate_arn    = ""
    },
    {
      port               = 84
      protocol           = "TLS"
      target_group_index = 0
      certificate_arn    = ""
    },
  ]

  target_groups = [
    {
      backend_protocol     = "HTTP"
      backend_port         = 80
      target_type          = "instance"
      deregistration_delay = 300
      health_check = {
        enabled             = true
        interval            = 30
        path                = "/"
        port                = "traffic-port"
        healthy_threshold   = 3
        unhealthy_threshold = 3
        timeout             = 10
        protocol            = "HTTP"
        matcher             = "200-399"
      }
    }
  ]

}
```

# Example : clb
```hcl
module "clb" {
  source             = "git::https://github.com/opsstation/terraform-aws-lb.git?ref=v1.0.0"

  name               = "app"
  load_balancer_type = "classic"
  clb_enable         = true
  internal           = true
  vpc_id             = module.vpc.id
  target_id          = module.ec2.instance_id
  subnets            = module.public_subnets.public_subnet_id
  with_target_group  = true
  listeners = [
    {
      lb_port            = 22000
      lb_protocol        = "TCP"
      instance_port      = 22000
      instance_protocol  = "TCP"
      ssl_certificate_id = null
    },
    {
      lb_port            = 4444
      lb_protocol        = "TCP"
      instance_port      = 4444
      instance_protocol  = "TCP"
      ssl_certificate_id = null
    }
  ]
  health_check_target              = "TCP:4444"
  health_check_timeout             = 10
  health_check_interval            = 30
  health_check_unhealthy_threshold = 5
  health_check_healthy_threshold   = 5
}
```

# Example : nlb
```hcl
module "nlb" {
  source                     = "git::https://github.com/opsstation/terraform-aws-lb.git?ref=v1.0.0"

  name                       = "app"
  enable                     = true
  internal                   = false
  load_balancer_type         = "network"
  instance_count             = 1
  subnets                    = module.public_subnets.public_subnet_id
  target_id                  = module.ec2.instance_id
  vpc_id                     = module.vpc.id
  enable_deletion_protection = false
  with_target_group          = true
  http_tcp_listeners = [
    {
      port               = 80
      protocol           = "TCP"
      target_group_index = 0
    },
    {
      port               = 81
      protocol           = "TCP"
      target_group_index = 0
    },
  ]
  target_groups = [
    {
      backend_protocol = "TCP"
      backend_port     = 80
      target_type      = "instance"
    },
    {
      backend_protocol = "TCP"
      backend_port     = 81
      target_type      = "instance"
    },
  ]

  https_listeners = [
    {
      port               = 443
      protocol           = "TLS"
      target_group_index = 1
      certificate_arn    = ""
    },
    {
      port               = 84
      protocol           = "TLS"
      target_group_index = 1
      certificate_arn    = ""
    },
  ]
}
```

### 🔐 Outputs (AWS Load Balancer Module)

| Name                     | Description                                                                 |
|---------------------------|------------------------------------------------------------------------------|
| `id`                      | The unique identifier (ID) of the created **Load Balancer**.                |
| `arn`                     | The ARN (Amazon Resource Name) of the created **Load Balancer**.            |
| `name`                    | The name of the created **Load Balancer**.                                  |
| `dns_name`                | The **DNS name** of the Load Balancer to use for application access.        |
| `zone_id`                 | The **Canonical Hosted Zone ID** of the Load Balancer.                      |
| `vpc_id`                  | The **VPC ID** in which the Load Balancer is created.                       |
| `security_groups`         | A list of **security group IDs** associated with the Load Balancer (if applicable). |
| `subnets`                 | A list of **subnet IDs** used by the Load Balancer.                         |
| `load_balancer_type`      | The type of the Load Balancer (**application**, **network**, or **gateway**).|
| `arn_suffix`              | The **ARN suffix** used in CloudWatch metrics and IAM policies.             |
| `listener_arns`           | A list of **listener ARNs** created for the Load Balancer.                  |
| `target_group_arns`       | A list of **target group ARNs** associated with the Load Balancer.          |
| `access_logs_bucket`      | The name of the **S3 bucket** used for access logging (if enabled).         |
| `tags`                    | A mapping of **tags** assigned to the Load Balancer resources.              |

### ☁️ Tag Normalization Rules (AWS)

| Cloud | Case      | Allowed Characters | Example                            |
|--------|-----------|------------------|------------------------------------|
| **AWS** | TitleCase | Any              | `Name`, `Environment`, `CostCenter` |

---

### 💙 Maintained by [OpsStation](https://www.opsstation.com)
> OpsStation — Simplifying Cloud, Securing Scale.
