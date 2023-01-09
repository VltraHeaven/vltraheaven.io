+++
title = "Cloud Supremacy, Pt. 1"
subtitle = "My experiences deploying a Hugo website to Kubernetes using RKE2 and Rancher"
date = "2022-01-05"
author = "Julio Hawthorne"
banner = "img/rancher.png"
description = "Cloud Supremacy, Pt. 1: Introduction"
tags = ["kubernetes"]
keywords = ["rancher", "kubernetes", "gcp", "terraform", "rke2"]
+++

# Table of Contents
1. [Preface](#preface)
2. [Tools of the Trade](#tools-of-the-trade)
3. [Architectural Design](#architectural-design)

<br>

# Introduction
---
After spending a year working on SUSE's Rancher Support Team, I've decided to start sharing what I've learned with a broader audience. The explanations and components in this series of articles are oriented toward those of an intermediate skill level. The design approach allows anyone with a background in tech to get their feet wet with Cloud-Native technologies. All source code created for this project will be held in the [Cloud Supremacy Project](https://github.com/VltraHeaven/cloud-supremacy-project) on Github.

<br>

# Tools of the Trade
---
I'll leverage [Terraform](https://developer.hashicorp.com/terraform/intro) and [Terragrunt](https://terragrunt.gruntwork.io/docs/getting-started/quick-start/) throughout this project, wherever practical. This way, all referenced resources will be clearly defined as code and adaptable to various use cases. The choice to make [Hugo](https://gohugo.io/about/what-is-hugo/) my deployment target was based on my familiarity with the Static-Site Generator and the framework's nature of generating user-facing websites. This decision will make this series even more adaptable to other web app-oriented use cases. [Google Cloud Platform](https://cloud.google.com/) is my defacto cloud hosting provider; thus, my Terraform will specifically target this provider's modules. Google Cloud Platform also offers a [$300 credit to new users](https://cloud.google.com/free) for anyone looking to get their feet wet in this technology at no initial cost. While using [Rancher](https://ranchermanager.docs.rancher.com/) is not explicitly necessary for this project, it provides several robust [monitoring](https://ranchermanager.docs.rancher.com/pages-for-subheaders/monitoring-v2-configuration) and [Continuous Delivery](https://ranchermanager.docs.rancher.com/pages-for-subheaders/fleet-gitops-at-scale) utilities that makes further management of deployments much more accessible to the everyday user. Lastly, I chose [rke2](https://docs.rke2.io/) as the flavor of Kubernetes for its ease of use, security-conscious default settings, and quality-of-life features it offers over other Kubernetes distributions. 

<br>

# Architectural Design
---
![Architecture Diagram](img/cloud-supremacy-diagram.png)

The diagram above visually describes the infrastructure's design. The rke2 cluster hosting the Rancher Management Server will only have one node for simplicity. Still, we will provision a Jumpbox host to access the Rancher WebUI to avoid exposing the cluster hosting Rancher to the internet. Firewall rules will also be in place to control network access between all hosts and the internet.

This concludes my brief overview of the project. Stay tuned for part 2, where I'll go over getting a development environment setup and begin building out my cloud networking infrastructure.