# DevOps Playground AWS and Terraform!

## Introduction

Terraform is defined by HashiCorp, the creator of Terraform, as a tool for building, changing, and versioning infrastructure safely and efficiently.

That sounds like an infrastructure as code solution, because Terraform is an infrastructure as code solution.


## Agenda - This will be Hands-on!

1. Connect to and AWS instance we have provided details of for you.
2. Editing the configuration file to create an ec2 instance
3. Running Terraform
4. Adding AWS tags to the instance
5. Scaling infrastructure
6. Using variables
7. Scaling down the infrastructure
8. Destroying the environment

---

## 1. SSH into your instance

* `ssh devops@your.IP`
* You will be prompted to confirm that the key looks correct, type `y` then Enter
* Then enter the password: `playground` and press Enter

---
## 2. EC2 Instance Terraform configuration 

The main.tf is the main configuration file that is to be used for the remainder of this playground.  

Open the file and take a look around.

Define an instance type.  Nothing larger than a t2.micro is required for today's Playground.

```
 instance_type = "t2.micro"
```

Save the file.

Now run tf plan

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

---

### Terraform init

The terraform init command is used to initialize a working directory containing Terraform configuration files. This is the first command that should be run after writing a new Terraform configuration or cloning an existing one from version control. It is safe to run this command multiple times.

After tunning terraform init or tf init as the aliases have been included for you run terraform -help to see the list of variables available.

* terraform -help

* Common commands:
```
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

```
tf init

```
---

### 2. Lets Create an instance

Open the main.tf file with `vi main.tf` and do the following.
1. Replace the ... with a name for your instance, e.g.
```
resource "aws_instance" "playground" {
```
2. Add an instance type to your instance, other items should  **NOT** be edited.

```
instance_type          = "t2.micro"
```

---

### 3.0 Terraform Plan
Run terraform plan to see the changes that are going to be applied.
```
Plan: 1 to add, 0 to change, 0 to destroy.
```
### 3.1 View AWS console

As you can see the instances are starting to be created but they have no name, not very useful so let look to how we can solve this.

---

### 4. Adding Tags

We can add a tag in place without having to re-provision infrastructure, next step is to demonstrate this.

1. Exercise: Add the Name tag
2. Edit the main.tf file and add the following item:
3. Add under region
```
Add Under tags

 tag {
     Name = "Atendee Name"
 }

```
Run terraform plan to view the changes that are going to be applied and apply.

```
tf plan


tf apply

```
---

### Tidying the code

In some cases you may come across come code that does not look very clean. Within Terraform there is a cmd that we can use to format the file.  View the main.tf file before and after we run the fmt cmd as outlined below:

Run the command:
```
tf fmt main.tf
```
Any changes? What did you see happen?  If no changes then you are already writing efficient code.

---

### 5.Scaling Infrastructure

What if you want to add more than one instance?  You can add the count variable to resolve this, see below:

```
meta-parameter *count*

```

Add the item count = 1 underneath resource so it will look like the following:

```

resource "aws_instance" "test" {
count = 1

```
---

### 6. Confusion between instances, how to identity?

**Interpolation**

The interpolation syntax is powerful and allows you to reference variables, attributes of resources, call functions.

You can perform simple math in interpolations, allowing you to write expressions such as ${count.index + 1}. And you can also use conditionals to determine a value based on some logic.

This looks useful to our excersize to lets add the interpolation to the

Now this is where your environment can become a little messy if you are creating multiple instances.  

First let's create a variable, you can place it under the AWS Provider stanza.

```
 variable "username" {        
   default = "AtendeeID"
 }
```

Now let's make use of this variable, by replacing the Name that you set for your instance.

```
Name   = "${var.username} ${count.index}"

```
Now run tf plan to see the proposed changes going to be applied.


This can very easily become very confusing between instances and the current naming convention shows the first instance as [0] but we need to identity them better as 0 is more used in binary.  To resolve this we add an additional item to the count.index.

```
${count.index+1}"

```

Run tf plan to see your proposed changes:

Anyone experiencing any errors?

If all is good then you can run terraform apply


**Refresh the AWS console**

---
### 6.1 Terraform show

The terraform show command is used to provide human-readable output from a state or plan file. This can be used to inspect a plan to ensure that the planned operations are expected, or to inspect the current state as Terraform sees it.

### Current state - used by terraform to know what it manages

Add a final item to the main.tf to display output IP for the new instances have been created.

### Add the output to the main.tf

```
output "public_IP" {
  value = "${aws_instance.test.*.public_ip}"
}

```
### 6.2 run terraform output

This will fail the first time because it looks inside the state, not the config, for the outputs. To pull this through we need to perform a Refresh

 ```
tf refresh

```
This should now show the output of your terraform environments

```
e.g

public_IP = [
    54.72.161.232
]
```

---
### 7.  Scaling back the infrastructure

There are a couple of options here

1. Go back to 1 instance only
2. Run tf destroy to remove all instances.

```
Edit the main.tf file instance count to 1

count = 1 OR remove count item altogether

```

* Whats going to happen?

* run tf apply and confirm the output.

* You Should now only be one instance runnning or none if you ran destroy.


To test this run your previous command, this should show only 1 IP address.

```
terraform output

```
---
### 8. Final Step Destroying the Environment

*** terraform destroy***

Terraform shows its execution plan and waits for approval before making any changes.


```
tf destroy

```
