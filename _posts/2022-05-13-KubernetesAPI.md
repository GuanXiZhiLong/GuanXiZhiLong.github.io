---
title: "K8SAPI文档"
excerpt: "K8SAPI文档"
last_modified_at: 2022-05-13T11:46:06-12:00
header:
tags:
- 虚拟化
- 容器技术
toc: true
---

## The Kubernetes API
Kubernetes的control plane(容器编排层，它公开API和接口，以定义、部署和管理容器的生命周期)核心是API服务器。API服务器公开一个HTTP API，允许终端用户、集群的不同部分和外部组件相互通信。

Kubernetes API允许您查询和操作Kubernetes中的API对象的状态(例如:Pods、Namespaces、ConfigMaps和Events)。

大多数操作可以通过kubectl命令行界面或其他命令行工具(如kubeadm)执行，这些工具反过来使用API。但是，您也可以使用REST调用直接访问API。

如果您正在使用Kubernetes API编写应用程序，请考虑使用其中一个client库。

## OpenAPI 规范
完整的API细节是由官方文档提供[openAPI]https://www.openapis.org/生成。 

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: minimal-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx-example
      rules:
      - http:
          paths:
          - path: /testpath
            pathType: Prefix
            backend:
              service:
                name: test
                port:
                  number: 80

 
    