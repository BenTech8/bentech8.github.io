---
title: argo-events
date: 2023-03-14 16:01:39
tags:
- Kubernetes
categories: 
- Kubernetes
---

## Argo Events概述

在kubernetes集群中，工作流程的触发通常依赖于特定事件的发生，而Argo Events则提供了一种强大的方式来监控外部事件，并将这些事件映射到集群内的操作或任务执行。Argo Events是Argo项目中的一部分，它与Argo Workflows一起使用，能够在事件发生时启动工作流，形成自动化的DevOps流程。

Argo Events是一种事件驱动的工作流程自动化工具。它允许Kubernetes集群根据外部或内部事件触发工作流程。这些事件可以是来自外部系统的HTTP请求、GitHub Webhook、Kafka消息、S3对象创建，或者Kubernetes内部资源的变化。通过Argo Events，我们可以定义触发器(Trigger)，并将它们与事件源(Event Source)关联，从而在事件发生时执行相应的操作，比如启动一个工作流。



## 核心概念

在理解Argo Events如何触发Kubernetes工作流之前，需要熟悉几个核心组件：

- Event Source：这是事件的来源，可以是HTTP、Webhook、定时器(Cron)、消息队列(如Kafka)、对象存储(如S3)等。
- Sensor：传感器负责监听Event Source中发生的事件，并在符合条件时触发相应的操作。Sensor会将事件处理为一个或多个触发器(Trigger)。
- EventBus：通过Event Source和Sensor来充当Argo-Events的传输层。EventBus有三种实现：NATS(已弃用)、Jetstream和Kafka。
- Trigger：触发器是事件到达时执行的操作。Trigger可以是Kubernetes中的任何资源，比如启动工作流(Workflow)，执行Pod，或者触发其他Kubernetes原生资源。

这三个组件构成了Argo Events的核心架构，使得它能够实现事件驱动的工作流自动化。



## 具体工作流程

### 安装Argo Workflows

```shell
export ARGO_WORKFLOWS_VERSION=3.5.4
kubectl create namespace argo
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/download/v$ARGO_WORKFLOWS_VERSION/install.yaml
```



### 安装Argo Events

```shell
kubectl create namespace argo-events
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml
# Install with a validating admission controller
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install-validating-webhook.yaml
```



### 运行eventbus pod

```shell
kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
```



### 配置Event Source

通过定义一个EventSource来指定事件的来源。假设通过HTTP Webhook来触发工作流，可以定义如下的EventSource资源：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventSource
metadata:
  name: webhook
spec:
  service:
    ports:
      - port: 12000
        targetPort: 12000
    metadata:
      labels:
        foo: bar
      annotations:
        foo: bar
  webhook:
    # event-source can run multiple HTTP servers. Simply define a unique port to start a new HTTP server
    example:
      # port to run HTTP server on
      port: "12000"
      # endpoint to listen to
      endpoint: /example
      # HTTP request method to allow. In this case, only POST requests are accepted
      method: POST

#    example-foo:
#      port: "12000"
#      endpoint: /example2
#      method: POST

# Uncomment to use secure webhook
#    example-secure:
#      port: "13000"
#      endpoint: "/secure"
#      method: "POST"
#      # k8s secret that contains the cert
#      serverCertSecret:
#        name: my-secret
#        key: cert-key
#      # k8s secret that contains the private key
#      serverKeySecret:
#        name: my-secret
#        key: pk-key
```

在这个例子中，我们定义了一个名为webhook的事件源，它通过一个HTTP POST请求触发。在实际应用中，可以将这个事件源与外部系统(如GitHub或Jenkins)集成，用以响应事件。



### 配置Sensor

事件源只是监听事件的入口，接下来我们需要通过Sensor将事件与触发操作关联。Sensor负责捕捉事件并触发工作流。

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Sensor
metadata:
  name: webhook
spec:
  template:
    serviceAccountName: operate-workflow-sa
  dependencies:
    - name: test-dep
      eventSourceName: webhook
      eventName: example
  triggers:
    - template:
        name: webhook-workflow-trigger
        k8s:
          operation: create
          source:
            resource:
              apiVersion: argoproj.io/v1alpha1
              kind: Workflow
              metadata:
                generateName: webhook-
              spec:
                entrypoint: whalesay
                arguments:
                  parameters:
                  - name: message
                    # the value will get overridden by event payload from test-dep
                    value: hello world
                templates:
                - name: whalesay
                  inputs:
                    parameters:
                    - name: message
                  container:
                    image: docker/whalesay:latest
                    command: [cowsay]
                    args: ["{{inputs.parameters.message}}"]
          parameters:
            - src:
                dependencyName: test-dep
                dataKey: body
              dest: spec.arguments.parameters.0.value
```



### 暴露Event-source pod

```shell
kubectl -n argo-events port-forward svc/webhook-eventsource-svc 12000:12000
```



### 触发工作流

```shell
curl -d '{"message":"this is my first webhook"}' -H "Content-Type: application/json" -X POST http://localhost:12000/example
```

当外部系统(比如GitHub或Jenkins)向EventSource指定的Webhook发送HTTP请求时，Sensor将捕获事件并启动工作流。此时，一个新的工作流将在Kubernetes集群中运行，并执行定义好的任务。



### 验证工作流

```shell
kubectl -n argo-events get workflows | grep "webhook"
```



## 实际应用案例

在现实的企业DevOps场景中，Argo Events常用于自动化CI/CD流程。例如，某公司利用Argo Events和GitHub Webhook集成，实现代码提交时自动触发Kubernetes集群中的测试工作流。

具体场景如下：

- 开发人员在GitHub上向主分支推送代码。
- GitHub的Webhook向Kubernetes集群发送POST请求。
- EventSource接收到Webhook请求，并将事件传递给Sensor。
- Sensor捕获事件并启动一个CI工作流。
- 工作流运行集成测试，验证代码的正确性。
- 测试成功后，工作流自动生成新的Docker镜像并推送到容器注册中心。

这一流程实现了完全自动化的持续集成，不仅减少了手动操作的时间，也提高了代码交付的质量和速度。



## 事件过滤与复杂场景

在复杂的场景中，往往需要对事件进行过滤和处理。Argo Events支持多种条件来控制何时触发工作流。比如，可以基于事件的内容、时间或其他自定义条件进行过滤。

假设我们只想在特定的时间范围内(比如工作日的9点到18点)触发工作流，我们可以在Sensor中定义时间窗口来限制事件的触发。也可以基于Webhook的数据内容来判断是否执行工作流，进一步增强自动化的灵活性。

```yaml
spec:
  filters:
	time:
	  schedule: "0 9 * * 1-5"
```

这个配置只允许在工作日的9点之后触发工作流，确保在合适的时间内进行自动化操作。



## 安全性与扩展性

为了确保系统的安全性，在使用Argo Events时通常会对事件源和传感器进行身份验证和授权。可以在Webhook事件源中增加TLS配置，确保传输安全，或者通过认证机制限制特定来源的请求。

在扩展性方面，Argo Events能够支持大量并发事件。得益于Kubernetes的弹性架构，多个EventSource和Sensor可以并行运行，处理大在规模的事件流。这使得它特别适用于具有复杂事件流和大规模集群的环境，比如金融交易系统、物联网平台等。



## 链接

- 官方文档：https://argoproj.github.io/argo-events/
