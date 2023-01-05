+++
title = "Awesome DevOps"
date = 2021-03-04
author = "Julio Hawthorne"
banner = "img/empire.gif"
description = "Reference list of common DevOps tools and terminology."
tags = ["devops"]
keywords = ["devops", "awesomelist", "linux", "kubernetes", "programming", "cloud native"]
+++

## Terminology

---

- DevOps - A set of practices that combine IT and Software Engineering practices that assist in reducing the time
  required to reliably engineer and deploy code to production environments.
- Version Control - The process of tracking and managing the incremental changes of source code and configuration files, 
  usually handled with various tools and utilities.
- Continuous Integration (CI) - The practice of merging the distributed copies of a development team's codebase into a 
  centralized repository, usually several times a day.
- Continuous Delivery (CD) - The cycle of creating, building, testing and manually releasing software at a high
  frequency and with consistency.
- Continuous Deployment (also CD) - Almost identical to the Continuous Delivery process but with automated releases
  instead of manual releases.
- Infrastructure - A collection of logically architected systems and network resources that act as the communication
  backbone between users and services.
- Infrastructure as Code - The process of designing and engineering infrastructure using code within configuration
  files and running deployments using software tools.
- Configuration as Code - Defining the configuration of Computer and Network infrastructure using code and configuration
  files with a focus on speed and consistent reproducibility. 
- Namespace - A label assigned to a set of resources and processes. Designed so that processes running in a specific namespace
  can only access other processes, files and resources that reside within that namespace. One process can be assigned several
  namespaces.
- Container - An entire operating system running within a virtual namespace on a host system while sharing that host's
  resources with varying degrees of restrictions. Containers are typically used to run a single target program with minimal
  overhead required from the host.
- Pod - A collection of inter-dependent containers, each running a single target program, that together run as a single
  application.
- Container Image - A compressed, frozen template of a pre-configured operating system. Used as the foundation to 
  create containers.
- Container Orchestration - The process of creating, managing, and monitoring containers through software automation.
  Container Orchestration was founded by Google's Borg project (now Kubernetes).
- Identity Access Management - A system that provides sets of policies, profiles and roles that ensures entities with 
  resource access are assigned the appropriate level of permissions within an environment.
  

## Operating Systems

---
- Alpine Linux
- Fedora CoreOS
- Red Hat CoreOS
- Flatcar Linux
- NixOS

## Programming Languages

---
- Python
- Go
- HCL
- Ruby
- YAML
- Bash
- Powershell

## Source Code Management 

---
- Subversion
- Git
- Gitflow
- Gitea
  
## DevOps Platforms

---
- Github
- Gitlab
- Bitbucket
- Gitkraken
- Azure DevOps
- JFrog

## CI/CD

---
- Jenkins
- Buildbot
- Drone
- Travis
- Circle
- Teamcity



## Infrastructure as Code

---
- Terraform
- Vagrant

## Configuration as Code

---
- Ansible
- Saltstack
- Puppet
- Chef
- Packer

## Container Runtimes

---
- OCI Specifications
- RunC
- ContainerD
- Docker
- Podman/Buildah
- LXC/LXD  
- Cri-O
- Crun
- Kata Containers  
- GVisor
- systemd-nspawn

## Container Orchestration

---
- Kubernetes (K8s)
- OKD
- Openshift
- K3s
- Minikube
- Kind
- MicroK8s
- Managed Kubernetes

## Public Cloud Platforms

---
- Amazon Web Services
- Microsoft Azure
- Google Cloud Platform
- IBM Cloud
- DigitalOcean

## Testing Frameworks

---
- Terratest

## Security

___
- Lynis
- Apparmor
- SELinux
- Vault