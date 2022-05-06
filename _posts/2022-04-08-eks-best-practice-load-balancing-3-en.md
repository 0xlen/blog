---
title: "[AWS][EKS] Best practice for load balancing - 3. what controller should I use"
description: "This article is sharing the best practice for doing load balancing on Amazon EKS, learn what is advantage and disadvantage of using different controller. We will discuss what controller should you use."
tags: ['aws', 'amazon web services', 'EC2', 'Elastic Compute Cloud', 'amazon', 'ELB', 'ALB', 'Load Balancer', 'Elastic Load Balancer', 'ALB Ingress Controller', 'Kubernetes', 'k8s', 'EKS', 'Elastic Kubernetes Service', 'AWS Load Balancer Controller']
header:
  og_image: /assets/images/posts/2022/04/eks-best-practice-load-balancing/nginx-ingress-architecture.png
  teaser: /assets/images/posts/2022/04/eks-best-practice-load-balancing/nginx-ingress-architecture.png
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
toc_sticky: true
---

This article is sharing the best practice for doing load balancing on Amazon EKS, learn what is advantage and disadvantage of using different controller. We will discuss what controller should you use.

## Compare different controller options

Here are some common load balancing solutions that can be applied on Amazon EKS:

### Kubernetes in-tree load balancer controller

This is the easiest way to provision your Elastic Load Balancer resource, which could be done by using default Kubernetes service deployment with `type: LoadBalancer`. In most case, the in-tree controller can quickly spin up the load balancer for experiment purpose; or, offers production workload.

However, you need to aware the problem as we mentioned in the previous posts [^eks-best-practice-load-balancing-1] [^eks-best-practice-load-balancing-2] because it generally can add a hop for your load balancing behavior on AWS and also can increase the complexity for your traffic.

In addition, you need to aware this method only applies for creating Classic Load Balancer and Network Load Balancer (by using annotation [^in-tree-aws-nlb-support]).

### nginx ingress controller

If you are using nginx Ingress controller in AWS, it will deploy Network load balancer (NLB) to expose the NGINX Ingress controller behind a Service of `type=LoadBalancer`. Here is an example for deploying Kubernetes service of nginx Ingress controller 1.1.3:


```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.3
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  externalTrafficPolicy: Local
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: LoadBalancer
```

Guess what, yes, it is still can rely on the in-tree controller. On the other hand, the problem we were mentioning can persist. It can be hard to expect which Pods will receive the traffic; however, the main issue is that an Ingress controller does not typically eliminate the need for an external load balancer, it simply adds an additional layer of routing and control behind the load balancer.

{% include figure image_path="/assets/images/posts/2022/04/eks-best-practice-load-balancing/nginx-ingress-architecture.png" alt="An architecture overview of using nginx Ingress controller" caption="Figure 1. An architecture overview of using nginx Ingress controller" %}

So why to choose Nginx Ingress controller? It probably can be the reason why as mentioned in the post [^nlb-nginx-ingress-aws-blog] as mentioned on the AWS Blog:

- By default, the NGINX Ingress controller will listen to all the ingress events from all the namespaces and add corresponding directives and rules into the NGINX configuration file. This makes it possible to use a centralized routing file which includes all the ingress rules, hosts, and paths.
- With the NGINX Ingress controller you can also have multiple ingress objects for multiple environments or namespaces with the same network load balancer.

### AWS Load Balancer Controller

AWS Load Balancer Controller is similar to the in-tree Kubernetes controller and use native AWS APIs to provision and manage Elastic Load Balancers. The controller was an open-source project originally named **ALB Ingress Controller** because it was only provides capability to manage Application Load Balancer at the intial stage, lately, it officially renamed as **AWS Load Balancer Controller** [^intro-aws-load-balancer-controller], which is maintaining by AWS product team and open-source community.

Unlike in-tree Kubernetes controller needs to wait the upstream code to be updated and requires you to upgrade Kubernetes control plane version if the controller has any bug or any new ELB features need to be supported. Using AWS Load Balancer Controller, it can gracefully to be replaced in this scenario because it will be running as Kubernetes deployment instead of integrating in the Kubernetes upstream source code.

The controller has flexibility to maintain your Elastic Load Balancer resources with up-to-date annotations. For example, if you are nginx ingress controller in order to provide Kubernetes Ingress object management, it requires you to provision and add an extra load balancing layer with Network Load Balancer, the traffic generally pass through the controller itself (nginx); instead, if you are using AWS Load Balancer, it doesn't involve and it will directly control the Elastic Load Balancer and pass the traffic to your Pods (by using IP mode).

The AWS Load Balancer Controller also start to support TargetGroupBinding [^aws-lb-controller-targetgroupbinding] and IngressGroup [^aws-lb-controller-ingressgroup] feature since v2.2, which enables you to group multiple Ingress resources together. This also enhance the capability to shared the same Elastic Load Balancer resource.

## Conclusion: What controller should I use?

After comparing different load balancer controllers, generally speaking, using **AWS Load Balancer** basically can have better feature supports as well as adopt with the performance optimization by configuring AWS Load Balancer attributes correctly. It is essential to enable **IP mode** when applying the Kubernetes service deployment with AWS Load Balancer Controller to reduce unnecessary hop that can be caused by Kubernetes networking itself, which is generally not totally suitable for AWS networking and elastic load balancing feature.

However, the disadvantage of using AWS Load Balancer can be all features require to be supported by Elastic Load Balancer itself because the controller doesn't involve additional functions to extend the traffic control. Using other controller still can have its benefit and provide different features that Elastic Load Balancer doesn't have, such as using nginx Ingress controller you may be able to define forward service to external FastCGI targets, using Regular Expression to perform path matching ... etc.

By the end of this article, I hope the comparison and information can better help you understand how to select load balancer controller that will be running in Amazon EKS, and choose the right option for your environment.

Thanks for reading! If you have any feedback or opinions, please feel free to leave the comment below.

{% include_relative related-post/eks-best-practice-load-balancing-en.md %}

## References

[^in-tree-aws-nlb-support]: in-tree controller - [Network Load Balancer support on AWS](https://kubernetes.io/docs/concepts/services-networking/service/#aws-nlb-support)
[^eks-best-practice-load-balancing-1]: [[AWS][EKS] Best pratice load balancing - Let's start with an example from Kubernetes document](/eks-best-practice-load-balancing-1-en)
[^eks-best-practice-load-balancing-2]: [[AWS][EKS] Best pratice load balancing - imbalanced problem](/eks-best-practice-load-balancing-2-en)
[^nginx-ingress-aws]: nginx ingress controller - [Installation Guide - AWS](https://kubernetes.github.io/ingress-nginx/deploy/#aws)
[^nlb-nginx-ingress-aws-blog]: [Using a Network Load Balancer with the NGINX Ingress Controller on Amazon EKS](https://aws.amazon.com/blogs/opensource/network-load-balancer-nginx-ingress-controller-eks/)
[^intro-aws-load-balancer-controller]: [Introducing the AWS Load Balancer Controller](https://aws.amazon.com/blogs/containers/introducing-aws-load-balancer-controller/)
[^aws-lb-controller-targetgroupbinding]: AWS Load Balancer controller v2.2 - [TargetGroupBinding](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/targetgroupbinding/targetgroupbinding/)
[^aws-lb-controller-ingressgroup]: AWS Load Balancer controller v2.2 - [IngressGroup](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/#group.name)
