+++
title = "Cloud Supremacy, Pt. 1"
subtitle = "My experiences deploying a Hugo website on Kubernetes using RKE2 and Rancher"
date = "2022-01-05"
author = "Julio Hawthorne"
banner = "img/rancher.png"
description = "Cloud Supremacy, Pt. 1: Introduction"
tags = ["kubernetes"]
keywords = ["guide", "rancher", "kubernetes", "gcp", "terraform", "rke2"]
+++

## Preface
---
After spending a year working on SUSE's Rancher Support Team, I've decided to start sharing what I've learned with a broader audience. The explanations and components in this series of articles are oriented toward those of an intermediate skill level. The design approach allows anyone with a background in tech to get their feet wet with Cloud-Native technologies. All source code created for this project will be held in the [Cloud Supremacy Project](https://github.com/VltraHeaven/cloud-supremacy-project) on Github.

<br>

## Design and Components
---
I'll leverage [Terraform](https://developer.hashicorp.com/terraform/intro) and [Terragrunt](https://terragrunt.gruntwork.io/docs/getting-started/quick-start/) throughout this project, wherever practical. This way, all referenced resources will be clearly defined as code and adaptable to various use cases. The choice to make [Hugo](https://gohugo.io/about/what-is-hugo/) my deployment target was based on my familiarity with the Static-Site Generator and the framework's nature of generating user-facing websites. This decision will make this series even more adaptable to other web app-oriented use cases. [Google Cloud Platform](https://cloud.google.com/) is my defacto cloud hosting provider; thus, my Terraform will specifically target this provider's modules. While using [Rancher](https://ranchermanager.docs.rancher.com/) is not explicitly necessary for this project, it provides several robust [monitoring](https://ranchermanager.docs.rancher.com/pages-for-subheaders/monitoring-v2-configuration) and [Continuous Delivery](https://ranchermanager.docs.rancher.com/pages-for-subheaders/fleet-gitops-at-scale) utilities that makes further management of deployments much more accessible to the everyday user. 

![Architecture Diagram](img/cloud-supremacy-diagram.png)