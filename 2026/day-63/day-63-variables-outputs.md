Task 1: Extract Variables

Take your Day 62 infrastructure config and refactor it:

1) Create a `variables.tf` file with input variables for:

- `region` (string, default: your preferred region)

        variable "region" {
          description = "This variable holds region"
          default     = "us-west-2"
          type        = string
        }

- `vpc_cidr` (string, default: `"10.0.0.0/16"`)

        variable "vpc_cidr" {
          description = "This variable holds vpc cidr"
          default     = "10.0.0.0/16"
          type        = string
        }

- `subnet_cidr` (string, default: `"10.0.1.0/24"`)

        variable "subnet_cidr" {
          description = "This variable holds subnet cidr"
          default     = "10.0.1.0/24"
          type        = string
        }

- `instance_type` (string, default: `"t2.micro"`)

        variable "instance_type" {
          description = "This variable holds ec2 instance type"
          default     = "t2.micro"
          type        = string
        }

- `project_name` (string, no default -- force the user to provide it)

        variable "project_name" {
          description = "This variable holds project name"
          type        = string
        }

- `environment` (string, default: `"dev"`)

        variable "environment" {
          description = "This variable holds environment"
          default     = "dev"
          type        = string
        }

- `allowed_ports` (list of numbers, default: `[22, 80, 443]`)

        variable "allowed_ports" {
          description = "This variable holds allowed ports"
          default     = [22, 80, 443]
          type        = list(number)
        }

- `extra_tags` (map of strings, default: `{}`)

        variable "extra_tags" {
          description = "This variable holds vpc cidr"
          default     = {}
          type        = map(string)
        }

2) Replace every hardcoded value in `main.tf` with `var.<name>` references

```hcl
data "aws_ami" "amazon_linux_2" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "state"
    values = ["available"]
  }
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = merge(
    {
      Name        = "${var.project_name}-VPC"
      Environment = var.environment
    },
    var.extra_tags
  )
}

resource "aws_subnet" "main" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.subnet_cidr
  map_public_ip_on_launch = true

  tags = merge(
    {
      Name        = "${var.project_name}-Public-Subnet"
      Environment = var.environment
    },
    var.extra_tags
  )
}

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    {
      Name        = "${var.project_name}-IGW"
      Environment = var.environment
    },
    var.extra_tags
  )
}

resource "aws_route_table" "example" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  tags = merge(
    {
      Name        = "${var.project_name}-RouteTable"
      Environment = var.environment
    },
    var.extra_tags
  )
}

resource "aws_route_table_association" "example" {
  subnet_id      = aws_subnet.main.id
  route_table_id = aws_route_table.example.id
}

resource "aws_security_group" "main" {
  name        = "${var.project_name}-SG"
  description = "Allow SSH and HTTP traffic"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      description = "Allow port ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    {
      Name        = "${var.project_name}-SG"
      Environment = var.environment
    },
    var.extra_tags
  )
}

resource "aws_instance" "main" {
  ami                         = data.aws_ami.amazon_linux_2.id
  instance_type               = var.instance_type
  subnet_id                   = aws_subnet.main.id
  vpc_security_group_ids      = [aws_security_group.main.id]
  associate_public_ip_address = true

  lifecycle {
    create_before_destroy = true
  }

  tags = merge(
    {
      Name        = "${var.project_name}-Server"
      Environment = var.environment
    },
    var.extra_tags
  )
}

resource "aws_s3_bucket" "app_logs" {
  bucket     = "${var.project_name}-app-logs-2026"
  depends_on = [aws_instance.main]

  tags = merge(
    {
      Name        = "${var.project_name}-App-Logs"
      Environment = var.environment
    },
    var.extra_tags
  )
}
```

3) Run `terraform plan` -- it should prompt you for `project_name` since it has no default

- terraform plan    

        var.project_name
          This variable holds project name
        
          Enter a value: TerraWeek

Document: What are the five variable types in Terraform? (`string`, `number`, `bool`, `list`, `map`)

| Type   | Example                           |
|--------|-----------------------------------|
| string | us-west-2                         |
| number | 2                                 |
| bool   | true                              |
| list   | [22, 80, 443]                     |
| map    | { Owner = Chetan, Team = DevOps } |

---

Task 2: Variable Files and Precedence

1) Create `terraform.tfvars`:
        
        project_name = "terraweek"
        environment  = "dev"
        instance_type = "t2.micro"

2) Create `prod.tfvars`:

        project_name = "terraweek"
        environment  = "prod"
        instance_type = "t3.small"
        vpc_cidr     = "10.1.0.0/16"
        subnet_cidr  = "10.1.1.0/24"

3) Apply with the default file:

        terraform plan                              # Uses terraform.tfvars automatically

4) Apply with the prod file:
        
        terraform plan -var-file="prod.tfvars"      # Uses prod.tfvars

5) Override with CLI:
        
        terraform plan -var="instance_type=t2.nano"  # CLI overrides everything

6) Set an environment variable:
        
        export TF_VAR_environment="staging"
        terraform plan                              # env var overrides default but not tfvars

Document: Write the variable precedence order from lowest to highest priority.

        1. Variable defaults
                ↓
        2. Environment variables (TF_VAR_*)
                ↓
        3. terraform.tfvars / *.auto.tfvars
                ↓
        4. -var-file
                ↓
        5. -var

*Default value → Environment variable → tfvars files → -var-file → CLI -var*

---

Task 3: Add Outputs

Create an `outputs.tf` file with outputs for:

1) `vpc_id` -- the VPC ID

        output "vpc_id" {
          value = aws_vpc.main.id
        }

2) `subnet_id` -- the public subnet ID

        output "subnet_id" {
          value = aws_subnet.main.id
        }

3) `instance_id` -- the EC2 instance ID

        output "instance_id" {
          value = aws_instance.main.id
        }

4) `instance_public_ip` -- the public IP of the EC2 instance

        output "instance_public_ip" {
          value = aws_instance.main.public_ip
        }

5) `instance_public_dns` -- the public DNS name

        output "instance_public_dns" {
          value = aws_instance.main.public_dns
        }

6) `security_group_id` -- the security group ID

        output "security_group_id" {
          value = aws_security_group.main.id
        }

Apply your config and verify the outputs are printed at the end:

        terraform apply
        
        # After apply, you can also run:
        terraform output                          # Show all outputs
        terraform output instance_public_ip       # Show a specific output
        terraform output -json                    # JSON format for scripting

Verify: Does `terraform output instance_public_ip` return the correct IP?
        
        should return the current public IPv4 address of your EC2 instance.

---

Task 4: Use Data Sources

Stop hardcoding the AMI ID. Use a data source to fetch it dynamically.

1) Add a `data "aws_ami"` block that:

- Filters for Amazon Linux 2 images

- Filters for `hvm` virtualization and `gp2` root device

        filter {
            name   = "virtualization-type"
            values = ["hvm"]
          }
        
        filter {
            name   = "name"
            values = ["amzn2-ami-hvm-*-x86_64-gp2"]
          }

- Uses `owners = ["amazon"]`
        
        owners      = ["amazon"]

- Sets `most_recent = true`
        
        most_recent = true

2) Replace the hardcoded AMI in your `aws_instance` with `data.aws_ami.amazon_linux.id`
        
        ami = data.aws_ami.amazon_linux.id

3) Add a `data "aws_availability_zones"` block to fetch available AZs in your region

        data "aws_availability_zones" "available" {
          state = "available"
        }

4) Use the first AZ in your subnet: `data.aws_availability_zones.available.names[0]`

        resource "aws_subnet" "main" {
          vpc_id                  = aws_vpc.main.id
          cidr_block              = var.subnet_cidr
          availability_zone = data.aws_availability_zones.available.names[0]
          map_public_ip_on_launch = true
        }

Apply and verify -- your config now works in any region without changing the AMI.

Document: What is the difference between a `resource` and a `data` source?

| resource                        | data source                          |
|---------------------------------|--------------------------------------|
| Creates/manages infrastructure  | Reads existing information           |
| Terraform manages the object    | Terraform does not create the object |
| Can create, update, and destroy | Primarily retrieves information      |
| Example: aws_instance           | Example: aws_ami                     |
| Example: aws_vpc                | Example: aws_availability_zones      |

---

Task 5: Use Locals for Dynamic Values

1) Add a `locals` block:

        locals {
          name_prefix = "${var.project_name}-${var.environment}"
          common_tags = {
            Project     = var.project_name
            Environment = var.environment
            ManagedBy   = "Terraform"
          }
        }

2) Replace all Name tags with `local.name_prefix`:

- VPC: `"${local.name_prefix}-vpc"`

        tags = merge(
          local.common_tags,
          var.extra_tags,
          {
            Name = "${local.name_prefix}-vpc"
          }
        )

- Subnet: `"${local.name_prefix}-subnet"`

        tags = merge(
          local.common_tags,
          var.extra_tags,
          {
            Name = "${local.name_prefix}-subnet"
          }
        )

- Instance: `"${local.name_prefix}-server"`

        tags = merge(
          local.common_tags,
          var.extra_tags,
          {
            Name = "${local.name_prefix}-server"
          }
        )

3) Merge common tags with resource-specific tags:

        tags = merge(local.common_tags, {
          Name = "${local.name_prefix}-server"
        })

Apply and check the tags in the AWS console -- every resource should have consistent tagging.

        terraweek-dev-vpc
        terraweek-dev-subnet
        terraweek-dev-igw
        terraweek-dev-route-table
        terraweek-dev-sg
        terraweek-dev-server
        terraweek-dev-app-logs

---

Task 6: Built-in Functions and Conditional Expressions

Practice these in `terraform console`:

        terraform console

1) String functions:

- `upper("terraweek")` -> `"TERRAWEEK"`

        upper("terraweek")
        "TERRAWEEK"

- `join("-", ["terra", "week", "2026"])` -> `"terra-week-2026"`

        join("-", ["terra", "week", "2026"])
        "terra-week-2026"

- `format("arn:aws:s3:::%s", "my-bucket")`

        format("arn:aws:s3:::%s", "my-bucket")
        "arn:aws:s3:::my-bucket"

2) Collection functions:

- `length(["a", "b", "c"])` -> `3`

        length(["a", "b", "c"])
        3

- `lookup({dev = "t2.micro", prod = "t3.small"}, "dev")` -> `"t2.micro"`

        lookup({dev = "t2.micro", prod = "t3.small"}, "dev")
        "t2.micro"

- `toset(["a", "b", "a"])` -> removes duplicates

        toset([
          "a",
          "b",
        ])

3) Networking function:

- `cidrsubnet("10.0.0.0/16", 8, 1)` -> `"10.0.1.0/24"`
        
        cidrsubnet("10.0.0.0/16", 8, 1)
        "10.0.1.0/24"

4) Conditional expression -- add this to your config:

- instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
        
        resource "aws_instance" "main" {
          ami                         = data.aws_ami.amazon_linux.id
          instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
          subnet_id                   = aws_subnet.main.id
          vpc_security_group_ids      = [aws_security_group.main.id]
          associate_public_ip_address = true
        
          lifecycle {
            create_before_destroy = true
          }
        
          tags = merge(
            local.common_tags,
            var.extra_tags,
            {
              Name = "${local.name_prefix}-server"
            }
          )
        }


Apply with `environment = "prod"` and verify the instance type changes.

Document: Pick five functions you find most useful and explain what each does.

1. length()

        Returns the number of elements in a collection or characters in a string.

2. lookup()

        Retrieves a value from a map using a key.

3. merge()

        Combines multiple maps into one map.

4. cidrsubnet()

        Calculates a subnet CIDR from a larger network CIDR.

5. upper()

        Converts a string to uppercase.

---

`variables.tf`
```hcl
variable "region" {
  description = "This variable holds region"
  default     = "us-west-2"
  type        = string
}

variable "vpc_cidr" {
  description = "This variable holds vpc cidr"
  default     = "10.0.0.0/16"
  type        = string
}

variable "subnet_cidr" {
  description = "This variable holds subnet cidr"
  default     = "10.0.1.0/24"
  type        = string
}

variable "instance_type" {
  description = "This variable holds ec2 instance type"
  default     = "t2.micro"
  type        = string
}

variable "project_name" {
  description = "This variable holds project name"
  type        = string
}

variable "environment" {
  description = "This variable holds environment"
  default     = "dev"
  type        = string
}

variable "allowed_ports" {
  description = "This variable holds allowed ports"
  default     = [22, 80, 443]
  type        = list(number)
}

variable "extra_tags" {
  description = "This variable holds vpc cidr"
  default     = {}
  type        = map(string)
}
```

`terraform.tfvars`
```hcl
project_name  = "terraweek"
environment   = "dev"
instance_type = "t2.micro"
```

`prod.tfvars`
```hcl
project_name  = "terraweek"
environment   = "prod"
instance_type = "t3.small"
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
```

`outputs.tf`
```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}

output "subnet_id" {
  value = aws_subnet.main.id
}

output "instance_id" {
  value = aws_instance.main.id
}

output "instance_public_ip" {
  value = aws_instance.main.public_ip
}

output "instance_public_dns" {
  value = aws_instance.main.public_dns
}

output "security_group_id" {
  value = aws_security_group.main.id
}
```

`locals.tf`
```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"

  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

`main.tf`

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "state"
    values = ["available"]
  }
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = merge(
    local.common_tags,
    var.extra_tags,
    {
      Name = "${local.name_prefix}-vpc"
    }
  )
}

resource "aws_subnet" "main" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.subnet_cidr
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = true

  tags = merge(
    local.common_tags,
    var.extra_tags,
    {
      Name = "${local.name_prefix}-subnet"
    }
  )
}

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    local.common_tags,
    var.extra_tags,
    {
      Name = "${local.name_prefix}-igw"
    }
  )
}

resource "aws_route_table" "example" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  tags = merge(
    local.common_tags,
    var.extra_tags,
    {
      Name = "${local.name_prefix}-route-table"
    }
  )
}

resource "aws_route_table_association" "example" {
  subnet_id      = aws_subnet.main.id
  route_table_id = aws_route_table.example.id
}

resource "aws_security_group" "main" {
  name        = "${local.name_prefix}-sg"
  description = "Allow SSH and HTTP traffic"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      description = "Allow port ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    local.common_tags,
    var.extra_tags,
    {
      Name = "${local.name_prefix}-sg"
    }
  )
}

data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_instance" "main" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
  subnet_id                   = aws_subnet.main.id
  vpc_security_group_ids      = [aws_security_group.main.id]
  associate_public_ip_address = true

  lifecycle {
    create_before_destroy = true
  }

  tags = merge(
    local.common_tags,
    var.extra_tags,
    {
      Name = "${local.name_prefix}-server"
    }
  )
}

resource "aws_s3_bucket" "app_logs" {
  bucket     = "${var.project_name}-app-logs-2026"
  depends_on = [aws_instance.main]

  tags = merge(
    local.common_tags,
    var.extra_tags,
    {
      Name = "${local.name_prefix}-app-logs"
    }
  )
}
```
