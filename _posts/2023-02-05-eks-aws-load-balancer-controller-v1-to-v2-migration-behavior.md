---
title: "[AWS] 那些沒寫在文件上的事 - AWS Load Balancer Controller v1 升級至 v2"
description: "This article is sharing the best practice for doing load balancing on Amazon EKS, learn what is advantage and disadvantage of using different controller. We will discuss more detail about what is the problem of using default Kubernetes service deployment as mentioned on official document."
tags: ['aws', 'amazon web services', 'amazon', 'ELB', 'ALB', 'Load Balancer', 'Elastic Load Balancer', 'ALB Ingress Controller', 'Kubernetes', 'k8s', 'EKS', 'Elastic Kubernetes Service', 'AWS Load Balancer Controller']
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

隨著 Amazon EKS 對於 Kubernetes 1.22 的推出，1.21 也預計於 February 15, 2023 在 Amazon EKS 終止支援 [^eks-kubernetes-release-calendar]，對於 AWS Load Balancer Controller 仍然使用舊版本的用戶來說，勢必面臨需要升級 AWS Load Balancer Controller 至 2.4.1 以上版本的需要。若在現有環境中仍持續運行 AWS Load Balancer Controller v1 (原 ALB Ingress Controller)，此時此刻將可能來到遷移業務需求的高峰。由於 AWS Load Balancer Controller v2 版本帶來了許多的改進，並且與 v1 版本有很大的差異，絕對不是套用一個 YAML 文件就能搞定的事情。

在 AWS Load Balancer Controller 文件中提及了具體的幾個注意事項[^lb-migrate-v1-to-v2]，然而，有些情境文件上不見得會全部捕捉 (或是不巧剛好被我遇到)，以下列舉在 AWS Load Balancer Controller 遷移到 v2 版本之前，我個人在協助用戶執行遷移關注到的幾個有趣的問題。

## 那些沒寫在文件上的事

### ELB 資源命名規則改變帶來可能的停機行為

也許有的人發現 v1 版本的控制器升級到 v2 版本控制器的過程，會由 v2 控制器產生一個新的 ELB 資源，並且將相關資源部署到新產生的資源中完成遷移，舊有 v1 建立的 ELB 資源就不會繼續使用。使得完成遷移的過程如果有依賴舊有 ELB 資源的位置 (e.g. `fe584233-echoserver-echose-XXXX-XXXXXXX.ap-northeast-1.elb.amazonaws.com`) 或是有依賴服務關聯，就要記得去改對應的 DNS 紀錄。在文件上也確實具體提到了這項行為：

> The AWS LoadBalancer resource created for your Ingress will be preserved. If migrating from\<v1.1.3, a new AWS LoadBalancer resource will be created and the old AWS LoadBalancer will remain in the account. However, the old AWS LoadBalancer will not be used for the ingress resource.

對於很多用戶來說，這可能不是什麼大問題；但對於已經有許多系統依賴單一 ELB 資源的用戶來說，前期規劃安裝也沒想太多就直接給他上了。導致這樣的升級過程對這部分用戶來說，就像跑了廁所蹲了馬桶，卻仍然還便秘一樣令人不愉快。

於是就有仍然在運行 v1.0.1 版本的客戶們很天才地提了我甚至都沒思考過的升級路徑：

> 既然文件上是說 `<v1.1.3`，那是不是我先升級到大於這個版本之後 (例如：`v1.1.9`)，再升級到 v2 版本就可以保留原有的 ELB 資源了呢？
>
> 升級路徑為 v1.0.1 -> v1.1.9 -> v2

邏輯上好像沒有什麼謬誤，但在看過 AWS Load Balancer Controlelr 的原始代碼後，很遺憾，只能說這種想法真的是太美好了。

#### 實際測試行為

根據 AWS Load Balancer Controller v2 版本對應產生的 ELB 資源名稱都存在 `k8s` 保留字串的不正確觀察，我大可以預料到只要是舊版本的 v1 控制器遷移很可能都無法複用舊有的 ELB 資源。

但針對上述的行為我們可以大膽假設，仍需要仔細驗證一下相關的行為。為求真相，實際在我的環境簡單複製測試後，直接地瓦解了上述的論證。

##### Step 1：部署 ALB Ingress Controller v1.0.1

首先，我在我的環境運行了 `v1.0.1` 的控制器，歷經一番考古和 kubectl convert 轉換 (舊有的 API 宣告)，將舊版 v1.0.1 控制器成功安裝到 Kubernetes 1.20 版本中運行，並且部署一個簡單的範例應用：

```bash
# Running controller v1.0.1
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.1/docs/examples/rbac-role.yaml
$ wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.1/docs/examples/alb-ingress-controller.yaml
$ kubectl apply -f alb-ingress-controller.yaml
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)

W0130 12:45:29.518393       1 client_config.go:552] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
-------------------------------------------------------------------------------
AWS ALB Ingress controller
  Release:    v1.0.1
  Build:      git-ebac62dd
  Repository: https://github.com/kubernetes-sigs/aws-alb-ingress-controller.git
-------------------------------------------------------------------------------

# Running a sample application
$ kubectl describe ing -n echoserver echoserver
Address:          fe584233-echoserver-echose-XXXX-XXXXXXX.ap-northeast-1.elb.amazonaws.com
...
```

##### Step 2：升級至 ALB Ingress Controller v1.1.9

並且直接更新到 `v1.1.9` 版本，即使 ALB Ingress Controller 存在刷新操作，仍然針對原有的 Ingress 物件保留了原本的部署關聯 (`fe584233-echoserver-echose-XXXX-XXXXXXX.ap-northeast-1.elb.amazonaws.com`)：

```bash
# Upgrade and deploy to v1.1.9

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.9/docs/examples/rbac-role.yaml

# Use kubectl and update the image to "docker.io/amazon/aws-alb-ingress-controller:v1.1.9"
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "alb-ingress[a-zA-Z0-9-]+")

W0130 13:05:04.770613       1 client_config.go:549] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
-------------------------------------------------------------------------------
AWS ALB Ingress controller
  Release:    v1.1.9
  Build:      6c19d2fb
  Repository: https://github.com/kubernetes-sigs/aws-alb-ingress-controller.git
-------------------------------------------------------------------------------

# ELB name doesn't change
$ kubectl describe ing -n echoserver echoserver
Address:          fe584233-echoserver-echose-XXXX-XXXXXXX.ap-northeast-1.elb.amazonaws.com
```

##### Step 3：升級至 AWS Load Balancer Controller v2

在預備升級 v2 的過程 ELB 資源都持續存在，且 Ingress 也同樣關聯舊有的 ELB 資源；然而，一旦 v2 版本一部署下去，新的 ELB 資源立即被建立，並且使用不同的命名 (`k8s-echoserv-echoserv-XXXXXXXX-XXXXXXX`) 運作，並且關聯的 Ingress 和 Kubernetes Service 均遷移使用新的 ELB 資源：

```bash
# Update the controller to v2
# The old ALB Ingress controller has been uninstalled at this moment, and can see the ingress object is still preserved

$ kubectl describe ing -n echoserver echoserver
Address:          fe584233-echoserver-echose-XXXX-XXXXXXX.ap-northeast-1.elb.amazonaws.com
....

$ helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=eks \
  --set serviceAccount.create=false \
  --set serviceAccount.name =aws-load-balancer-controller

# Once v2 controller has been installed, the controller will update the ELB name

$ kubectl describe ing -n echoserver echoserver
Address:          k8s-echoserv-echoserv-XXXXXXXX-XXXXXXX.ap-northeast-1.elb.amazonaws.com
...

Events:
  Type    Reason                  Age   From     Message
  ----    ------                  ----  ----     -------
  Normal  SuccessfullyReconciled  11s   ingress  Successfully reconciled
```

此時，舊有的 ELB 資源 (`fe584233-echoserver-echose-XXXX-XXXXXXX.ap-northeast-1.elb.amazonaws.com `) 仍然存在，只是 AWS Load Balancer Controller 並未直接管理及操作該資源，並且不再執行註冊 Kubernetes Service 至相關 Target Group 資源。

#### 行為分析

從上述部署來看，我們可以觀察到 v1 版本的控制器在古時候，使用 namespace + ingress 名稱的方式進行組合命名，這項行為也確實在 `v1.0.1` 如此呼叫 (Source: [L299-L317](https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/v1.0.1/internal/alb/lb/loadbalancer.go#L299-L317))，在 `v1.1.9` (Source: [L285-L304](https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/v1.1.9/internal/alb/lb/loadbalancer.go#L285-L304)) 亦同。一個 `NameLB` 短短幾行副程式道盡前人的字串處理之美 (Source: [v1.0.1](https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/v1.0.1/internal/alb/generator/name.go#L23-L39), [v1.1.9](https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/v1.1.9/internal/alb/generator/name.go#L23-L39))：

```go
func (gen *NameGenerator) NameLB(namespace string, ingressName string) string {
  hasher := md5.New()
  _, _ = hasher.Write([]byte(namespace + ingressName))
  hash := hex.EncodeToString(hasher.Sum(nil))[:4]

  r, _ := regexp.Compile("[[:^alnum:]]")
  name := fmt.Sprintf("%s-%s-%s",
    r.ReplaceAllString(gen.ALBNamePrefix, "-"),
    r.ReplaceAllString(namespace, ""),
    r.ReplaceAllString(ingressName, ""),
  )
  if len(name) > 26 {
    name = name[:26]
  }
  name = name + "-" + hash
  return name
}
```

然而，在 v2 版本除了功能性的改進，針對 ALB Ingress Controller 也確實做了多項程式上的重構。最顯著的就是上述命名的變更，在 v2 版本的命名中存在了更多元的規範 (Source: [v2.4.4, L90-L124](https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/8d282339857c615b0bec6e3d984063c697e98f70/pkg/ingress/model_build_load_balancer.go#L90-L124))：

```go
func (t *defaultModelBuildTask) buildLoadBalancerName(_ context.Context, scheme elbv2model.LoadBalancerScheme) (string, error) {
  ...
  if len(explicitNames) == 1 {
    name, _ := explicitNames.PopAny()
    // The name of the loadbalancer can only have up to 32 characters
    if len(name) > 32 {
      return "", errors.New("load balancer name cannot be longer than 32 characters")
    }
    return name, nil
  }
  if len(explicitNames) > 1 {
    return "", errors.Errorf("conflicting load balancer name: %v", explicitNames)
  }
  uuidHash := sha256.New()
  _, _ = uuidHash.Write([]byte(t.clusterName))
  _, _ = uuidHash.Write([]byte(t.ingGroup.ID.String()))
  _, _ = uuidHash.Write([]byte(scheme))
  uuid := hex.EncodeToString(uuidHash.Sum(nil))

  if t.ingGroup.ID.IsExplicit() {
    payload := invalidLoadBalancerNamePattern.ReplaceAllString(t.ingGroup.ID.Name, "")
    return fmt.Sprintf("k8s-%.17s-%.10s", payload, uuid), nil
  }

  sanitizedNamespace := invalidLoadBalancerNamePattern.ReplaceAllString(t.ingGroup.ID.Namespace, "")
  sanitizedName := invalidLoadBalancerNamePattern.ReplaceAllString(t.ingGroup.ID.Name, "")
  return fmt.Sprintf("k8s-%.8s-%.8s-%.10s", sanitizedNamespace, sanitizedName, uuid), nil
}
```

除了對於 ELB 命名的檢查更為嚴謹了，也多了以 Cluster 名稱、Ingress Group 和多個關聯的識別資料進行雜湊產生 UUID，最終透過 `k8s-` 前綴組織成了人見 - 人們不見得愛的正則化命名。

## 總結

本章用了數百字的篇幅傳遞 v2 版本真的改很多東西，作為總結：

若從 v1 版本遷移至 v2 控制器涉及你目前所遭遇或未來將面臨的情境，則生成新的 ELB 資源都是**很有可能且可以預期的行為**。

在規劃遷移的同時，若尚未對 ELB 資源存取設計另一層存取介面的情境，考慮透過 DNS 紀錄 (CNAME) 方法管理應對 ELB 資源更新變更的存取位置是一種常見的做法，以降低用戶端因上述升級行為所產生的改動；也建議透過規劃相應的停機時間和更新 DNS 對應紀錄的變更紀錄納入考量。

畢竟，有很多事情只是人生暫時過不去的坎，在這樣的機制下，沒有什麼事情是清一下 DNS 快取以及「請稍候重試」提示訊息不能解決的。

## Related Posts

{% include_relative related-post/eks-best-practice-load-balancing-en.md %}

## References

[^eks-kubernetes-release-calendar]: [Amazon EKS Kubernetes versions - Amazon EKS Kubernetes release calendar](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-release-calendar)
[^lb-migrate-v1-to-v2]: [AWS Load Balancer Controller - Migrate from v1 to v2](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/upgrade/migrate_v1_v2/)
