---
layout: post
title: Terraform Tutorial -  Terraform_Data Part 2
subtitle: A Gateway to Extending Terraform
gh-badge: [star, fork, follow]
---
 
## A Note About This Article

This guide uses embedded [Asciinema Recordings](https://asciinema.org/) of the topics illustrated.  These recordings are excellent because they not only illustrate the command and expected outputs, but are also fully interactive.  You can highlight the actual text, copy and paste data out of these videos. Because of this, I believe they are better than producing YouTube videos of these concepts as YouTube videos are not interactive.  I will also commit the raw `.cast` files in the code repositories so you can play them back locally from the comfort of your own terminal. If you have any comments or suggestions about improving these videos, make sure to reach out on [X/Twitter](https://x.com/aric49).

If these articles go over well, I will publish all future articles using these interactive recordings in the future. 

## Introduction

As we saw in the [last installment](2024-12-18-Terraform-Data-Blocks.md) of this tutorial series, Terraform Data resources are extremely powerful and flexible both for learning Terraform and testing functionality.  In this installment of the series, we will learn the basics of extending Terraform using the `provisioner` sub-block of the Terraform Data resource. 

Before moving forward, extending Terraform using a provisioner is generally regarded as a last resort for providing additional functionality.  Generally speaking, you always want to leverage available Terraform Providers or resources before using a resource provisioner. Terraform providers are designed around the Terraform paradigm and seamlessly integrate with other providers for the purposes of sharing data and consolidating resource state. Using a `provisioner` in a Terraform Data resource, you can provide additional functionality that might be not be currently available in any provider on the Terraform Registry.  This is useful for use cases such as the requirement to execute a specific CLI command, run a custom script, or perform a unique piece of business logic when a parent resource gets created or changed.  The [Terraform Registry](https://registry.terraform.io) hosts a surprising number of providers written by service providers and individual hackers like you or I.   Always check there first before using a Terraform provisioner. 

As usual, the full example in this article can be found on my [Automation Tutorials GitHub Repo](https://github.com/aric49/automation-tutorials/tree/main/terraform/terraform_data2)


## Using the `provisioner` Sub-Block

Provisioners are ways that Terraform resources can be extended to be useful in situations in which no existing Terraform providers or resources can be used. At the time of writing, there are 3 Terraform Provisioners that can be used in the Terraform Data resource: 

* **file**: Used for copying files from the machine running Terraform to a remote machine
* **local_exec**: Executes a command on the local machine running Terraform (Your machine in these examples)
* **remote_exec**:  Executes a command on a remote machine (usually over SSH or WinRM)

The general syntax for configuring a provisioner will always resemble the following: 

```hcl
resource terraform_data "my_resource" {
    #Other data block configuration: 

    provisioner "$provisioner_type" {
        #configuration
    }
}
```

Depending on the type of provisioner (`file` or `remote_exec`), there may be a need to specify and additional `connection` sub-block as well so that Terraform knows how to connect to the remote machine. 

The scope of this tutorial will cover the `local_exec` provisioner to illustrate how you can use Terraform to execute a Python script on your local machine during resource creation and changes. 

## Building the `local_exec` Provisioner

Before building out the Terraform resources, we will first write a short Python script that will look up the value of an environment variable called, `TERRAFORM_VARIABLE` and print it to the screen. For this script to run successfully, make sure you have Python 3.10+ installed and properly configured so that the `python -V` command shows Python 3.10 or higher: 

```shell
$ python -V
Python 3.13.1
```

In your current working directory, create a file called, `python_script.py` and paste in the following code: 

```python
import os


def main(): 
    #Print a basic  message based on the value of a Terraform variable
    terraform_variable = os.environ.get("TERRAFORM_VARIABLE", "UNDEFINED")
    print(f"hello world! I'm being executed by Terraform!!")
    print(f"The Value of the Terraform variable is {terraform_variable}")


if __name__ == "__main__":
    main()
```

The main functionality of this script is simple:  Use the `os` library to lookup an environment variable called, `TERRAFORM_VARIABLE` and store the value to a local variable called, `terraform_variable`.  If the environment variable does not exist, the value of `terraform_variable` will be `UNDEFINED`. Finally, it prints two messages: one that states: `hello world! I'm being executed by Terraform!!` and a second that shows the value of the Terraform variable: `The Value of the Terraform variable is {terraform_variable}`. 

Before moving to writing the Terraform code, we can check to ensure this script works as expected by executing it without defining the environment variable, and a second time explicitly defining the environment variable: 

<script src="https://asciinema.org/a/696310.js" id="asciicast-696310" async="true"></script>



Next, we can create our main terraform file:  `main.tf`, with the following contents: 

**main.tf**
```hcl
resource "terraform_data" "my_block" {
  input = var.provisioner_data

  triggers_replace = [
    var.provisioner_data
  ]

  provisioner "local-exec" {
    command = "python python_script.py"
    environment = {
      "TERRAFORM_VARIABLE" = var.provisioner_data
    }
  }
}
```

This will instantiate a `terraform_data` resource tied to the value of the variable `var.provisioner_data`.  If this variable changes, it will cause a change to trigger on the Terraform resource based on the values supplied in the `triggers_replace` block. Since `triggers_replace` is a list, we could also provide multiple resources or variables that should trigger the script execution should they change. In this case, we will just use the value of the variable: `var.provisioner_data` to trigger the re-execution of the script. 

We are similarly using an `environment` sub-block to expose the value of this Terraform variable as the value of the environment variable, `TERRAFORM_VARIABLE`. When the provisioner runs, it will create a shell session with the environment variable defined called, `TERRAFORM_VARIABLE`. It will then execute the command defined as the value of the `command` parameter.  In this case, it will be the same Python command we executed when testing the script. 

Next, create the following supporting files: `variables.tf`, `output.tf`, and `terraform.tfvars`.  Use the following examples to populate these files: 

**variables.tf**:
```
variable "provisioner_data" {
  type        = string
  description = "This variable provides input to the terraform_data block"
  default     = null
}
```

**output.tf**:
```
output "terraform_data_output" {
  value = terraform_data.my_block.output
}
```

**terraform.tfvars**:
```
#Controls the terraform_data block state

provisioner_data = "Initial Value"
```


## Executing the Full Example: 

Once all the file are in place, we can execute the full example using the standard Terraform Lifecycle commands: `terraform init`, `terraform plan`, `terraform apply` and when you have finished `terraform destroy`.  Don't forget to change the value of the `var.provisioner_data` variable to test that the script execution is working as expected. Make sure to pay attention to the `terraform apply` `local-exec` output lines to make sure the output is changing as expected: 

<script src="https://asciinema.org/a/696316.js" id="asciicast-696316" async="true"></script>


## Conclusion

It is my hope that you can see how powerful `Terraform_Data`resource blocks can be for extending Terraform in ways that are not covered by existing providers or resources. By using Terraform Data blocks, custom logic can be triggered based on changes to upstream resources, data lookups, or other native Terraform objects. This can be an extremely useful way to integrate Terraform into existing legacy processes, or custom business logic that may never have a Terraform provider written for it. I hope this article and accompanying videos were useful to you! 

**Remember**:  If this tutorial was helpful to you in anyway, you can show me your appreciation by [Buying Me a Coffee](https://buymeacoffee.com/aric49)

## Resources: 

1. [Terraform Data Documentation from Hashicorp](https://developer.hashicorp.com/terraform/language/resources/terraform-data)
2. [Terraform Provisioners Documentation from Hashicorp](https://developer.hashicorp.com/terraform/language/resources/provisioners/syntax)
3. [Full Examples and Code on GitHub](https://github.com/aric49/automation-tutorials/tree/main/terraform/terraform_data2)
4. [My Asciinema Page](https://asciinema.org/~aric49)
