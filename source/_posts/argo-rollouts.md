---
title: argo-rollouts
date: 2025-03-14 15:39:17
tags:
categories: Kubernetes
---

## Argo Rollouts简介

Argo Rollouts是一个Kubernetes Operator实现，它为Kubernetes提供更加高级的部署能力，如蓝绿、金丝雀、金丝雀分析、实验和渐进式交付功能，为云原生应用和服务实现自动化、基于GitOps的逐步交付。

支持如下特性：

- 蓝绿更新策略
- 金丝雀更新策略
- 更加细粒度、加权流量拆分
- 自动回滚
- 手动判断
- 可定制的指标查询和业务KPI分析
- Ingress控制器集成：NGINX，ALB
- Metrics指标集成：Prometheus、Wavefront、Kayenta、Web、Kubernetes Jobs、Datadog、New Relic、Graphite、InfluxDB



## 实现原理

与Deployment对象类似，Argo Rollout控制器将管理ReplicaSets的创建、缩放和删除，这些ReplicaSet由Rollout资源中的spec.template定义，使用与Deployment对象相同的pod模板。

当spec.template变更时，这会向Argo Rollouts控制器发出信号，表示将引入新的ReplicaSet，控制器将使用spec.strategy字段内的策略来确定从旧ReplicaSet到新ReplicaSet的rollout将如何进行，一旦这个新的ReplicaSet被放大(可以选择通过一个Analysis)，控制器会将其标记为稳定。

如果在spec.template从稳定的ReplicaSet过渡到新的ReplicaSet的过程中发生另一次变更(即在发布过程中更改了应用程序版本)，那么之前的新ReplicaSet将缩小，并且控制器将尝试反映更新spec.template字段的ReplicaSet。
