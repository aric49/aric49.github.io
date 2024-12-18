---
layout: post
title: Terraform Tutorial -  Terraform_Data Introduction
subtitle: The best way to learn and practice Terraform
gh-badge: [star, fork, follow]
---
 
## Introduction

 Infrastructure Automation can be an incredibly complex topic and quite intimidating for new developers and engineers who are just getting started. If you are just getting started with learning tools such as Ansible or Terraform, it can be difficult to practice writing code in these languages without talking to "real" infrastructure.  We can gain confidence on what the downstream impact of a code change between Terraform resources might be by simulating it through the use of non-impactful Terraform resources.   

 Even if you are an experienced developer, you may wish to want to mock up problems in a local directory using fake Terraform resources to try and rule out variables or bugs that might lurk outside of Terraform.  In the past, (pre Terraform 1.0), one would have to use the `null_resource` provider from Hashicorp to mock up Terraform resources, practice writing Terraform code, or simply create a shim for execution of external scripts or tooling. 

 Recent versions of Terraform have included a replacement for the `null_resource` provider using the native `terraform_data` resource block. This resource block is incredibly handy and can be leveraged immediately without having to instantiate any providers or require any additional Terraform setup.  If you have the Terraform binary loaded in your shell's `$PATH` variable, you already have access to it.  This tutorial series will be looking at the `terraform_data` resource block and ways it can be leveraged. In the first part of this tutorial series, I will be looking at mocking up Terraform resources, resource dependencies, and simulating changes locally. 

 Before getting started, please ensure you are running Terraform version 1.6 or higher.  I will be using Terraform version v1.10.2, as it is the latest at the time of writing. If you want the full version of this code, with all the examples, you can find it on my [GitHub Automation Tutorials Repository](https://github.com/aric49/automation-tutorials/tree/main/terraform/terraform_data) 

 ```shell
 $ terraform -v
Terraform v1.10.2
on linux_amd64
 ```

## Creating Your First `terraform_data` Resource Block

Creating a `terraform_data` resource block requires nothing more than simply creating a `main.tf` file and adding an empty `terraform_data` resource. In a new directory, create a file called, `main.tf` and populate it with the following code: 

**main.tf**:
```hcl
resource "terraform_data" "resource1" {
    //intentionally leaving empty
}
```
As you can see, we are creating a simple resource called `resource1` with no arguments using the resource type of `terraform_data`. Running the `terraform plan` command you can see that Terraform will create a one resource called, `resource1`: 

```shell
$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # terraform_data.resource1 will be created
  + resource "terraform_data" "resource1" {
      + id = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

Running the terraform apply command, and passing "yes" at the subsequent prompt will create that resource: 

```shell
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # terraform_data.resource1 will be created
  + resource "terraform_data" "resource1" {
      + id = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

terraform_data.resource1: Creating...
terraform_data.resource1: Creation complete after 0s [id=c02782b5-d233-6586-989d-d8cf0821f5b7]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

You have successfully created your first `terraform_data` resource. This resource only exists in your `terraform.tfstate` file and can be created and destroyed as often as you wish. Let's look at the file `terraform.tfstate` in a a text editor and see what it looks like. 

Under the `resources: []` section of the state file JSON, you will find a single resource listed: 

```json
    {
      "mode": "managed",
      "type": "terraform_data",
      "name": "resource1",
      "provider": "provider[\"terraform.io/builtin/terraform\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "id": "c02782b5-d233-6586-989d-d8cf0821f5b7",
            "input": null,
            "output": null,
            "triggers_replace": null
          },
          "sensitive_attributes": []
        }
      ]
    }
```

Notice the name of the resource is: `resource1`. You can see the `type` is `terraform_data` and it was created from the `terraform.io/builtin` provider. Under the `instances: []` section, you can see some additional attributes such as a unique resource identifier, as well as `input` and `output` values currently set to `null`. These input and output parameters will be what we are going to look at next. 

## Controlling The Terraform Data Block using Variables

 One of most useful aspects of the `terraform_data` block is the ability for it to accept an input through the resource parameter aptly named, `input`. Since `terraform_data` blocks do not do very much on their own, the value that is passed in through the `input` parameter becomes available to other resources as an `output` attribute that can be referenced. Changes to the `terraform_data` block become dependent on the value of the `input` parameter as well.

 To see how this works in practice, in `main.tf`, let's add a second resource blocked named `resource2` which has an input parameter based on the variable:  `var.variable_1`

 **main.tf**: 
 ```hcl
resource "terraform_data" "resource2" {
    input = var.variable_1
}
 ```

Similarly, create an additional file called `variables.tf`, with the following content: 

**variables.tf**:
```hcl
variable "variable_1" {
    type = string
    description = "This variable controls the lifecycle of the terraform data block"
    default = "My default value"
}
```

If we run `terraform plan` and `terraform apply`, you will see that a new `terraform_data` resource has been provisioned: 

```shell
$ terraform plan
terraform_data.resource1: Refreshing state... [id=c02782b5-d233-6586-989d-d8cf0821f5b7]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # terraform_data.resource2 will be created
  + resource "terraform_data" "resource2" {
      + id     = (known after apply)
      + input  = "My default value"
      + output = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.



$ terraform apply
terraform_data.resource1: Refreshing state... [id=c02782b5-d233-6586-989d-d8cf0821f5b7]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # terraform_data.resource2 will be created
  + resource "terraform_data" "resource2" {
      + id     = (known after apply)
      + input  = "My default value"
      + output = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

terraform_data.resource2: Creating...
terraform_data.resource2: Creation complete after 0s [id=e3be9d1d-7ede-2551-8325-c7d0106add7d]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

We have now successfully created a secondary resource block which points to the value of `var.variable_1` as the input. Now, let's create a change by overriding the value of `var.variable_1` creating a new file called, `terraform.tfvars` in the same directory. Variable values assigned using `terraform.tfvars` will by default override any default values already ascribed in the `default` parameter of the `variables` block we created previously: 


**terraform.tfvars**: 
```hcl
variable_1 = "This is an override."
```

We can check this change is working as expected by running the `terraform plan` command once more: 

```shell
$ terraform plan
terraform_data.resource1: Refreshing state... [id=c02782b5-d233-6586-989d-d8cf0821f5b7]
terraform_data.resource2: Refreshing state... [id=e3be9d1d-7ede-2551-8325-c7d0106add7d]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # terraform_data.resource2 will be updated in-place
  ~ resource "terraform_data" "resource2" {
        id     = "e3be9d1d-7ede-2551-8325-c7d0106add7d"
      ~ input  = "My default value" -> "This is an override."
      ~ output = "My default value" -> (known after apply)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

As you can see in the above output, Terraform has correctly identified that a change has occurred in our `terraform_data.resource2` block because the value of the variable controlling that block has changed. However, not only the input parameter has changed, the `output` attribute has changed as well. This is due to the fact that `terraform_data` blocks automatically generate the value of the `output` attribute based on the value of the `input` parameter.   We can leverage this to simulate implicit resource dependencies in Terraform. If you wish to apply the above change, you may do so now using the `terraform apply` command. 


## Outputs and Resource Dependencies 

 Terraform has a very well thought out dependency system based on resources and the relationships between them. Dependencies can take two forms:  *Implicit* and *Explicit* dependencies.  

 Implicit dependencies are dependencies formed between resources through shared attributes. For example, a load balancer may need to know the IP address of a server it is sending traffic to. By referencing the server's IP address creates a relationship between the server and load balancer. If the IP address ever changes, it will cause an update on the load balancer so the load balancer has the correct IP address of the server. 

 Explicit Dependencies are forged between resources manually by specifying a special `depends_on` attribute from a resource. This type of dependency ensures that one resource will not be created without first having the dependent resource be created first.  This is useful in situations where two resources will not share attributes but still are required to be maintain some relationship such as creation or destruction order. Further explanation of Explicit Dependencies goes beyond the scope of this article. 

 Using the `input` and `output` parameters of the `terraform_data` blocks, we can create *Implicit Dependencies* between two resources to simulate changes. In the file  `main.tf` we will create an additional `terraform_data` block called: `resource3` to illustrate how implicit dependencies work:

 **main.tf**: 
 ```
 resource "terraform_data" "resource3" {
    input = terraform_data.resource2.output
}
 ```

 Notice in this example, the `input` parameter is referencing the `output` attribute of the `resource2` terraform data block. This means that any changes made to `resource2` will also reflect as a change in the `resource3` block. Let's apply this configuration as-is to see it in action: 

```shell
$ terraform plan
terraform_data.resource1: Refreshing state... [id=c02782b5-d233-6586-989d-d8cf0821f5b7]
terraform_data.resource2: Refreshing state... [id=e3be9d1d-7ede-2551-8325-c7d0106add7d]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # terraform_data.resource3 will be created
  + resource "terraform_data" "resource3" {
      + id     = (known after apply)
      + input  = "This is an override."
      + output = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

As you can see, a third resource block has been provisioned which has the same `input` parameter value has `resource2`.  By connecting the output of `resource2` to the input of `resource3`, we have now created an implicit dependency between the two resources. You can go ahead and apply this change to create the `resource3` resource. 

We can see the effect of changes between implicitly related resources by changing the value of the `var.variable1` variable. 
In `terraform.tfvars`,  Let's update the value of `var.variable1` to a new value: `This is another override`.

**variables.tfvars**: 
```
variable1 = "This is another override." 
```

Running `terraform plan` and `terraform apply` we can see that by changing the value of one variable, we have created a cascading change that impacts two resources: 

```shell
$ terraform plan
terraform_data.resource1: Refreshing state... [id=c02782b5-d233-6586-989d-d8cf0821f5b7]
terraform_data.resource2: Refreshing state... [id=e3be9d1d-7ede-2551-8325-c7d0106add7d]
terraform_data.resource3: Refreshing state... [id=09b332fa-31ec-d64f-04eb-cad588bda7cb]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # terraform_data.resource2 will be updated in-place
  ~ resource "terraform_data" "resource2" {
        id     = "e3be9d1d-7ede-2551-8325-c7d0106add7d"
      ~ input  = "This is an override." -> "This is another override."
      ~ output = "This is an override." -> (known after apply)
    }

  # terraform_data.resource3 will be updated in-place
  ~ resource "terraform_data" "resource3" {
        id     = "09b332fa-31ec-d64f-04eb-cad588bda7cb"
      ~ input  = "This is an override." -> (known after apply)
      ~ output = "This is an override." -> (known after apply)
    }

Plan: 0 to add, 2 to change, 0 to destroy.
```
Notice that Terraform marks the `output` attribute of `resource2` as **known after apply**.  This is because Terraform technically has to compute this value during the apply phase.  We know the functionality of the resource is to use the same value for the output as what was provided to the input, but Terraform doesn't know that until the apply phase runs.  This is why both the `input` and `output` of `resource3` is marked as **Known after apply**. 

## Conclusion

Although simple, it is my hope that this brief tutorial sheds more light on how the `terraform_data` block works. I have used this block numerous times in multiple Terraform projects as a helpful way to not only create resource dependencies, but also as a shim to extend Terraform by executing scripts and additional logic that exists outside of Terraform. The topic of the next lesson will be leveraging `terraform_data` blocks with a shell provisioner to perform in order to extend Terraform in a multitude of different ways. 

Remember:  If this tutorial was helpful to you in anyway, you can show me your appreciation by [Buying Me a Coffee](https://buymeacoffee.com/aric49)

## Resources: 

1. [Terraform Data Documentation from Hashicorp](https://developer.hashicorp.com/terraform/language/resources/terraform-data)
2. [Full Examples and Code on GitHub](https://github.com/aric49/automation-tutorials/tree/main/terraform/terraform_data)
