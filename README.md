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



