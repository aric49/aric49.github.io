---
layout: post
title: Running Ansible in Docker
subtitle: Deploying a re-usable Ansible for CI/CD Deployments
gh-badge: [star, fork, follow]
---
# Running Ansible in Docker

Even in the fast paced world of CloudNative architectures, containers, and Kubernetes, managing resources on virtual machines and traditional server infrastructure is still a pain point for many engineers. Luckily, the [Ansible](https://ansible.com) project makes configuration management of servers and other Infrastructure easy using its robust language to define OS-level configuration.  However, the execution of Ansible Playbooks at scale, in a repeatible way is still quite cumbersome despite tools such as AWX and Ansible Tower which provide many automated capabilities around execution of playbooks and one-off tasks. Sometimes however, you just want to run a simple playbook in a CI/CD pipeline without the overhead of managing Ansible runners or installations of AWX and Ansible Tower. 

Using the power of Docker and containers, it is possible to build a container which contains (heh-heh) an installation of Ansible which can be executed from any CI/CD pipeline which has Docker enabled.   This workflow is useful for platforms such as GitLab CI, Bitbucket Pipelines, or Jenkins CI.  You can even extend this Docker image using any other tools or plugins your individual usecase may require. Below is a simple `Dockerfile` based on Alpine Linux which installs Ansible and sets it as the containers entrypoint: 

```Dockerfile
FROM alpine:latest

RUN apk add python3 ansible

WORKDIR /ansible

#Set this so that Ansible doesn't prompt you to accept host keys everytime it runs
ENV ANSIBLE_HOST_KEY_CHECKING False

ENTRYPOINT ["ansible"]
```

Use the following command to build the Docker Image: 

```shell
$ docker build -t ansible:latest .
```

Finally, run Ansible by Volume mounting the location of your SSH keys and any inventory files you may wish to include: 

```shell
$ docker run --rm -it -v $PWD/inventory.yml:/ansible/inventory.yml -v ~/.ssh/:/root/.ssh ansible:latest all -m ping -i /ansible/inventory.yml --private-key ~/.ssh/your-private-key.pem
```

The above Docker command is volume mounting a file called: `inventory.yml` which exists in your current directory. It is also volume mounting the `.ssh` directory to the default location inside in the container: `/root/.ssh/`. Since `ansible` is set as the entrypoint of the container, you simply need to pass in any additional arguments to Ansible after specifying the Docker image tag. In this case, we are using the `ping` module, specifying the path to the volume mounted inventory as well as the SSH key: `all -m ping -i /ansible/inventory.yml --private-key ~/.ssh/your-private-key.pem`

You should see Ansible successfully able to ping the targets specified in your inventory file: 

```
192.168.25.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

**NOTE:** It may be tempting to build the private keys into the Ansible Docker image.  **DO NOT DO THIS** as you would be exposing private key files to any container registries you would be pushing your container to. It is much better to volume-mount the private keys at runtime. 

It is even possible to perform playbook executions from within containers. For example, you can use the following command to run a playbook using the `ansible-playbook` command: 

```shell
$ docker run --rm -it -v $PWD/inventory.yml:/ansible/inventory.yml -v ~/.ssh/:/root/.ssh -v $PWD/test-playbook.yml:/ansible/test-playbook.yml --entrypoint="" ansible:latest ansible-playbook test-playbook.yml
```

This will execute the playbook as follows: 

```
PLAY [Test Ansible Playbook] ***************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************
ok: [localhost]

TASK [Run a shell command] *****************************************************************************************************************
changed: [localhost]

TASK [Display the output] ******************************************************************************************************************
ok: [localhost] => {
    "msg": "The  output was Hello World!"
}

PLAY RECAP *********************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Below I have provided these examples in a GitLab repository as well as a prebuilt container image on Quay.io:

* [GitLab Repository](https://gitlab.com/arenzo/containerized-ansible)
* [Quay.IO Docker Image](https://quay.io/repository/aric49/ansible?tab=info)