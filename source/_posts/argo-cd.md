---
title: argo-cd
date: 2025-03-14 11:32:37
tags:
categories: DevOps
---

## ArgoCD简介

Argo CD是一个基于Kubernetes的声明式的GitOps工具。



### GitOps

GitOps是以Git为基础，使用CI/CD来更新运行在云原生环境的应用，它秉承了DevOps的核心理念——"构建它并交付它(you built it you ship it)"。

当开发人员将开发完成的代码推送到git仓库会触发CI制作镜像并推送到镜像仓库。

CI处理完成后，可以手动或者自动修改应用配置，再将其推送到git仓库。

GitOps会同时对比目标状态和当前状态，如果两者不一致会触发CD将新的配置部署到集群中。

其中，目标状态是Git中的状态，现有状态是集群里的应用状态。



### ArgoCD

Argo CD 遵循GitOps模式，使用Git存储库存储所需应用程序的配置。

Kubernetes清单可以通过以下几种方式指定：

- kustomize
- helm
- ksonnet
- jsonnet
- 基于YAML/json
- 配置管理插件配置的任何自定义配置管理工具

Argo CD实现为kubernetes控制器，它持续监视运行中的应用程序，并将当前的活动状态与期望的目标状态进行比较(如Git repo中指定的那样)。如果已部署的应用程序的活动状态偏离了目标状态，则认为是OutOfSync。Argo CD报告和可视化这些差异，同时提供了方法，可以自动或手动将活动状态同步回所需的目录状态。在Git repo中对所需目标状态所做的任何修改都可以自动应用并反映到指定的目标环境中。



### ArgoCD优势

- 应用定义、配置和环境信息是声明式的，并且可以进行版本控制。
- 应用部署和生命周期管理是全自动化的，是可审计的，清晰易懂。
- Argo CD是一个独立的部署工具，支持对多个环境、多个Kubernetes集群上的应用进行统一部署和管理。



## 链接

- 官方文档：https://argo-cd.readthedocs.io/en/stable/
