---
title: argo-workflows
date: 2023-03-14 16:01:29
tags: 
- Kubernetes
categories: 
- Kubernetes
---

## Argo Workflows简介

Argo Workflows是一个开源项目，为Kubernetes提供container-native工作流程，其主要通过Kubernetes CRD实现的。

特点如下：

- 工作流的每一步都是一个容器
- 将多步骤工作流建模为一系列任务，或者使用有向无环图（DAG）描述任务之间的依赖关系
- 可以在短时间内轻松运行用于机器学习或数据处理的计算密集型作业。
- 在Kubernetes上运行CI/CD Pipeline，无需复杂的软件配置



## 安装

### 安装Argo Workflows

```shell
ARGO_WORKFLOWS_VERSION="v3.6.5"
kubectl create namespace argo
kubectl apply -n argo -f "https://github.com/argoproj/argo-workflows/releases/download/${ARGO_WORKFLOWS_VERSION}/quick-start-minimal.yaml"
```



###  安装Argo Workflows CLI

Mac/Linux

```shell
# Detect OS
ARGO_OS="darwin"
if [[ uname -s != "Darwin" ]]; then
  ARGO_OS="linux"
fi

# Download the binary
curl -sLO "https://github.com/argoproj/argo-workflows/releases/download/v3.6.5/argo-$ARGO_OS-amd64.gz"

# Unzip
gunzip "argo-$ARGO_OS-amd64.gz"

# Make binary executable
chmod +x "argo-$ARGO_OS-amd64"

# Move binary to path
mv "./argo-$ARGO_OS-amd64" /usr/local/bin/argo

# Test installation
argo version
```

argo命令

```shell
# 提交一个workflow至kubernetes
argo submit hello-world.yaml

# 列出当前所有workflows
argo list

# 获取某个特定workflow的信息
argo get hello-world-xxx

# 打印某个workflow的日志
argo logs hello-world-xxx

# 删除workflow
argo delete hello-world-xxx
```



## 核心概念

### Workflow

Workflow是Argo中最重要的资源，其主要有两个重要功能：

- 它定义要执行的工作流
- 它存储工作流的状态

要执行的工作流定义在Workflow.spec字段中，其主要包括templates和entrypoint，如：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-
  labels:
    workflows.argoproj.io/archive-strategy: "false"
  annotations:
    workflows.argoproj.io/description: |
      This is a simple hello world example.
spec:
  entrypoint: hello-world          # 工作流的入口点，即工作流启动时首先执行的模板
  templates:                       # 模板
  - name: hello-world
    container:
      image: busybox
      command: [echo]
      args: ["hello world"]
```



### Templates

templates是列表结构，主要分为两类：

- 定义具体的工作流
- 调用其他模板提供并行控制



#### 定义具体的工作流

定义具体的工作流有4种类别，如下：

- Container
- Script
- Resource
- Suspend



**Container**

container是最常见的模板类型，它将调度一个container，其模板规范和K8S的容器规范相同，如下：

```yaml
- name: hello-world
  container:
    image: busybox
    command: [echo]
    args: ["hello world"]
```



**Script**

Script是Container的另一种包装实现，其定义方式和Container相同，只是增加了source字段用于自定义脚本，如下：

```yaml
- name: gen-random-int
  script:
    image: python:alpine3.6
    command: [python]
    source: |
      import random
      i = random.randint(1, 100)
      print(i)
```

脚本的输出结果会根据调用方式自动导出到tasks.{NAME}.outputs.result或steps.{NAME}.outputs.result中。



**Resource**

Resource主要用于直接在K8S集群上执行集群资源操作，可以get,create,apply,delete,replace,patch集群资源。如下在集群中创建一个ConfigMap类型资源：

```yaml
- name: k8s-owner-reference
  resource:
    action: create
    manifest: |
      apiVersion: v1
      kind: ConfigMap
      metadata:
        generateName: owned-eg-
      data:
        some: value
```



**Suspend**

Suspend主要用于暂停，可以暂停一段时间，也可以手动恢复，命令使用argo resume进行恢复。定义格式如下：

```yaml
- name: delay
  suspend:
    duration: "20s"
```



#### 调用其他模板提供并行控制

调用其他模板也有两种类别：

- Steps
- Dag



**Steps**

Steps主要是通过定义一系列步骤来定义任务，其结构是“list of lists”，外部列表将顺序进行，内部列表将并行执行。如下：

```yaml
- name: hello-hello-hello
  steps:
  - - name: step1
      template: prepare-data
  - - name: step2a
      template: run-data-first-half
    - name: step2b
      template: run-data-second-half
```

其中step1和step2是顺序执行，而step2a和step2b是并行执行。

还可以通过when来进行条件判断。如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: coinflip-
spec:
  entrypoint: coinflip
  templates:
  - name: coinflip
    steps:
    - - name: flip-coin
        template: flip-coin
    - - name: heads
        template: heads
        when: "{{steps.flip-coin.outputs.result}} == heads"
      - name: tails
        template: tails
        when: "{{steps.flip-coin.outputs.result}} == tails"

  - name: flip-coin
    script:
      image: python:alpine3.6
      command: [python]
      source: |
        import random
        result = "heads" if random.randint(0,1) == 0 else "tails"
        print(result)

  - name: heads
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was heads\""]

  - name: tails
    container:
      image: alpine:3.6
      command: [sh, -c]
      args: ["echo \"it was tails\""]
```

除了使用when进行条件判断，还可以进行循环操作，示例代码如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: loops-
spec:
  entrypoint: loop-example
  templates:
  - name: loop-example
    steps:
    - - name: print-message
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "{{item}}"
        withItems:
        - hello world
        - goodbye world

  - name: whalesay
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
```



**Dag**

Dag主要用于定义任务的依赖关系，可以设置开始特定任务之前必须完成其他任务，没有任何依赖关系的任务将立即执行。示例代码如下：

```yaml
- name: diamond
  dag:
    tasks:
    - name: A
      template: echo
    - name: B
      dependencies: [A]
      template: echo
    - name: C
      dependencies: [A]
      template: echo
    - name: D
      dependencies: [B, C]
      template: echo
```

其中A会立即执行，B和C会依赖A，D依赖B和C。



### Variables

在argo的Workflow中允许使用变量，示例代码如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-parameters-
spec:
  entrypoint: whalesay
  arguments:
    parameters:
      - name: message
        value: hello world
  templates:
    - name: whalesay
      inputs:
        parameters:
          - name: message
      container:
        image: docker/whalesay
        command: [ cowsay ]
        args: [ "{{inputs.parameters.message}}" ] 
```

首先在spec字段定义arguments，定义变量message，其值是hello world，然后在templates字段中需要先定义一个inputs字段，用于templates的输入参数，然后再使用双大括号形式引用变量。

变量还可以进行一些函数运算，主要有：

- filter: 过滤
- asInt: 转换为Int
- asFloat: 转换为float
- string: 转换为String
- toJson: 转换为Json

例如：

```
filter([1, 2], { # > 1})
asInt(inputs.parameters["my-int-param"])
asFloat(inputs.parameters["my-float-param"])
string(1)
toJson([1, 2])
```



### 制品库

在安装argo的时候，已经安装了minio作为制品库，那么到底该如何使用呢？

示例代码如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: artifact-passing-
spec:
  entrypoint: artifact-example
  templates:
  - name: artifact-example
    steps:
    - - name: generate-artifact
        template: whalesay
    - - name: consume-artifact
        template: print-message
        arguments:
          artifacts:
          - name: message
            from: "{{steps.generate-artifact.outputs.artifacts.hello-art}}"

  - name: whalesay
    container:
      image: docker/whalesay:latest
      command: [sh, -c]
      args: ["sleep 1; cowsay hello world | tee /tmp/hello_world.txt"]
    outputs:
      artifacts:
      - name: hello-art
        path: /tmp/hello_world.txt

  - name: print-message
    inputs:
      artifacts:
      - name: message
        path: /tmp/message
    container:
      image: alpine:latest
      command: [sh, -c]
      args: ["cat /tmp/message"]
```

其分为两步：

- 首先生成制品
- 然后获取制品



### WorkflowTemplate

WorkflowTemplate是Workflow的模板，可以从WorkflowTemplate内部或集群上其他Workflow和WorkflowTemplate引用它们。

WorkflowTemplate和template的区别：

- template只是Workflow中template下的一个任务，当我们定义一个Workflow时，至少需要定义一个template
- WorkflowTemplate是驻留在集群中的Workflow的定义，它是Workflow的定义，因为它包含模板，可以从WorkflowTemplate内部或集群上其他Workflow和WorkflowTemplate引用它们。

示例代码如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: workflow-template-1
spec:
  entrypoint: whalesay-template
  arguments:
    parameters:
      - name: message
        value: hello world
  templates:
    - name: whalesay-template
      inputs:
        parameters:
          - name: message
      container:
        image: docker/whalesay
        command: [cowsay]
        args: ["{{inputs.parameters.message}}"]
```

创建WorkflowTemplate，如下：

```shell
argo template create workflowtemplate.yaml
```

在Workflow中引用，如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: workflow-template-hello-world-
spec:
  entrypoint: whalesay
  templates:
  - name: whalesay
    steps:                              # 引用模板必须在steps/dag/template下
      - - name: call-whalesay-template
          templateRef:                  # 应用模板字段
            name: workflow-template-1   # WorkflowTemplate名
            template: whalesay-template # 具体的template名
          arguments:                    # 参数
            parameters:
            - name: message
              value: "hello world"
```



### ClusterWorkflowTemplate

ClusterWorkflowTemplate创建的是一个集群范围内的WorkflowTemplate，其他workflow可以引用它。

示例代码如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ClusterWorkflowTemplate
metadata:
  name: cluster-workflow-template-whalesay-template
spec:
  templates:
  - name: whalesay-template
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
```

在workflow中使用templateRef去引用它，如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: workflow-template-hello-world-
spec:
  entrypoint: whalesay
  templates:
  - name: whalesay
    steps:          
      - - name: call-whalesay-template
          templateRef:                  #引用模板
            name: cluster-workflow-template-whalesay-template   # ClusterWorkflow名
            template: whalesay-template # 具体的模板名
            clusterScope: true          # 表示是ClusterWorkflow
          arguments:                    #  参数
            parameters:
            - name: message
              value: "hello world"
```



## 实践

以简单的CI/CD流程实践，来实践argo workflow应该如何做。

CI/CD的整个流程很简单，即：拉代码--->编译--->构建镜像--->上传镜像--->部署。

定义一个workflowTemplate，如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  annotations:
    workflows.argoproj.io/description: |
      Checkout out from Git, build and deploy application.
    workflows.argoproj.io/maintainer: '@joker'
    workflows.argoproj.io/tags: java, git
    workflows.argoproj.io/version: '>= 2.9.0'
  name: devops-java 
spec:
  entrypoint: main
  arguments:
    parameters:
      - name: repo
        value: gitlab-test.coolops.cn:32080/root/springboot-helloworld.git
      - name: branch
        value: master
      - name: image
        value: registry.cn-hangzhou.aliyuncs.com/rookieops/myapp:202103101613
      - name: cache-image
        value: registry.cn-hangzhou.aliyuncs.com/rookieops/myapp
      - name: dockerfile
        value: Dockerfile
      - name: devops-cd-repo
        value: gitlab-test.coolops.cn:32080/root/devops-cd.git
      - name: gitlabUsername
        value: devops-argo
      - name: gitlabPassword
        value: devops123456
  templates:
    - name: main
      steps:
        - - name: Checkout
            template: Checkout
        - - name: Build
            template: Build
        - - name: BuildImage
            template: BuildImage
        - - name: Deploy
            template: Deploy
    # 拉取代码
    - name: Checkout
      script:
        image: registry.cn-hangzhou.aliyuncs.com/rookieops/maven:3.5.0-alpine
        workingDir: /work
        command:
        - sh
        source: |
          git clone --branch {{workflow.parameters.branch}} http://{{workflow.parameters.gitlabUsername}}:{{workflow.parameters.gitlabPassword}}@{{workflow.parameters.repo}} .
        volumeMounts:
          - mountPath: /work
            name: work
    # 编译打包  
    - name: Build
      script:
        image: registry.cn-hangzhou.aliyuncs.com/rookieops/maven:3.5.0-alpine
        workingDir: /work
        command:
        - sh
        source: mvn -B clean package -Dmaven.test.skip=true -Dautoconfig.skip
        volumeMounts:
          - mountPath: /work
            name: work
    # 构建镜像  
    - name: BuildImage
      volumes:
      - name: docker-config
        secret:
          secretName: docker-config
      container:
        image: registry.cn-hangzhou.aliyuncs.com/rookieops/kaniko-executor:v1.5.0
        workingDir: /work
        args:
          - --context=.
          - --dockerfile={{workflow.parameters.dockerfile}}
          - --destination={{workflow.parameters.image}}
          - --skip-tls-verify
          - --reproducible
          - --cache=true
          - --cache-repo={{workflow.parameters.cache-image}}
        volumeMounts:
          - mountPath: /work
            name: work
          - name: docker-config
            mountPath: /kaniko/.docker/
    # 部署  
    - name: Deploy
      script:
        image: registry.cn-hangzhou.aliyuncs.com/rookieops/kustomize:v3.8.1
        workingDir: /work
        command:
        - sh
        source: |
           git remote set-url origin http://{{workflow.parameters.gitlabUsername}}:{{workflow.parameters.gitlabPassword}}@{{workflow.parameters.devops-cd-repo}}
           git config --global user.name "Administrator"
           git config --global user.email "coolops@163.com"
           git clone http://{{workflow.parameters.gitlabUsername}}:{{workflow.parameters.gitlabPassword}}@{{workflow.parameters.devops-cd-repo}} /work/devops-cd
           cd /work/devops-cd
           git pull
           cd /work/devops-cd/devops-simple-java
           kustomize edit set image {{workflow.parameters.image}}
           git commit -am 'image update'
           git push origin master
        volumeMounts:
          - mountPath: /work
            name: work
  volumeClaimTemplates:
    - name: work
      metadata:
        name: work
      spec:
        storageClassName: nfs-client-storageclass
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi 
```

使用kaniko来创建镜像，不用挂载docker.sock，但是push镜像的时候需要config.json，所以首先需要创建一个secret，如下：

```yaml
kubectl create secret generic docker-config --from-file=.docker/config.json -n argo
```

创建WorkflowTemplate，如下：

````yaml
argo template create -n argo devops-java.yaml
````

创建Workflow，可以手动创建，如下：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: workflow-template-devops-java-
spec:
  workflowTemplateRef:
    name: devops-java
```



## 链接

官方文档：https://argo-workflows.readthedocs.io/en/latest/
