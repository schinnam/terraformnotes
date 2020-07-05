# terraformnotes
terraform notes pratice

Install - brew install terraform

Commands:

terraform version
terraform -help

terraform init      --> initializes terraform, reads all .tf files and installs required plugins. Eg: providers like AWS, GCP, Docker etc.

terraform fmt       --> format the .tf for easy readability and consistency.

terraform validate  --> validates syntax, errors in modules, attribute names, value types

terraform apply     --> read and apply .tf files in the folder

*known after apply* - values will be known only after the infrastructure is created.

terraform plan      --> called execution plan, shows what terraform will do when applied. Not Required as apple command itself has this step.

*terraform.tfstate* this files is created by terraform to store the state of the infra.

terraform show      --> inspect the current state of the infra.

*we can use the state of existing resource as input to create new resources*

terraform state     --> state management. Eg: terraform state list -> to list all resources.

As you change Terraform configurations, Terraform builds an execution plan that only modifies what is necessary to reach your desired state.

By using Terraform to change infrastructure, you can version control not only your configurations but also your state so you can see how the infrastructure evolved over time.

```
provider "aws" {
  profile    = "default"
  region     = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```
-/+ means that Terraform will destroy and recreate the resource
~   attributes can be updated in-place 
-   terraform destroys the resource

terraform destroy   -->  terminates all the resources specified by the configuration. Shows execution plan and takes cofirmation before execution. For complicated cases with multiple resources, Terraform will destroy them in a suitable order to respect dependencies.

Dependencies:
Implicit dependency is the preferred default way:
```
resource "aws_eip" "ip" {
    vpc = true
    instance = aws_instance.example.id
}

```

Explicit is sometimes required:
```
# New resource for the S3 bucket our application will use.
resource "aws_s3_bucket" "example" {
  # NOTE: S3 bucket names must be unique across _all_ AWS accounts, so
  # this name must be changed before applying this example to avoid naming
  # conflicts.
  bucket = "terraform-getting-started-guide"
  acl    = "private"
}

# Change the aws_instance we declared earlier to now include "depends_on"
resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"

  # Tells Terraform that this EC2 instance must be created only after the
  # S3 bucket has been created.
  depends_on = [aws_s3_bucket.example]
}

```

Resources are created in the order of dependency. Independent resources are created concurrently.

Provisioner:
Provisioners let you upload files, run shell scripts, or install and trigger other software like configuration management tools, etc.

```
resource "aws_instance" "example" {
  ami           = "ami-b374d5a5"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${aws_instance.example.public_ip} > ip_address.txt"
  }
}

```


```
provider "aws" {
  profile = "default"
  region  = "us-west-2"
}

resource "aws_key_pair" "example" {
  key_name   = "examplekey"
  public_key = file("~/.ssh/terraform.pub")
}

resource "aws_instance" "example" {
  key_name      = aws_key_pair.example.key_name
  ami           = "ami-04590e7389a6e577c"
  instance_type = "t2.micro"

  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/terraform")
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo amazon-linux-extras enable nginx1.12",
      "sudo yum -y install nginx",
      "sudo systemctl start nginx"
    ]
  }
}

```

terraform taint resource.id
terraform taint aws_instance.example

If a resource successfully creates but fails during provisioning, Terraform will error and mark the resource as "tainted". In next run, Terraform will remove any tainted resources and create new resources, attempting to provision them again after creation. Terraform taint command will not modify infrastructure, but does modify the state file in order to mark a resource as tainted. 


INPUT Variables:

```
variable "region" {
  default = "us-east-1"
}

---------------------------
provider "aws" {
  region = var.region
}

```
Input variables can be provided in different ways:
1. Command-line flags: terraform apply -var 'region=us-east-1'
2. From a file: terraform.tfvars or *.auto.tfvars or terraform apply -var-file="secret.tfvars" \ -var-file="production.tfvars"
3. From environment variables: TF_VAR_name
4. UI input

Strings and numbers are the most commonly used variables, but lists (arrays) and maps (hashtables or dictionaries) can also be used.

OUTPUTS:
Outputs are a way to tell Terraform what data is important. This data is outputted when apply is called, and can be queried using the terraform output command.

```
-> *.tf file

output "ip" {
  value = aws_eip.ip.public_ip
}

-
terraform output ip
```


