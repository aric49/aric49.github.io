---
layout: post
title: Test using Terminal Videos
subtitle: A terminal recorded video test
gh-badge: [star, fork, follow]
---
 
## Test Post

This is a test post to see how Asciinema videos render. I may start using Asciinema videos if this is a good solution.


### Embedded Player
**Can you see me?**
<script src="https://asciinema.org/a/696204.js" id="asciicast-696204" async="true"></script>


### HTML

**Can you see me?**

<a href="https://asciinema.org/a/696204" target="_blank"><img src="https://asciinema.org/a/696204.svg" /></a>

### Markdown

**Can you see me?**

[![asciicast](https://asciinema.org/a/696204.svg)](https://asciinema.org/a/696204)


## Using the `provisioner` Sub-Block

## Conclusion

Although simple, it is my hope that this brief tutorial sheds more light on how the `terraform_data` block works. I have used this block numerous times in multiple Terraform projects as a helpful way to not only create resource dependencies, but also as a shim to extend Terraform by executing scripts and additional logic that exists outside of Terraform. The topic of the next lesson will be leveraging `terraform_data` blocks with a shell provisioner to perform in order to extend Terraform in a multitude of different ways. 

Remember:  If this tutorial was helpful to you in anyway, you can show me your appreciation by [Buying Me a Coffee](https://buymeacoffee.com/aric49)

## Resources: 

1. [Terraform Data Documentation from Hashicorp](https://developer.hashicorp.com/terraform/language/resources/terraform-data)
2. [Full Examples and Code on GitHub](https://github.com/aric49/automation-tutorials/tree/main/terraform/terraform_data2)
