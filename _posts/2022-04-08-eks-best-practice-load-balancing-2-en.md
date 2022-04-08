---
title: "[AWS][EKS] Best practice for load balancing - 2. imbalanced problem"
description: "This article is sharing the best practice for doing load balancing on Amazon EKS, learn what is advantage and disadvantage of using different controller. We will discuss more detail about the imbalanced problem after applying controller to deploy the Elastic Load Balancer."
tags: ['aws', 'amazon web services', 'EC2', 'Elastic Compute Cloud', 'amazon', 'ELB', 'ALB', 'Load Balancer', 'Elastic Load Balancer', 'ALB Ingress Controller', 'Kubernetes', 'k8s', 'EKS', 'Elastic Kubernetes Service', 'AWS Load Balancer Controller']
header:
  og_image: /assets/images/posts/2022/04/eks-best-practice-load-balancing/ip-target.png
  teaser: /assets/images/posts/2022/04/eks-best-practice-load-balancing/ip-target.png
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
toc_sticky: true
---

This article is sharing the best practice for doing load balancing on Amazon EKS, learn what is advantage and disadvantage of using different controller. We will discuss more detail about the imbalanced problem after applying controller to deploy the Elastic Load Balancer.

## The load imbalanced problem

Follow the example as mentioned in the previous article, if you deployed a Kubernetes service and noticed the utilization on your backend application is not balanced; or, if you are using AWS Load Balancer controller, Traefik, nginx-ingress controller by finding the Elastic Load Balancer wasn't correctly separate the loads (when using instance mode to register your Pods as targets), and you may find the imbalanced traffic, that's the major topic in this article would like to talk about: discuss how to improve and optimize it.

### Problem description

Let's say if I am deploying 4 Pods in my Kubernetes cluster, which is using the default deployment as mentioned below to expose my Kubernetes service:

```bash
$ kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP
nginx-deployment-594764c789-5s668   1/1     Running   0          30m   192.168.42.171
nginx-deployment-594764c789-9k949   1/1     Running   0          30m   192.168.39.194
nginx-deployment-594764c789-b292m   1/1     Running   0          33m   192.168.29.24 
nginx-deployment-594764c789-s226c   1/1     Running   0          30m   192.168.15.158
```

The Kubernetes service:
 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```

To better understand the problem I am describing in this post, the application I deployed will response Pod IP address to let us know which one received the request:

{% include figure image_path="/assets/images/posts/2022/04/eks-best-practice-load-balancing/imbalanced-test-result.png" alt="Testing the service and see the response from the backend." caption="Figure 1. Testing the service and see the response from the backend." %}

After running a loop and making at least **79 HTTP requests** in my test, I get the following response to know how the load has been distributed:

- `192.168.42.171`: 12 times
- `192.168.39.194`: 33 times
- `192.168.29.24`: 23 times
- `192.168.15.158`: 10 times

According to the testing, we can see the load is not very evenly distributed.

### Why this could happen?

As mentioned in the [previous post](/eks-best-practice-load-balancing-1-en), whether you are defining `externalTrafficPolicy=Cluster` or `externalTrafficPolicy=Local`, the routing behavior is relying on `iptables`(or `ipvs`) can be unpredictable. Because it is doing second layer of load balancing, which is totally unnecessary for increasing a hop in AWS VPC.

Elastic Load Balancer in AWS already provides a straightforward solution to balance your loads, and its algorithm will try to distribute the requests to all backend servers as even as possible. Doing load balancing in Kubernetes network generally is increasing the complexity of your architecture, and make traffic can be hard to trace; or, even worse, cause the imbalanced issue as you can observe.

This also makes the load balancing became unpredictable. Although the traffic send to the registered EC2 instance can be evenly distributed; however, it doesn't mean the load can be separated to Pods as well. You will never know which Pods will be routed due to this load balancing layer implemented by Kubernetes networking.

No matter choose Traefik, nginx-ingress, if you are still following the default load balancing pattern offered by upstream Kubernetes code, then you can expect the traffic can come with load imbalanced.

## How to optimize the load balancing?

The major problem is the default load balancing behavior can involve the Kubernetes load balancing and add a hop for the traffic. So you may start to wondering how to better resolve this problem; however, there is no specific feature can be adjusted on Kubernetes to remove the default load balancing, but it still could be possible to skip the Kubernetes load balancing and forward the traffic to Pods directly.

If you are running Pods on Amazon EKS and using default AWS VPC CNI Plugin[^aws-vpc-cni-plugin], you can expect your Pods should have dedicated secondary private IP address that can be communicated within your AWS VPC network; therefore, it also means that the IP address can be registered to your Elastic Load Balancer as backend target. The flow can be:

```
Client -> NLB (forawrd request to IP target) -> Pod IPs (Reach out to Pods directly)
```


For Application Load Balancer (ALB) and Network Load Balancer (NLB), both provide a feature that you can register backend targets with IP addresses ([NLB](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#target-type), [ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html#target-type). Note: Classic Load Balancer doesn't offer this option). We can simply to associate these Pod IP addresses as backend targets instead of using instances. As long as the Pod IP addresses are reachable, it can move the request be forwarded to the backend Pods by skipping the Kubernetes load balancing behavior.

### Using IP mode

So how to register Pod IP addresses in Elastic Load Balancer? A seamlessly way is to deploy your Kubernetes service and use AWS Load Balancer Controller[^aws-load-balancer-controller] to enable this feature. Instead of using the default Kubernetes controller to deploy your Elastic Load Balancer, using AWS Load Balancer Controller helps you manage load balancer resource including all functionality features and different type of load balancer such as NLB, ALB, both are can be supported by the controller. After installing the AWS Load Balancer on your EKS cluster, you can enable the IP registration type for your Pods by simply adding annotations to the deployment manifests.

#### Network Load Balancer (NLB)

Here is a deployment sample that use IP targets with pods deployed to Amazon EC2 nodes. Your Kubernetes service must be created as type `LoadBalancer`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "external"
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"
  ...
spec:
  type: LoadBalancer
  ...
```

#### Application Load Balancer (ALB)

To deploy application load balancer on Amazon EKS through the AWS Load Balancer Controller, you generally will create an Ingress object in your deployment. With the AWS Load Balancer Controller, it also provides supported annotation that can register pods as targets for the ALB. Traffic reaching the ALB is directly routed to pods for your service. Here is an example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: game-2048
  name: ingress-2048
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    ...
```

In the AWS EKS documentation, it also mentioned detailed guide regarding how to deploy these two load balancers and share an example by using IP target to register your Pods. If you are interested to learn more, please check out to the following documents to get more detail:

- [Network load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html)
- [Application load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)

By using IP mode, it totally removes the layer of load balancing manipulated by Kubernetes. This generally forward requests to the Pods without doing second forwarding:

{% include figure image_path="/assets/images/posts/2022/04/eks-best-practice-load-balancing/ip-target.png" alt="Register Pods with IP mode" caption="Figure 2. Register Pods with IP mode" %}

## Are you sure it is balanced? Let's have a test!

This time I used the same testing strategy as mentioned in the first problem description section and ran four Pods associated with Network Load Balancer using IP mode, which is showing below:

```bash
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP               NODE                                              NOMINATED NODE   READINESS GATES
nginx-deployment-75d48f6698-b5fm7   1/1     Running   0          35m     192.168.17.15    ip-192-168-5-38.ap-northeast-1.compute.internal   <none>           <none>
nginx-deployment-75d48f6698-l4gw5   1/1     Running   0          2m45s   192.168.27.143   ip-192-168-5-38.ap-northeast-1.compute.internal   <none>           <none>
nginx-deployment-75d48f6698-q2q57   1/1     Running   0          41m     192.168.22.126   ip-192-168-5-38.ap-northeast-1.compute.internal   <none>           <none>
nginx-deployment-75d48f6698-x5m25   1/1     Running   0          2m45s   192.168.14.48    ip-192-168-5-38.ap-northeast-1.compute.internal   <none>           <none>
```

After passing at least 50 requests, I can see the request distributions are showing below:

- `192.168.17.15`: 10 times
- `192.168.27.143`: 12 times
- `192.168.22.126`: 14 times
- `192.168.14.48`: 13 times

For each target, it nearly have ~25% chances will be routed evenly by the Network Load Balancer. Because it skip the load balancing layer of the Kubernetes, it will follow the routing algorithm[^routing-algorithm] and separate load evenly as we expected.

### I tried to use IP mode but the traffic still get imbalanced

In my testing, I was running a couple of Pods with nginx image and provided simple web server in my backend. The scenario in this article mentioning generally is describing all targets were using stateless HTTP connections. However, in some cases, it could be possible ELB might unequally route traffic to your targets if:

- Clients are routing requests to an incorrect IP address of a load balancer node with a DNS record that has an expired TTL.
- Sticky sessions (session affinity) are enabled for the load balancer. Sticky sessions use cookies to help the client maintain a connection to the same instance over a cookie's lifetime, which can cause imbalances over time.
- Available healthy instances aren’t evenly distributed across Availability Zones.
- Instances of a specific capacity type aren’t equally distributed across Availability Zones.
- There are long-lived TCP connections between clients and instances.
- The connection uses a WebSocket.

Generally speaking, if the client or any configuration can cause sticky session, it still have possibility can get the traffic imbalanced. The detail can refer to the following article on AWS knowledge center:

- [Why is Elastic Load Balancing unequally routing my load balancer traffic?](https://aws.amazon.com/premiumsupport/knowledge-center/elb-fix-unequal-traffic-routing/)

But overall, using the IP mode to register our Pods, literally can resolve the problem as we described due to the design of Kubernetes service networking.

## A summary if you would like to optimize the traffic imbalanced when using AWS Load Balancer Controller

### Using IP mode as register target to prevent Kubernetes additional hop

Although Elastic Load Balancer can offer an option to register your targets by instances, however, it generally would be suitable when you are running single service and expose it with a port on a dedicated EC2 instance. With Kubernetes service running on your EC2 instance but exposed as `NodePort` service, it can involve multiple Pods behind the service port offered on your instance due to the service load balancing. The packet can be replaced to other destination field of your Pod's private IP address when the packet flood into the instance through Linux ipvs or iptables rules.

If the service work load is relying on Kubernetes deployment, it is recommended  such as `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type` for NLB, `alb.ingress.kubernetes.io/target-type` for ALB.

### Prevent to enable sticky session on ELB

It is also important to make sure the Elastic Load Balancer won't stick your client session to specific target[^aws-elb-sticky-clb] [^aws-elb-sticky-alb]. Although Elastic Load Balancer provides cookie-based stickiness session to bind a user's session to a specific target, which can be achieved by configuring the load balancer attribute and also supported by AWS Load Balancer Controller as below, but to optimize the traffic imbalanced, it is recommended to avoid use the sticky session as it can potentially cause the phenomena.

```yaml
# ALB
alb.ingress.kubernetes.io/target-group-attributes: stickiness.enabled=true,stickiness.lb_cookie.duration_seconds=60

# NLB
service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: stickiness.enabled=true,stickiness.type=source_ip
```

### Equally distribute Pods across Availability Zones

As ELB requires to strike the balance between your Availability Zones to ensure the service high availability. This helps your traffic can correctly be separated on all backend target.

## Summary

In this article, it explains the practice of optimizing the load balancing and mitigate the imbalanced traffic problem when deploying service with Kubernetes. This article also brings you an overview and learn what other scenarios that you can potentially find out ELB might unequally route traffic to your backend targets.

In the next article we will review a couple of Kubernetes load balancer controllers that can be deployed on Amazon EKS and see what option can be the best practice for your environment.

{% include_relative related-post/eks-best-practice-load-balancing-en.md %}

## References

[^routing-algorithm]: How Elastic Load Balancing works - [Routing algorithm](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#routing-algorithm)
[^aws-vpc-cni-plugin]: [AWS VPC CNI Plugin](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html)
[^aws-load-balancer-controller]: [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)
[^aws-elb-sticky-clb]: [Configure sticky sessions for your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-sticky-sessions.html)
[^aws-elb-sticky-alb]: [Sticky sessions for your Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/sticky-sessions.html)
