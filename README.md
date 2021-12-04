# terraform 101

## Intro articles
## Modules
### [On writing terraform modules]( https://learn.hashicorp.com/tutorials/terraform/pattern-module-creation?in=terraform/modules )
### [on using modules from registry](https://learn.hashicorp.com/tutorials/terraform/module-use)
<br>
### [on input variables](https://www.terraform.io/docs/language/values/variables.html)

### [On validating input module variables in variables.tf](https://medium.com/codex/terraform-variable-validation-b9b3e7eddd79)

### [On using dynamic in terraform script or module](https://www.terraform.io/docs/language/expressions/dynamic-blocks.html)
### [more on dynamic](https://lgallardo.com/2019/06/14/dynamic-blocks-in-terraform-0.12.x/#:~:text=Terraform%200.12.x%20proposes%20dynamic%20blocks%20to%20solve%20this,function%20doesn%E2%80%99t%20find%20the%20index%20on%20the%20map.) 

### [On using terragrunt with modules to make them DRY](https://www.youtube.com/watch?v=LVgP63BkhKQ)

### [On testing terraform modules with terratest go](https://terratest.gruntwork.io/docs/getting-started/introduction/)

### [on misleading error message when fmt or init, missing '='](https://github.com/hashicorp/terraform/issues/25039)

## testing and documenting

### [Auto-generating docs from TF files with terraform-docs](https://github.com/terraform-docs/terraform-docs)

### [check your Cloud IaaC including terraform modules/scripts for errors/mistakes with checkov](https://www.checkov.io/)

### [Visualize your tf IaaC with rover](tba)

### DataSources and obtaining latest AMI ID 
```
provider "aws" {}


data "aws_availability_zones" "working" {}
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}
data "aws_vpcs" "my_vpcs" {}


data "aws_vpc" "prod_vpc" {
  tags = {
    Name = "prod"
  }
}


resource "aws_subnet" "prod_subnet_1" {
  vpc_id            = data.aws_vpc.prod_vpc.id
  availability_zone = data.aws_availability_zones.working.names[0]
  cidr_block        = "10.10.1.0/24"
  tags = {
    Name    = "Subnet-1 in ${data.aws_availability_zones.working.names[0]}"
    Account = "Subnet in Account ${data.aws_caller_identity.current.account_id}"
    Region  = data.aws_region.current.description
  }
}

resource "aws_subnet" "prod_subnet_2" {
  vpc_id            = data.aws_vpc.prod_vpc.id
  availability_zone = data.aws_availability_zones.working.names[1]
  cidr_block        = "10.10.2.0/24"
  tags = {
    Name    = "Subnet-2 in ${data.aws_availability_zones.working.names[1]}"
    Account = "Subnet in Account ${data.aws_caller_identity.current.account_id}"
    Region  = data.aws_region.current.description
  }
}



output "prod_vpc_id" {
  value = data.aws_vpc.prod_vpc.id
}

output "prod_vpc_cidr" {
  value = data.aws_vpc.prod_vpc.cidr_block
}

output "aws_vpcs" {
  value = data.aws_vpcs.my_vpcs.ids
}


output "data_aws_availability_zones" {
  value = data.aws_availability_zones.working.names
}


output "data_aws_caller_identity" {
  value = data.aws_caller_identity.current.account_id
}

output "data_aws_region_name" {
  value = data.aws_region.current.name
}

output "data_aws_region_description" {
  value = data.aws_region.current.description
}
```

### Obtaining the latest Ubuntu 18.04 server AMI-id with Datasources

```

provider "aws" {
  region = "ap-southeast-2"
}

data "aws_ami" "latest_ubuntu" {
  owners      = ["099720109477"]
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
  }
}


data "aws_ami" "latest_amazon_linux" {
  owners      = ["amazon"]
  most_recent = true
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}


data "aws_ami" "latest_windows_2016" {
  owners      = ["amazon"]
  most_recent = true
  filter {
    name   = "name"
    values = ["Windows_Server-2016-English-Full-Base-*"]
  }

}

// How to use
/*
resource "aws_instance" "my_webserver_with_latest_ubuntu_ami" {
  ami           = data.aws_ami.latest_ubuntu.id
  instance_type = "t3.micro"
}
*/


output "latest_windows_2016_ami_id" {
  value = data.aws_ami.latest_windows_2016.id
}

output "latest_windows_2016_ami_name" {
  value = data.aws_ami.latest_windows_2016.name
}


output "latest_amazon_linux_ami_id" {
  value = data.aws_ami.latest_amazon_linux.id
}

output "latest_amazon_linux_ami_name" {
  value = data.aws_ami.latest_amazon_linux.name
}


output "latest_ubuntu_ami_id" {
  value = data.aws_ami.latest_ubuntu.id
}

output "latest_ubuntu_ami_name" {
  value = data.aws_ami.latest_ubuntu.name
}

```
