---
title: "[AWS][EKS] Zero downtime deployment(RollingUpdate) when using ALB Ingress Controller on Amazon EKS"
description: "This article is describing the thing you need to aware when using ALB Ingress Controller (AWS Load Balancer Controller) to do deployment and prevent 502 errors"
tags: ['aws', 'amazon web services', 'EC2', 'Elastic Compute Cloud', 'amazon', 'ELB', 'ALB', 'Load Balancer', 'Elastic Load Balancer', 'ALB Ingress Controller', 'Kubernetes', 'k8s', 'EKS', 'Elastic Kubernetes Service', 'AWS Load Balancer Controller']
header:
  image: /assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-4-side-note-of-deregister.png
  og_image: /assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-4-side-note-of-deregister.png
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
classes: wide
---

This article is describing the thing you need to aware when using ALB Ingress Controller (AWS Load Balancer Controller) to do deployment and prevent 502 errors.

## What's ALB Ingress Controller

Kubernetes doesn't involve the Application Load Balancer (ALB) deployment in the native implementation for using Kubernetes service object with `type=LoadBalancer`. Therefore, if you would like to expose your container service with Application Load Balancer (ALB) on EKS, it is recommended to integrate with ALB Ingress Controller.

If you don't know about what is the ALB Ingress Controller, here is an overview diagram to help you catch up:

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/controller-design.png" alt="How ALB ingress controller works" caption="How ALB ingress controller works - [source](https://kubernetes-sigs.github.io/aws-load-balancer-controller/guide/controller/how-it-works/)" %}

- (1) The controller watches for ingress events from the API server.
- (2) An ALB (ELBv2) is created in AWS for the new ingress resource. This ALB can be internet-facing or internal.
- (3) Target Groups are created in AWS for each unique Kubernetes service described in the ingress resource.
- (4) Listeners are created for every port detailed in your ingress resource annotations.
- (5) Rules(ELB Listener Rules) are created for each path specified in your ingress resource. This ensures traffic to a specific path is routed to the correct Kubernetes Service.

> Note: The new version of AWS ALB Ingress Controller is upcoming, while rename it to be "AWS Load Balancer Controller" with several new features coming out. For more detail, please refer the [GitHub project - kubernetes-sigs/aws-alb-ingress-controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller)

## How to deploy Kubernetes with ALB Ingress Controller (AWS Load Balancer Controller)?

Basically, the ALB Ingress Controller will be deployed as a Pod running on your worker node while continously monitor/watch your cluster state. Once there have any request for `Ingress`  object creation, ALB Ingress Controller will help you to manage and create Application Load Balancer resource. Here is a part of example for `v1.1.8` deployment manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: alb-ingress-controller
  name: alb-ingress-controller
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: alb-ingress-controller
  template:
    metadata:
      labels:
        app.kubernetes.io/name: alb-ingress-controller
    spec:
      containers:
        - name: alb-ingress-controller
          args:
            # Setting the ingress-class flag below ensures that only ingress resources with the
            # annotation kubernetes.io/ingress.class: "alb" are respected by the controller. You may
            # choose any class you'd like for this controller to respect.
            - --ingress-class=alb

            # REQUIRED
            # Name of your cluster. Used when naming resources created
            # by the ALB Ingress Controller, providing distinction between
            # clusters.
            # - --cluster-name=devCluster

            # AWS VPC ID this ingress controller will use to create AWS resources.
            # If unspecified, it will be discovered from ec2metadata.
            # - --aws-vpc-id=vpc-xxxxxx

            # AWS region this ingress controller will operate in.
            # If unspecified, it will be discovered from ec2metadata.
            # List of regions: http://docs.aws.amazon.com/general/latest/gr/rande.html#vpc_region
            # - --aws-region=us-west-1
           image: docker.io/amazon/aws-alb-ingress-controller:v1.1.8
      serviceAccountName: alb-ingress-controller
```

The deployment basically will run a copy of ALB Ingress Controller (pod/alb-ingress-controller-xxxxxxxx-xxxxx) in `kube-system`:

```bash
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   pod/alb-ingress-controller-5fd8d5d894-8kf7z   1/1     Running   0          28s

NAMESPACE     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/alb-ingress-controller   1/1     1            1           3m48s
```

Depends on your environment, suggested installation steps may involve the configuration of IRSA (IAM Role for Service Account) to grant permission for the ALB Ingress Controller Pods in order to interact with AWS resources, so it is recommended to take a look official documentation to help you quickly understand how to install ALB Ingress Controller:

- [ALB Ingress Controller on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)

In addition, the service can be deployed as `Ingress` Object. For example, if you tried to deploy the simple 2048 application:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.8/docs/examples/2048/2048-namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.8/docs/examples/2048/2048-deployment.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.8/docs/examples/2048/2048-service.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.8/docs/examples/2048/2048-ingress.yaml
```

The file `2048-ingress.yaml` is mentioning the `annotations`, `spec` in format that supported by ALB Ingress Controller can recognize:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: "2048-ingress"
  namespace: "2048-game"
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
  labels:
    app: 2048-ingress
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: "service-2048"
              servicePort: 80
```

The ingress object will construct ELB Listeners according rules and forward the connection to the corresponding backend(`serviceName`), which match the group of service `service-2048`, any traffic match the rule `/*` will be routed to the group of selected Pods. In this case, Pods are exposed on the worker node based on `type=NodePort`:

Content of the file `2048-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: "service-2048"
  namespace: "2048-game"
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: NodePort
  selector:
    app: "2048"
```

## So ... what's the problem?

**Zero downtime deployment** is always a big challenge for DevOps/Operation team for any kind of business. When you adopt the ALB Ingress Controller as a solution to expose your service, it has couple things need to take care due to the behavior of Kubernetes, ALB and ALB Ingress Controller ... you need to consider many perspectives. Especially, it can have some issue when you would like to roll out the new deployment for your Pods with ALB Ingress Controller.

Let's use the 2048 game as example to describe the scenario when you are trying to roll out a new version of your container application. In my environment, I have:

- A Kubernetes service `service/service-2048` using `NodePort` to expose the service
- The deployment also have 5 copy of Pods for 2048 game, which is my backend application waiting for connections forwarding by Application Load Balancer (ALB)

```bash
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
2048-game     pod/2048-deployment-58fb66554b-2f748          1/1     Running   0          53s
2048-game     pod/2048-deployment-58fb66554b-4hz5q          1/1     Running   0          53s
2048-game     pod/2048-deployment-58fb66554b-jdfps          1/1     Running   0          53s
2048-game     pod/2048-deployment-58fb66554b-rlpqm          1/1     Running   0          53s
2048-game     pod/2048-deployment-58fb66554b-s492n          1/1     Running   0          53s

NAMESPACE     NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
2048-game     service/service-2048   NodePort    10.100.53.119   <none>        80:30337/TCP    52s

NAMESPACE     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
2048-game     deployment.apps/2048-deployment          5/5     5            5           53s
```

And for sure, once the ALB Ingress Controller correctly set up and provision the ELB resource, the full domain of ELB also will be recorded to the `Ingress` object:

```
$ kubectl get ingress -n 2048-game
NAME           HOSTS   ADDRESS                                                                      PORTS   AGE
2048-ingress   *       xxxxxxxx-2048game-xxxxxxxx-xxxx-xxxxxxxxx.ap-northeast-1.elb.amazonaws.com   80      11m
```

I can use the DNS name as endpoint to visit my container service:

```bash
$ curl -s xxxxxxxx-2048game-xxxxxxxx-xxxx-xxxxxxxxx.ap-northeast-1.elb.amazonaws.com | head
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>2048</title>

  <link href="style/main.css" rel="stylesheet" type="text/css">
  <link rel="shortcut icon" href="favicon.ico">
  ...
```

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/alb-ingress-controller-with-2048-game-screenshot.png" alt="2048 Game deployed with ALB Ingress Controller" caption="2048 Game deployed with ALB Ingress Controller" %}

Like any kind of container application, as a administrator/SRE (Site Reliability Engineer)/part of operation team or DevOps engineer, the goal and our duty is: **we always try to ensure the service can run properly without any issue and without any interruption**, especially facing the challenges like: when your developers are saying that **"Oh! we need to upgrade/update the application"**, **"we are going to roll out a bug fix"**, **"the new feature is going to online"**, any service downtime can lead anyone of stakeholders(users, operation team or leadership) unhappy.

I am going to use a simple loop trick to continously access my container service via the endpoint `xxxxxxxx-2048game-xxxxxxxx-xxxx-xxxxxxxxx.ap-northeast-1.elb.amazonaws.com` to demonstrate a scenario: This is a popular web service and we always have customer access to it. (Like social media service, bitcoin trading platform or any else, we basically have zero tolerance for any service downtime as it can impact our revenue.), as below:

```bash
$ while true;do ./request-my-service.sh; sleep 0.1; done
HTTPCode=200_TotalTime=0.010038
HTTPCode=200_TotalTime=0.012131
HTTPCode=200_TotalTime=0.005366
HTTPCode=200_TotalTime=0.010119
HTTPCode=200_TotalTime=0.012066
HTTPCode=200_TotalTime=0.005451
HTTPCode=200_TotalTime=0.010006
HTTPCode=200_TotalTime=0.012084
HTTPCode=200_TotalTime=0.005598
HTTPCode=200_TotalTime=0.010086
HTTPCode=200_TotalTime=0.012162
HTTPCode=200_TotalTime=0.005278
HTTPCode=200_TotalTime=0.010326
HTTPCode=200_TotalTime=0.012193
HTTPCode=200_TotalTime=0.005347
...
```

Meanwhile, I am using `RollingUpdate` strategy in my Kubernetes deployment strategy with `maxUnavailable=25%`, which means, when Kubernetes need to update or patch(Like update the image or environment variables), the maximum number of unavailable Pods cannot exceed over `25%` as well as it ensures that at least 75% of the desired number of Pods are up (only replace 1-2 Pods if I have 5 copies at the same time):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: 2048-deployment
  namespace: 2048-game
spec:
  ...
  selector:
    matchLabels:
      app: "2048"
  ...
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```

### Scenario: Rolling the new container image to existing container application with potential service downtime

When rolling the new version of my container application (for example, I update my deployment by replacing the container image with image `nginx`), there potentially can have a period of time that would get `HTTP Status Code 502` error in few hits:

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/http-502-error-during-deployment-using-instance-mode.png" alt="The HTTP 502 Error response from ELB (instance mode)" caption="The HTTP 502 Error response from ELB during the rolling update deployment (instance mode)" %}

By default, ALB Ingress Controller is using `instance` mode to register targets(Pods) to the ELB Target Group by using worker nodes' instance ID with exposing `NodePort`. In this case, the traffic will the Kubernetes networking design to do second tier of transportation according to `externalTrafficPolicy` defined in Kubernetes `Service` object (No matter using `externalTrafficPolicy=Cluster` or `externalTrafficPolicy=Local`).

Due to the ALB Ingress Controller only care about to register Worker Node to the target group, so if the scenario doesn't involve the worker node replacement, the case basically have miniumun even no downtime(expect that it is rare to have downtime if the Kubernetes can perfectly handle the traffic forwarding), however, this is not how real world operate, few seconds downtime still can happen potentially due to the workflow below:

**This is the general workflow when the client reach out to the service endpoint (ELB) and how was traffic goes**

```
Client ----> ELB ----> Worker Node (iptables) / In this step it might be forwarded to other Worker Node ----> Pod
```

So, in these cases, you can see the downtime:

- (1) The client established the connection with ELB, ELB is trying to but the Worker Node is not ready.
- (2) Follow the iptables rules, the traffic forward to the Pod just terminated due to RollingUpdate (Or the Pod just got the reqeust but need to be terminated, it haven't response back yet, caused the ELB doesn't get the response from Pod.)
- (3) ELB established connection with Worker Node-1, once the packet enter into the Worker Node-1, it follows the iptables then forward it to the Pod running on Worker Node-2 (jump out the current worker node), however, the Worker Node-2 just got terminated due to auto scaling strategy or any replacement due to upgrade, caused the connection lost.

Let's say if you try to remove the encapsulation layer of the Kubernetes networking design and make thing more easier based on the AWS supported CNI Plugin (Only rely on the ELB to forward the traffic to the Pod directly by using `IP mode` with annotation setting `alb.ingress.kubernetes.io/target-type: ip` in my Ingress object), you can see the downtime more obvious when Pod doing RollingUpdate. That's because not only the problem we mentioned the issues in case (1)/(2)/(3), but also there has different topic on the behavior of ALB Ingress Controller need to be covered if the question comes to **zero downtime deployment**:

Here is an example by using IP mode (`alb.ingress.kubernetes.io/target-type: ip`) as [resgistration type](https://kubernetes-sigs.github.io/aws-load-balancer-controller/guide/ingress/annotations/#target-type) to route traffic directly to the Pod IP

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: "2048-ingress"
  namespace: "2048-game"
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
  labels:
    app: 2048-ingress
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: "service-2048"
              servicePort: 80
```

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/alb-target-group-with-pod-in-ip-mode.png" alt="An example when using IP mode in ALB Ingress Controller" caption="An example when using IP mode in ALB Ingress Controller - Can see my Pods all are registering with Pod owns IP address" %}

Again follow the issue we mentioned (1) (2) (3), when doing the rolling update (I was replacing the image again in `IP mode`), similar problem can be observed, potentially you can have 10-15 seconds even longer downtime can show up:

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/http-502-error-during-deployment-using-ip-mode.png" alt="The HTTP 502 Error response from ELB (IP mode)" caption="The HTTP 502 Error response from ELB during the rolling update deployment (IP mode)" %}

When Kubernetes is rolling the deployment, in the target group, you can see ALB Ingress Controller was issuing old targets draining process(Old Pods) in the meantime

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/old-target-doing-draining-in-target-group-with-ip-mode.png" alt="Old targets were going to be draining state in target group" caption="Old targets were going to be draining state in target group" %}

However, you still can see HTTP 502/504 errors exceed 3-10 seconds for single requset

```bash
HTTPCode=200_TotalTime=0.005413
2048
HTTPCode=200_TotalTime=0.009980
502 Bad Gateway
HTTPCode=502_TotalTime=3.076954
2048
HTTPCode=200_TotalTime=0.005700
2048
HTTPCode=200_TotalTime=0.010019
502 Bad Gateway
HTTPCode=502_TotalTime=3.081601
2048
HTTPCode=200_TotalTime=0.005527
502 Bad Gateway
HTTPCode=502_TotalTime=3.070947
502 Bad Gateway
HTTPCode=502_TotalTime=3.187812
504 Gateway Time-out
HTTPCode=504_TotalTime=10.006324
Welcome to nginx!
HTTPCode=200_TotalTime=0.011838
Welcome to nginx!
```

### The issue and the workflow of ALB Ingress Controller

Let's use this scenario as it is a edge problem we need to consider for most use case, the issue is that the workflow between the Kubernetes, ALB Ingress Controller and ELB can lead HTTP status code 502/503(5xx) erros during deployment when having Pod termination.

**In short, when a pod is being replaced, ALB Ingress Controller registers the new pod in the target group and removes the old Pods.** However, at the same time:

- For the the new Pods, the target is in `initial` state, until it pass the defined health check threshold (ALB health check)
- For the old Pods is remaining as `draining` state, until it completes draining action for the in-flight connection, or reaching out the `Deregistration delay` defined in the target group.

Which result in the service to be unavailable and return HTTP 502.

To better understand that, I made following diagrams, so it might be helpful to you understanding the workflow:

1) In diagram, I used following IP addresses to remark and help you recognize new/old Pods. Here is the initial deployment.

- Old Pods: Target-1(Private IP: 10.1.1.1), Target-2(Private IP: 10.2.2.2)
- New Pods: Target-3(Private IP: 10.3.3.3), Target-4(Private IP: 10.4.4.4)

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-1-initial.png" alt="Deployment workflow of ALB Ingress Controller - 1. the initial deployment" caption="Deployment workflow of ALB Ingress Controller - 1. the initial deployment" %}

2) At this stage, I was doing container image update and start rolling out the new copies of Pods. The ALB Ingress Controller made RegisterTarget API call to ELB on behalf of the Kubernetes.

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-2-register-target.png" alt="Deployment workflow of ALB Ingress Controller - 2. start rolling out the new copies of Pods and ALB Ingress Controller is going to issue RegisterTarget API call" caption="Deployment workflow of ALB Ingress Controller - 2. start rolling out the new copies of Pods and ALB Ingress Controller is going to issue RegisterTarget API call" %}

3) Meanwhile, the DeregisterTarget API will be called by ALB Ingress Controller and new targets are in `initial` state.

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-3-start-dereigster-old-target.png" alt="Deployment workflow of ALB Ingress Controller - 3. ALB Ingress Controller start to dereigster old targets on ELB Target Group" caption="Deployment workflow of ALB Ingress Controller - 3. ALB Ingress Controller start to dereigster old targets on ELB Target Group" %}

4) At this stage, anything could happen to cause service outage. Because the DeregisterTarget API call might take some time to process, but, Kubernetes doesn't have any design to monitor the current state of the ELB Target Group, it only care about rolling the new version of Pods and terminate old one.

In this case, if the Pod got terminated by Kubernetes but `Target-1` or `Target-2` are still leaving in the ELB Target Group as `Active`/`Healthy` state (It need to wait few seconds to be `Unhealthy` once it reach out to the threshold of ELB HTTP health check), result in the ELB cannot forward the front-end request to the backend correctly.

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-4-side-note-of-deregister.png" alt="Deployment workflow of ALB Ingress Controller - 4. Note: issue cause by inconsistent state between Kubernetes and ELB" caption="Deployment workflow of ALB Ingress Controller - 4. Note: issue cause by inconsistent state between Kubernetes and ELB" %}

5) ELB received the DeregisterTarget request. So the ELB Target Group will start to perform connection draining(set old targets as `draining`), and mark the `Target-1`/`Target-2` as `draining` state, any new connection won't be routed to these old targets.

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-5-elb-doing-draining.png" alt="Deployment workflow of ALB Ingress Controller - 5. ELB start to perform connection draining for old targets" caption="Deployment workflow of ALB Ingress Controller - 5. ELB start to perform connection draining for old targets" %}

6) However, here brings another issue: if the new targets (`Target-3` and `Target-4`) are still working on passing the health check of ELB(Currently those are in `Initial` state), there has no backend can provide service at this moment, which can cause the ELB only can return HTTTP 5XX status code

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-6-response-5XX-error.png" alt="Deployment workflow of ALB Ingress Controller - 6. ELB response HTTP 5XX error due to no healthy targets in can provide service" caption="Deployment workflow of ALB Ingress Controller - 6. ELB response HTTP 5XX error due to no healthy targets in can provide service" %}

7) Until the new Pods is in `Running` state as well as can react the health check reqeust from ELB through HTTP/HTTPS protocol, the ELB end up mark the targets as `Active/Healthy` and the service become available

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/deployment-workflow-alb-ingress-controller-7-new-pod-in-active.png" alt="Deployment workflow of ALB Ingress Controller - 7. The service need to wait a period to recover until new targets passed the ELB health check" caption="Deployment workflow of ALB Ingress Controller - 7. The service need to wait a period to recover until new targets passed the ELB health check" %}


## How to resolve the issue and meet zero-downtime?

As mentioned in the previous workflow, it is required to use several workarounds to ensure the Pod state consistency between ALB, ALB Ingress Controller and Kubernetes. Here are few things you can aware:

- Since version [v1.1.6](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/releases/tag/v1.1.6), ALB Ingress Controller introduced [Pod readiness gates](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/blob/master/docs/guide/ingress/pod-conditions.md): This feature can monitor the rolling deployment state and trigger the deployment pause due to any unexpected issue(such as: getting timeout error for AWS APIs), which guarantees you always have Pods in the Target Group even having issue on calling ELB APIs when doing rolling update.

Here is an example to add a readiness gate with `conditionType: target-health.alb.ingress.k8s.aws/<ingress name>_<service name>_<service port>`

(As it might be changed afterward,For more detail, please refer the documentation provided by ALB Ingress Controller (AWS Load Balancer Controller) project on GitHub):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  clusterIP: None
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: internal
spec:
  rules:
    - http:
        paths:
          - backend:
              serviceName: nginx-service
              servicePort: 80
            path: /*
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      readinessGates:
      - conditionType: target-health.alb.ingress.k8s.aws/nginx-ingress_nginx-service_80
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

- For existing connections(As mentioned in workflow-4), the case is involving the gracefully shutdown/termination handling in Kubernetes. Therefore, it is requires to use the method provided by Kubernetes.

You can use [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination) with `preStop` hook and make some pause(like using `sleep` command) for Pod termination. This trick ensures ALB can have some time to completely remove old targets on Target Group (It is recommended to adjust longer based on your `Deregistration delay`):

```yaml
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 40"]
  terminationGracePeriodSeconds: 70
```

> Note: If a container has a preStop hook configured, that runs before the container enters the Terminated state. Also, if the preStop hook needs longer to complete than the default grace period allows, you must modify `terminationGracePeriodSeconds` to suit this.


### An example to achieve zero downtime when doing rolling update after applying methods above

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: "2048-deployment"
  namespace: "2048-game"
spec:
  selector:
    matchLabels:
      app: "2048"
  replicas: 5
  template:
    metadata:
      labels:
        app: "2048"
    spec:
      readinessGates:
      - conditionType: target-health.alb.ingress.k8s.aws/2048-ingress_service-2048_80
      terminationGracePeriodSeconds: 70
      containers:
      - image: alexwhen/docker-2048
        imagePullPolicy: Always
        name: "2048"
        ports:
        - containerPort: 80
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 40"]
```

Here is an example after following the practice I was getting a try. The deployment will apply the feature and can see the status of the readiness gates:

```bash
$ kubectl get pods -n 2048-game -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP               NODE                                              NOMINATED NODE   READINESS GATES
2048-deployment-99b6fb474-c97ht   1/1     Running   0          78s   192.168.14.209   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.compute.internal   <none>           1/1
2048-deployment-99b6fb474-dcxfs   1/1     Running   0          78s   192.168.31.47    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.compute.internal   <none>           1/1
2048-deployment-99b6fb474-kvhhh   1/1     Running   0          54s   192.168.29.6     XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.compute.internal   <none>           1/1
2048-deployment-99b6fb474-vhjbg   1/1     Running   0          54s   192.168.18.161   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.compute.internal   <none>           1/1
2048-deployment-99b6fb474-xfd5q   1/1     Running   0          78s   192.168.16.183   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.compute.internal   <none>           1/1
```

Once rolling the new version of the container image, the deployment goes smoothly and prevent the downtime issue as mentioned in previous paragraphs:

{% include figure image_path="/assets/images/posts/2020/10/zero-downtime-deployment-when-using-alb-ingress-controller/zero-downtime-deployment-with-alb-demo.png" alt="Zero downtime with ALB Ingress Controller - Can see the targets are gracefully replaced when the Kubernetes is doing rolling update" caption="Zero downtime with ALB Ingress Controller - Can see the targets are gracefully replaced when the Kubernetes is doing rolling update" %}

In my scenario, the Kubernetes need to take at least 40 seconds termination period for single Pod, so the old targets are gradually moved out instead of remove all of them at once within few seconds, until entire target group only exists new targets.

Also, the client can get normal responses from old Pods/existing connection during the deployment:

```bash
HTTPCode=200_TotalTime=0.012028
2048
HTTPCode=200_TotalTime=0.005383
2048
HTTPCode=200_TotalTime=0.010174
2048
HTTPCode=200_TotalTime=0.012233
Welcome to nginx!
HTTPCode=200_TotalTime=0.007116
2048
HTTPCode=200_TotalTime=0.010090
2048
HTTPCode=200_TotalTime=0.012201
2048
HTTPCode=200_TotalTime=0.005532
2048
HTTPCode=200_TotalTime=0.010107
2048
HTTPCode=200_TotalTime=0.012163
Welcome to nginx!
HTTPCode=200_TotalTime=0.005452
Welcome to nginx!
HTTPCode=200_TotalTime=0.009950
2048
HTTPCode=200_TotalTime=0.012082
Welcome to nginx!
HTTPCode=200_TotalTime=0.005349
2048
HTTPCode=200_TotalTime=0.010142
2048
HTTPCode=200_TotalTime=0.012143
2048
HTTPCode=200_TotalTime=0.005507

...
HTTPCode=200_TotalTime=0.012149
Welcome to nginx!
HTTPCode=200_TotalTime=0.005364
Welcome to nginx!
HTTPCode=200_TotalTime=0.010021
Welcome to nginx!
HTTPCode=200_TotalTime=0.012092
Welcome to nginx!
HTTPCode=200_TotalTime=0.005463
Welcome to nginx!
HTTPCode=200_TotalTime=0.010136
Welcome to nginx!
```

This is the practice in case having ALB Ingress Controller for doing graceful deployment with `RollingUpdate`. However, it is another big topic need to be discussed regarding what type of the application when rolling the update. Because maybe some kind of applications need to establish long connection with the ELB or have requirement for considering persistence data need to be stored on the backend. All these things can bring out other issues we need to talk about.

But in summarize, with the deployment strategy above, it is also recommended to design the client/backend application as stateless, implement retry and fault-tolerance. These mothod usually help to reduce the customer complain and provide better user experience in most common use case.

## Conclusion

Due to the current design of Kubernetes, it is involving the state inconsistent issue when you are exposing the service with Application Load Balancer. Therefore, in this article, I mentioned the potential issue when doing rolling update in the scenario having container service integrating with ALB Ingress Controller (AWS Load Balancer Controller).

Even the technology is always in revolution, I am still willing to help people better handle the deployment strategy. I used couple hours to draft this content and tried to cover several major issues, metioned things you might need to aware, break down the entire workflow and shared few practical suggestions that can be achieved in ALB Ingress Controller in order to meet the goal when doing zero downtime deployment.

The article was written based on my working experience (Of course many communications back and forth with different customers using AWS), it might not be perfect but I hope it is helpful to you. For sure, if you find any typo or have any suggestions, please feel free and leave comment below.

### References

- [ALB Ingress Controller on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
- [Using pod conditions / pod readiness gates](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/blob/master/docs/guide/ingress/pod-conditions.md)
- [Issue#1124](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/issues/1124)
- [Issue#814](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/issues/814)
- [Issue#1064](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/issues/1064)
