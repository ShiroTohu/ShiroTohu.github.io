---
layout: post
title: 
categories: [Blogging, Server]
tags: [linux, docker, podman, distrobox, virtualisation]
date: 2025-11-27 12:00 +1000
---

---
layout: post
title: Using Ansible and OpenTofu to Automate Server Deployment and Configuration
categories:
  - Blogging
  - Server
tags:
  - homelab
  - linux
  - server
date: 2026-03-42 12:00 +1000
---

## Introduction
Last year I became increasingly frustrated managing my server infrastructure as it was becoming increasingly harder. It became immutable and hard to keep track of intricate details. I was quite concerned at my current deployment as humans are often prone to make errors and one small misconfiguration could lead to a security incident. Therefore, I set in finding ways to document my infrastructure, but what I found was much greater.
## Quick Summary of the Technologies
This blog post goes over OpenTofu and Ansible. I wanted to quickly discuss the purpose of these technologies and when you would use them. In simple terms, after planning your network infrastructure you can use OpenTofu to spin up virtual machines for on your preferred cloud provider using the *code* you have written. This is why it is called the term *Infrastructure as Code*. After the Virtual machines have been configured Ansible is used to automate the processes inside of the machine like installing packages and copying files.  

![Homelab Diagram](./assets/img/posts/automation/Diagram.png){: width="400" height="200" }
You write peer reviews, submit pull requests, run unit tests on your code. Because you declare how your servers should look like it allows for repeatable results making your infrastructure more reliable. There are many benefits to using Infrastructure as Code and I am glad I have implemented it in my home deployment.

I also want to stress that planning is very important for production environments. I believe that how you plan will determine how your code repository will turn out. I also believe that if not enough planning is done you'll end up having to refactor more times than needed. Make sure you understand the technologies you want to use and how you want services to communicate with each other.
## Project Layout
Due to the private nature of this project and general security concerns I won't be releasing the source code for my home deployment. But I want to share what I have done and the processes I followed in my project.

In the project layout I have two separate folders. One for Ansible and one for OpenTofu. I like to have these two separated as opposed to having Ansible automatically deploy after server provisioning as it allows for more fine grained control and to use playbooks as tools.
### OpenTofu
My OpenTofu configuration is quite simple as I have limited resources to work with. In my configuration I use the [bpg/proxmox](https://registry.terraform.io/providers/bpg/proxmox/latest/docs) provider and setup API keys on my Proxmox instance so that OpenTofu can interface with my Proxmox instance securely.

When deploying virtual machines I use the Debian cloud image. You can browse the cloud images [here](https://cloud.debian.org/images/cloud/). The OpenTofu snippet is shown below:
```terraform
resource "proxmox_virtual_environment_download_file" "centos_cloud_image" {
  content_type = "import"
  datastore_id = "local"
  node_name    = var.node_name
  url          = "https://cloud.debian.org/images/cloud/trixie/20260413-2447/debian-13-genericcloud-amd64-20260413-2447.qcow2"
  checksum     = 3d020c868339c8387f6f40f63e6b5ccb6174e6f2963510cb51cacfd6618244e96549d1b60b3d629df3963cb5350c6dfaf5cf34998301aa578b9d455a76d3c434
}
```
I do not like using the latest version as the URL as when a new version is pushed the checksum will not match and it will fail to run. If you decide on a different distribution make sure that you are downloading a file with the `.qcow2` extension and that it is labelled `genericcloud`.

When configuring the virtual machines with Ansible you need SSH in order for Ansible to  run commands inside the VM. Therefore it is advised especially in production environments that SSH keys are used in order for it to be accessible. You can define the SSH keys that are given to the default user in the cloud image.

I use a `variables.tf` to hold all of my input variables since my network is quite small. You can think of these variables are parameters of a function where you list the required and optional inputs for your module. The actual arguments that you parse into this function is defined in `terraform.tfvars` and is ignored by git.

Modules are like functions in programming. You define inputs and outputs to these modules. I defined a module called `vm` which holds a reusable module that I can just parse in arguments into to get different results.

In the end the folder structure looks like this. In the modules folder you can specify other modules like LXC containers and cloud_image resources. There is nothing in my `outputs.tf` file at the moment but that file is very useful if you want to pipe data into Ansible later.

```
.
├── keys
│   └── id_ed25519.pub
├── main.tf
├── modules
│   └── vm
│       ├── main.tf
│       ├── outputs.tf
│       └── variables.tf
├── outputs.tf
├── providers.tf
├── terraform.tfvars
└── variables.tf
```

### Ansible
I am very familiar with using docker compose to deploy my services. I use it in software development sprints with team members to quickly deploy reproducible environments. Therefore as a result of this high degree of familiarity I use Ansible and docker compose to start services on my virtual machines.

I house all of my docker compose files in a `compose_files` folder. Each compose file is housed in it's own folder as the service might need other files in order to function.

The next folder named `inventory` houses the `inventory.ini` file which specifies the variables and destination of my virtual machines. So when running a playbook it knows where to execute the functions.

`playbooks` stores my playbooks and roles. Roles can be thought as reusable playbooks, so anything that it done across all virtual machines I create a role for it. This allows for the codebase to remain small and to encourage modularity. 

Depending on the services needed a list is provided to the Docker role telling Ansible what compose folders are needed. This gets uploaded to the Ansible host and ran as a daemon. Because docker specifies the folders needed and networks the containers communicate over setup is kept to a minimum with Ansible.

## Further Improvements
There are a fair amount of improvements I want to make, especially with Ansible as there are some glaring issues that I am facing currently.

### Netbird VPN
Netbird is the VPN provider that I want to deploy on my home lab, though, installation using docker is quite unique as it requires you to run a installation script to generate the necessary files to use docker compose. This means I have to make an exception to Netbird and how it is deployed by creating a special Ansible role for it and I cannot edit the compose file and re-upload it causing a lot of friction.

### Grouping Compose Files in the Virtual Machine
Another improvement I want to make is to group folders in respect to what virtual machine they run on. So `wazuh` would go in a management folder, `minecraft-server` would go in production etc. I believe this is very important as sometimes I have to run traefik on two virtual machines but the configuration is different.

### Moving `debian_cloud_image` to their own Module
Currently the `debian_cloud_image` resource that is used to pull the Debian cloud image from the internet sits inside the `vm` module. This means that every time a new virtual machine is created the cloud image is pulled from the internet. This means if you started 100 virtual machines you would end up downloading the cloud image 99 times more than needed.

The `debian_cloud_image` resource should be parsed into the `vm` module instead. This would significantly reduce the time it takes to create all the virtual machines. Even with just 5 virtual machines it takes around 2 minutes from memory.

## Conclusion
Ansible and OpenTofu has been instrumental in allowing me to change my infrastructure very quickly. It also allows me to treat it as code which has been instrumental in maintaining stuff and things like that.

I am still learning the quirks of both technologies and my repository is changing constantly as I slowly figure out different ways of approaching the same problem.