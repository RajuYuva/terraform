# Terraform Notes

This document covers various Terraform concepts, including using count and for_each, working with variables and locals, merging maps, data sources, dynamic blocks, and modules.

---

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Using count.index and Variables](#2-using-countindex-and-variables)
- [3. Merging Maps](#3-merging-maps)
- [4. Locals vs. Variables](#4-locals-vs-variables)
- [5. Data Sources](#5-data-sources)
- [6. Using for_each](#6-using-foreach)
- [7. Dynamic Blocks](#7-dynamic-blocks)
- [8. Provisioners: local-exec and remote-exec](#8-provisioners-localexec-and-remote-exec)
- [9. Modules and Best Practices](#9-modules-and-best-practices)
- [10. Conclusion](#10-conclusion)

---

## 1. Introduction

Terraform is an infrastructure-as-code tool that enables you to define and manage your cloud resources efficiently. This document details key features like looping, merging maps, using locals, data sources, and modules—helping you build reusable and maintainable Terraform configurations.

---

## 2. Using count.index and Variables

### Example: Using count.index with a List Variable

Define a list variable for instance names:

```hcl
variable "instance_name" {
  type    = list(string)
  default = ["db", "backend", "frontend"]
}
```

Use `count` to create multiple resources based on the length of the list:

```hcl
resource "aws_instance" "expense" {
  count         = length(var.instance_name)
  instance_name = var.instance_name[count.index]
  # ... other resource configurations ...
}
```

*Note: `count.index` provides the current index within the resource block iteration.*

---

## 3. Merging Maps

Terraform provides a `merge` function to combine multiple maps.  
For example, given two maps:

- Map 1:  
  ```hcl
  { a = "b", b = "c", c = "d" }
  ```
- Map 2:  
  ```hcl
  { x = "z", e = "f", a = "e" }
  ```

Using the merge command:

```hcl
locals {
  merged_map = merge(
    { a = "b", b = "c", c = "d" },
    { x = "z", e = "f", a = "e" }
  )
}
```

The resulting map would be:

```hcl
{
  a = "e"
  b = "c"
  c = "d"
  x = "z"
  e = "f"
}
```

*Note: Values in later maps override those in earlier maps.*

---

## 4. Locals vs. Variables

- **Variables:**  
  - Defined in `variables.tf`
  - Hold static key-value pairs
  - Cannot reference dynamic expressions inside variables
  
- **Locals:**  
  - Defined in `locals {}` blocks
  - Can hold computed expressions and be used throughout your configuration
  - You can reference variables inside locals

Example:

```hcl
variable "environment" {
  type    = string
  default = "production"
}

locals {
  app_name = "expense-app-${var.environment}"
}
```

*Note: Use locals for computed values or to simplify complex expressions.*

---

## 5. Data Sources

Data sources allow you to query information from providers (like retrieving an AMI ID).  
For example, if another team created infrastructure manually, you can query that using a data source:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
```

*Note: Data sources make it possible to reference existing resources outside of Terraform’s management.*

---

## 6. Using for_each

When you need to create resources from a map, use `for_each`:

```hcl
variable "instance_type_map" {
  type = map(string)
  default = {
    db       = "t3.small"
    frontend = "t2.micro"
    backend  = "t2.micro"
  }
}

resource "aws_instance" "expense" {
  for_each      = var.instance_type_map
  instance_name = each.key
  instance_type = each.value
  # ... other resource configurations ...
}
```

*Note: `for_each` iterates over each key-value pair in the map, simplifying resource creation for multiple similar objects.*

---

## 7. Dynamic Blocks

Dynamic blocks are used to iterate over repeated configuration blocks, such as multiple ingress rules, without duplicating code.

Example (simplified for an ingress block):

```hcl
resource "aws_security_group" "example" {
  name = "example-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

*Note: Replace `var.ingress_rules` with your variable containing a list or map of ingress configurations.*

---

## 8. Provisioners: local-exec and remote-exec

- **local-exec:** Executes commands on your local machine.
- **remote-exec:** Executes commands on a remote resource created by Terraform.

Example using local-exec:

```hcl
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo 'This runs on the local machine'"
  }
}
```

*Note: Use provisioners sparingly, as they can reduce the portability of your configuration.*

---

## 9. Modules and Best Practices

### Advantages of Modules

- **Code Reuse:** Write once and use across different environments.
- **Best Practices Enforcement:** Modules can enforce standards and best practices.
- **Maintainability:** Easier to update and manage.
- **Restrictions:** Modules can include validations to ensure compliance with company guidelines.

### Module Usage

There are two main types of modules:
1. **Custom Module Development:** Create your own modules for organization-specific needs.
2. **Open Source Modules:** Use community-maintained modules from the Terraform Registry.

*Example of calling a module:*

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "x.y.z"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
  # ... additional configuration ...
}
```

*Note: Separate module code from your main configuration to promote reusability.*

---

## 10. Conclusion

This document has outlined key Terraform concepts including:
- Using `count.index` with list variables.
- Merging maps and the differences between locals and variables.
- Querying existing resources with data sources.
- Iterating with `for_each` and dynamic blocks.
- Provisioners for local and remote execution.
- The benefits and best practices of module usage.
