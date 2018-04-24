# DevOps Playground AWS and Terraform!

## Introduction

Terraform is defined by HashiCorp, the creator of Terraform, as a tool for building, changing, and versioning infrastructure safely and efficiently.

That sounds like an infrastructure as code solution, because Terraform is an infrastructure as code solution.


## Agenda - This will be Hands-on!

1. SSH into your workstation
2. Editing the configuration file to create an ec2 instance
3. Running Terraform
4. Adding AWS tags to the instance
5. Scaling the infrastructure
6. Using variables
7. Scaling down the infrastructure
8. Destroying the environment

---

## 1. SSH into your workstation

* `ssh devops@your.IP`
* You will be prompted to confirm that the key looks correct, type `y` then Enter
* Then enter the password: `playground` and press Enter

---
## 2. Editing the configuration file to create an ec2 instance

The main.tf is the main configuration file that is to be used for the remainder of this playground.  

Open the file and take a look around.

The goal here is to create an EC2 instance. We have already pre-populated an aws_instance stanza with some values that are necessary for you to be able to run it, so please leave them in.

### Adding an instance type
The first change you need to do is defining an instance type. 
Nothing larger than a t2.micro is required for today's Playground.

Open the main.tf file with `vi main.tf` and do the following.
1. Replace the ... with a name for your instance, e.g.
```
resource "aws_instance" "playground" {
```
2. Add an instance type to your instance, other items should  **NOT** be edited.

```
instance_type = "t2.micro"
```

Save the file.

---
## 3. Running Terraform

### Running terraform plan
Now run `terraform plan`

```
Plugin reinitialization required. Please run "terraform init".
Reason: Could not satisfy plugin requirements.

Plugins are external binaries that Terraform uses to access and manipulate
resources. The configuration provided requires plugins which can't be located,
don't satisfy the version constraints, or are otherwise incompatible.

1 error(s) occurred:

provider.aws: no suitable version installed
version requirements: "(any version)"
versions installed: none

This error occured because we had not run the tf init command.

```

### Terraform init

The terraform init command is used to initialize a working directory containing Terraform configuration files. This is the first command that should be run after writing a new Terraform configuration or cloning an existing one from version control. It is safe to run this command multiple times.

After tunning terraform init as the aliases have been included for you run terraform -help to see the list of variables available.

```
$ terraform -help

* Common commands:

     apply              Builds or changes infrastructure
     console            Interactive console for Terraform interpolations
     destroy            Destroy Terraform-managed infrastructure
     env                Workspace management
     fmt                Rewrites config files to canonical format
     get                Download and install modules for the
     configuration
     graph              Create a visual graph of Terraform resources
     import             Import existing infrastructure into Terraform
     init               Initialize a Terraform working directory
     output             Read an output from a state file
     plan               Generate and show an execution plan
     providers          Prints a tree of the providers used in the configuration
     push               Upload this Terraform module to Atlas to run
     refresh            Update local state file against real resources
     show               Inspect Terraform state or plan
     taint              Manually mark a resource for recreation
     untaint            Manually unmark a resource as tainted
     validate           Validates the Terraform files
     version            Prints the Terraform version
     workspace          Workspace management

* All other commands:

     debug              Debug output management (experimental)
     force-unlock       Manually unlock the terraform state
     state              Advanced state management

```
Run `tf init` now.


### Terraform Plan
Run `terraform plan` to see the changes that are going to be applied.
```
Plan: 1 to add, 0 to change, 0 to destroy.
```


### Terraform Apply
Now that you're happy with the plan, you can run `terraform apply`.
This will run another plan, if you are happy withit you can then enter `yes` and press Enter.
Terraform will then apply your change.


---

## 4. Adding AWS tags to the instance

We can add a tag in place without having to re-provision the infrastructure, the next step will demonstrate this.

Under the existing "Identity" tag, add the "Name" tag, with your name as the value:
```
 tag {
     Identity = ...
     Name = "Mark"
 }
```

Run `terraform apply` and if you are happy with the plan, go ahead and run it.


### Tidying the code

In some cases you may come across some code that isn't well formatted. Terraform has a command allowing you to format the file. View the main.tf file before and after we run the `terraform fmt` command.

Any changes? What did you see happen?  If no changes then you are already writing efficient code.

---

## 5.Scaling the infrastructure

What if you want to add more than one instance?  You can use the count meta-parameter to do this.
This parameter allows you to very simply state how many instances of a resource you want Terraform to create.

In this exercise, we want to create 2 AWS instances.

Add the item `count = 2` at the top of the aws_instance stanza, so that it looks like this:

```
resource "aws_instance" "playground" {
  count = 2
```

---

## 6. Using variables

### Interpolation syntax

The interpolation syntax is powerful and allows you to reference variables, attributes of resources, call functions.

You can perform simple math in interpolations, allowing you to write expressions such as ${count.index + 1}. And you can also use conditionals to determine a value based on some logic.

As you build more complex infrastructure, your terraform configuration will become longer and more complex. In order to avoid repetition, you can use variables. These variables also allow you to make your configuration re-usable by other teams/users, by using the same configuration and simply changing the variables.

### Creating a variable
Let's create a variable, you can place it under the AWS Provider stanza.

```
 variable "username" {        
   default = "Mark"
 }
```

Now let's make use of this variable, by replacing the Name that you set for your instance.

```
Name   = "${var.username}"
```
Now run `terraform apply`.

### Making use of the count index

It can very easily become very confusing between instances as both instances have the same Name tag, which is used on the AWS Console as the Instance name. To resolve this we add a _counter_ to the Name tag.

```
Name   = "${var.username} ${count.index+1}"
```

Run `terraform apply` to see your changes come to life.

---

## 7. Understanding the state

The terraform show command is used to provide human-readable output from a state or plan file. This can be used to inspect a plan to ensure that the planned operations are expected, or to inspect the current state as Terraform sees it.

Run `terraform show` and look at the output of the command.

---

## 8. Terraform outputs

At the end of a terraform run, you might want to output specific data.

Add a final item to the main.tf file to display the public IP for the new instances that have been created.

```
output "public_IP" {
  value = "${aws_instance.playground.*.public_ip}"
}
```

Now run `terraform output` to show the outputs.

This will fail the first time because it looks inside the state, not the config, for the outputs. To pull this through we need to refresh our state, using the `terrafrom refresh` command.

This should now show the output of your terraform environments:

```
public_IP = [
    54.72.161.232,
    54.94.212.153
]
```

---
## 7. Scaling down the infrastructure

If you have created too many instances, you can scale down by changing the count number.

Edit the main.tf file and change the instance count to 1

```
count = 1 // OR remove the count parameter altogether
```

Run `terraform apply` and confrm if you're happy with the plan.

You Should now only be one instance runnning or none if you ran destroy.


---
## 8. Destroying the Environment

When you are done with your environment, you can choose to destroy everything you created previously using the `terraform destroy` command.

Terraform shows its execution plan and waits for approval before making any changes.
