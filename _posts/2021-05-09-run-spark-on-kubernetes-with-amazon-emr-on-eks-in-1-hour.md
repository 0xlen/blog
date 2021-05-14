---
title: "一小時內搭建好 Amazon EMR on EKS 運行 Spark on Kuberentes"
description: "在這篇內容中，我會展示如何整合 AWS EMR 及 Amazon EKS 以幫助你運行 Spark 進行你的 Big Data 大數據 Map Reduce 運算"
tags: ['aws', 'amazon web services', 'EMR', 'EKS', 'Spark', 'BigData', 'Kubernetes', 'Map Reduce']
header:
  og_image: /assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/cover.png
  teaser: /assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/cover.png
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
classes: wide
---

自 2020 年 12 月，Amazon EMR 推出支持 Amazon EKS 作為一項部署運行 Apache Spark 的方案。因此，在這篇內容中，我會展示如何整合 AWS EMR 及 Amazon EKS 以幫助你運行 Spark 進行你的 Big Data 大數據 Map Reduce 運算。

{% include figure image_path="/assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/cover.png" alt="一小時內搭建好 Amazon EMR on EKS 運行 Spark on Kuberentes" caption="一小時內搭建好 Amazon EMR on EKS 運行 Spark on Kuberentes" %}

## 簡介

**Amazon EMR 是什麼？**

> 如果你慣於運行 Apache Spark, Apache Hive, Apache HBase 等平台。Amazon EMR 主要針對這些開源服務提供了整合的託管式集群，以供你可以輕鬆的運行你的 Map Reduce 運算工作。如果你自己有建置過 Spark 或是 Hadoop Cluster 的經驗，會很清楚知道要維護集群的可用性是一項複雜的工作。
>
> Amazon EMR 就是為了解決這項問題而存在，同時也幫助你自動化像是基於大數據運算所需要的底層硬體擴展、Performance Tunning。你只需要透過 Amazon EMR 提供的介面就能夠輕鬆設定、操作和擴展您的大數據環境，可以處理 PB 規模等級以上的分析工作，同時，通常成本不到傳統內部部署解決方案的一半，速度也比標準 Apache Spark 快 3 倍以上。

**Amazon EKS 是什麼？**

> Amazon Elastic Kubernetes Service (Amazon EKS) 屬於託管式 Kubernetes Cluster 的一種，在過去，如果你要自己搭建 Kubernetes 集群，是一項非常複雜的工作。同時，Kubernetes 社群通常 3-6 個月都會有提供相應的更新，這促使你會需要解決頻繁更新 Kubernetes 集群的問題。
>
> Amazon EKS 就是為了減少你維運複雜性而存在，這項服務幫助你可以輕鬆的擁有高可用的 Kubernetes Cluster、自動化且不中斷的更新流程，專注在部署及擴展你的 Kubernetes 應用程式。

在過去，Amazon EMR 會幫助你執行管理底層運算資源，包含幫助你配置 Cluster Manager、管理 Spark Worker 及 Executor 的資源。即使 Spark 原生就有以 Kubernetes 集群作為一項可支持運算方式，其配置及設定過程仍有一定的複雜性。但在 Amazon EMR 這項支持推出後，你無需做過多的設定，只需專注發出 Submit Job 進行 Map Reduce 運算，意味著你可以更容易的使用 Kubernetes 的運算集群幫助你執行 Spark 運算，並執行 Map Reduce 工作。也可以在單一個 Kubernetes cluster 中運行混合不同的應用，進一步為你的團隊或是組織更有效的運用運算資源，並節省成本。

甚至，在 Amazon EKS，其支持使用 AWS Fargate 幫助你以無服務器的狀態運行你的任務，你更無需煩惱如何有效管理底層所需的運算資源、擴展等工作。 

## 架構概覽 (Architecture Overview)

{% include figure image_path="/assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/emr-on-eks-architecture.png" alt="架構概覽" caption="架構概覽" %}

Amazon EMR on EKS 主要提供了單一的 API 介面，以供你可以使用過去操作 Amazon EMR 的體驗執行 Submit Job (Spark) 的工作。差別僅在於，Amazon EMR 並不會管理任何底層運算集群，其底層會交付由 Amazon EKS 其管理的 Kubernetes Cluster 執行運算，其提供了跨可用區的高可用運算，以避免單點故障的影響。最終，將交付由運行在 Kubernetes Cluster 的 Spark Worker 及 Executor 將結果及輸出，統一儲存於資料儲存 (Amazon S3)。

如果你已經慣於使用 Amazon EKS 的用戶，要理解並使用這樣的解決方案並不陌生。但如果你是第一次了解 Amazon EKS，看完這樣的架構仍無法理解如何操作。因此，以下我將一一講解怎麼使用，幫助你在一小時內就能夠搭建完成並且運行第一個 Map Reduce 運算！

## 開始動手做

為了幫助你快速搭建，以下我將一一列舉運行步驟：

### 預先準備工作 

以下環境以 Linux 為基礎範例安裝必要套件：

- 安裝 AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

為 AWS CLI 設定相應的操作用戶資訊

```
aws --version
aws configure
```

註：你可以在你的 IAM User 中附加相關的 IAM Policy (e.g AdministratorAccess) 以具備執行這項測試相關的權限。並在這個階段你需要指定使用相應的 IAM User 對應的 Access Key & Secure Key。

- 安裝 eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

- 安裝 kubectl (可選，以下使用 Kubernetes 1.19.6 為範例)

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```

### Step 1. 建立 EKS Cluster

為了方便測試，以下資源都在 N.Virginia (us-east-1) 區域建立。在該區域建立一個 EKS Cluster (名稱為 `emr-eks-demo`) 並啟用三個節點運行：

```bash
eksctl create cluster --name emr-eks-demo --nodes 3 --region us-east-1
```

註：EKS Cluster 建立過程約 15-20 分鐘

### Step 2. 為 Amazonn EMR 賦予操作 Kubernetes Cluster 的權限

為了方便測試及區別，我在 Kubernetes Cluster 中建立一個用於計算 EMR 任務的 namespace (Kubernetes namespace)，並命名為 `emr-job`

```bash
kubectl create ns emr-job
```

建立後，下一步便是為 Amazon EMR 賦予操作該 namespace 相應的權限 (Amazon EMR 將會使用 Service Linked Role `AWSServiceRoleForAmazonEMRContainers` 操作集群)，因此，這個動作將會使用 eksctl 建立 Amazon EMR 操作 EKS Cluster 所必須的 RBAC 相關權限，如此一來，使用 Amazon EMR 送出運算任務時，才能正確的在 EKS Cluster 中建立相關的資源 (例如：Job, Pod, Secret ... 等等)：

```bash
eksctl create iamidentitymapping \
    --cluster emr-eks-demo \
    --namespace emr-job \
    --service-name "emr-containers"
```

註：這個步驟會由 eksctl 工具幫助你配置 RBAC 以及 `aws-auth` ConfigMap。請使用最新版本的 eksctl 工具以確保選項可以被支持，你可以使用以下命令驗證。

```bash
$ kubectl get cm/aws-auth -o yaml -n kube-system
apiVersion: v1
data:
  mapRoles: |
      - rolearn: arn:aws:iam::11122233344:role/AWSServiceRoleForAmazonEMRContainers
        username: emr-containers
```

### Step 3. 啟用 OIDC Provider 以利後續配置 IAM Role for Service Account (IRSA)

為了讓 Amazon EMR 啟動的容器 (Pod) 能夠有相應的權限存取、寫入日誌 (輸出至 Amazon S3 等) 以儲存任務運行的資訊，Amazon EMR 同樣使用了 Amazon EKS 所提供的 IAM Role for Service Account (IRSA) 功能為容器賦予這項權限。在 Amazon EMR 中會為運行於 Amazon EKS 的 Pod 指定使用特定的 Job Execution Role (IAM Role)。為了啟用這樣的功能，首先可以使用 eksctl 工具幫助你關聯 EKS 必要關聯的 OIDC Provider：

```bash
eksctl utils associate-iam-oidc-provider --cluster emr-eks-demo --approve
```

### Step 4. 建立 EMR 可以操作的角色 (IAM Role) 並更新對應的權限

在成功利用 eksctl 關聯 OIDC Provider 後，下一步的動作便是建立 Job Execution Role 以利啟動的 Pod 擁有操作 Amazon S3、CloudWatch 的權限，這些權限為必要並且用於監控任務狀態和存取日誌的權限。

- 將以下的 IAM Policy 儲存為 `job-execution-role-policy.json`：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:PutLogEvents",
                "logs:CreateLogStream",
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams"
            ],
            "Resource": [
                "arn:aws:logs:*:*:*"
            ]
        }
    ]
}
```

使用 AWS CLI 建立 IAM Role Policy，名稱為 (`EMR-EKS-JobExecutionRolePolicy`)：

```bash
aws iam create-policy --policy-name EMR-EKS-JobExecutionRolePolicy --policy-document file://job-execution-role-policy.json
```

[output]
```
{
    "Policy": {
        "PolicyName": "EMR-EKS-JobExecutionRolePolicy",
        "Arn": "arn:aws:iam::111122223333:policy/EMR-EKS-JobExecutionRolePolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2021-05-09T12:39:39+00:00",
        "UpdateDate": "2021-05-09T12:39:39+00:00"
        ...
    }
}
```

- 使用 eksctl 工具建立 IAM Role / Service Account，名稱為 (`emr-on-eks-job-execution-role`)，請將前述獲得的 Policy ARN (`arn:aws:iam::111122223333:policy/EMR-EKS-JobExecutionRolePolicy`) 根據你的環境取代以下命令 `--attach-policy-arn` 中的設置：

```bash
eksctl create iamserviceaccount \
    --name emr-on-eks-job-execution-role \
    --namespace emr-job \
    --cluster emr-eks-demo \
    --attach-policy-arn arn:aws:iam::111122223333:policy/EMR-EKS-JobExecutionRolePolicy \
    --approve \
    --override-existing-serviceaccounts
```

註：請注意 Service Account 名稱建議使用小寫，以符合 Kubernetes 命名規範 (DNS-1123)，否則將會在建立過程遇到下列錯誤。

```
2021-05-09 12:41:47 [✖]  failed to create service account emr-job/EMR-on-EKS-job-execution-role: ServiceAccount "EMR-on-EKS-job-execution-role" is invalid: metadata.name: Invalid value: "EMR-on-EKS-job-execution-role": a DNS-1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*')
```

為了能夠讓 Amazon EMR 具備操作 IAM Roles for Service Accounts (IRSA) 和 Kubernetes Cluster 相關的 Service Account 權限，下一步的動作便是設定相關 Job Execution Role 對應的 Trust Relationship，以賦予 Pod 能夠在運行時，有權限獲得臨時性憑證：

- 獲取 IAM Role 名稱 (`arn:aws:iam::11122233344:role/eksctl-emr-eks-demo-addon-iamserviceaccount-Role1-18HQGD62AI3QM`)：

```bash
$ kubectl describe sa/emr-on-eks-job-execution-role -n emr-job

Name:                emr-on-eks-job-execution-role
Namespace:           emr-job
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::11122233344:role/eksctl-emr-eks-demo-addon-iamserviceaccount-Role1-18HQGD62AI3QM
...
```

- 更新 EMR 操作 Job Execution Role 相應的 Trust Policy：

在取得 IAM Role ARN 後，請再套用命令前，將以下更換為你相應的 IAM Role 名稱 (`<eksctl-emr-eks-demo-addon-iamserviceaccount-Role1-18HQGD62AI3QM>`)：


```bash
 aws emr-containers update-role-trust-policy \
    --cluster-name emr-eks-demo \
    --namespace emr-job \
    --role-name <eksctl-emr-eks-demo-addon-iamserviceaccount-Role1-18HQGD62AI3QM>
```

[output]

```
Successfully updated trust policy of role eksctl-emr-eks-demo-addon-iamserviceaccount-Role1-18HQGD62AI3QM
```

### Step 5. 將 Amazon EKS 註冊至 Amazon EMR 作為可運算對象 (Virtual Cluster)

使用以下命令於 EMR 建立一個 Virtual Cluster (可以將 `virtual_cluster_name` 更換為你容易識別的名稱)：

```bash
aws emr-containers create-virtual-cluster \
    --name virtual_cluster_name \
    --container-provider '{
        "id": "emr-eks-demo",
        "type": "EKS",
        "info": {
        "eksInfo": {
                "namespace": "emr-job"
            }
        }
    }'
```

[output]
```
{
    "id": "01h5iv9ihkzwoxslejev29bl1",
    "name": "virtual_cluster_name",
    "arn": "arn:aws:emr-containers:us-east-1:11122233344:/virtualclusters/01h5iv9ihkzwoxslejev29bl1"
}
```

這裡取得的 `01h5iv9ihkzwoxslejev29bl1` 便是在 EMR 中相應的 Virtual Cluster ID。你同時也可以在 EMR Console 中檢視 Virtual Cluster：

{% include figure image_path="/assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/virtual-cluster-in-emr-console.png" alt="在 EMR Console 中檢視 Virtual Cluster" caption="在 EMR Console 中檢視 Virtual Cluster" %}

## 運行你的 Map Reduce Job！

你可以使用 `aws emr-containers start-job-run` 運行一個 Job。使用以下命令套用前，有部分

> -virtual-cluster-id <前面獲取的 Virtual Cluster ID，例如：01h5iv9ihkzwoxslejev29bl1>
> --name <自訂的 job 名稱>
> --execution-role-arn <前面建立的 Job Execution Role，例如：arn:aws:iam::11122233344:role/eksctl-emr-eks-demo-addon-iamserviceaccount-Role1-18HQGD62AI3QM>
>
> "entryPoint": <啟動應用位置 (可以是 Spark 支持的格式，例如 s3://<bucket-name-path>/<script-name>.py)>
> "entryPointArguments": [<應用啟動相關的參數>]
>
> {"cloudWatchMonitoringConfiguration": {"logGroupName": "<自定義的 CloudWatch Log Group 名稱>", "logStreamNamePrefix": "<自定義的 CloudWatch Log Group Stream 前綴>"}
>
> {"s3MonitoringConfiguration": {"logUri": "s3://<存放 EMR 日誌的 S3 Bucket>" }

(運行 Spark Job 完整配置)

```bash
aws emr-containers start-job-run \
    --virtual-cluster-id 123456 \
    --name myjob \
    --execution-role-arn execution-role-arn \
    --release-label emr-6.2.0-latest \
    --job-driver '{"sparkSubmitJobDriver": {"entryPoint": "entryPoint_location", "entryPointArguments": [arguments_list], "sparkSubmitParameters": "--class <main_class> --conf spark.executor.instances=2 --conf spark.executor.memory=2G --conf spark.executor.cores=2 --conf spark.driver.cores=1"}}' \
    --configuration-overrides '{"applicationConfiguration": [{"classification": "spark-defaults", "properties": {"spark.driver.memory": "2G"}}], "monitoringConfiguration": {"cloudWatchMonitoringConfiguration": {"logGroupName": "log_group_name", "logStreamNamePrefix": "log_stream_prefix"}, "persistentAppUI":"ENABLED",  "s3MonitoringConfiguration": {"logUri": "s3://my_s3_log_location" }}}'   
```


(套用上述設置運行 Spark Job 的一項使用範例)

```bash
aws emr-containers start-job-run \
    --virtual-cluster-id 01h5iv9ihkzwoxslejev29bl1 \
    --name pi-job \
    --execution-role-arn arn:aws:iam::11122233344:role/eksctl-emr-eks-demo-addon-iamserviceaccount-Role1-18HQGD62AI3QM \
    --release-label emr-6.2.0-latest \
    --job-driver '{"sparkSubmitJobDriver": {"entryPoint": "local:///usr/lib/spark/examples/src/main/python/pi.py", "sparkSubmitParameters": "--conf spark.executor.instances=1 --conf spark.executor.memory=2G --conf spark.executor.cores=1 --conf spark.driver.cores=1"}}' \
    --configuration-overrides '{"applicationConfiguration": [{"classification": "spark-defaults", "properties": {"spark.driver.memory": "2G"}}], "monitoringConfiguration": {"cloudWatchMonitoringConfiguration": {"logGroupName": "emr-on-eks-log", "logStreamNamePrefix": "pi"}, "persistentAppUI":"ENABLED",  "s3MonitoringConfiguration": {"logUri": "s3://My-EMR-Log-Bucket" }}}'
```

[output]

```
{
    "id": "00000002uah9c3rkt9v",
    "name": "pi-job",
    "arn": "arn:aws:emr-containers:us-east-1:11122233344:/virtualclusters/01h5iv9ihkzwoxslejev29bl1/jobruns/00000002uah9c3rkt9v",
    "virtualClusterId": "01h5iv9ihkzwoxslejev29bl1"
}
```

如果成功送出，在返回訊息中可以得知建立的 Job ID，在 EMR Console 中可以檢視 Job 運行的狀態。：

{% include figure image_path="/assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/view-job-state-on-emr-console.png" alt="在 EMR Console 中檢視 Job 運行的狀態" caption="在 EMR Console 中檢視 Job 運行的狀態" %}

同時，在 EKS Cluster 中也可以注意到由 EMR 啟動的相關 Spark Containers (Driver, Executor, Job)

```
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
emr-job       pod/00000002uah9c3rkt9v-gbqhl          3/3     Running   0          2m2s
emr-job       pod/spark-00000002uah9c3rkt9v-driver   0/2     Pending   0          81s

NAMESPACE     NAME                                                            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
emr-job       service/spark-00000002uah9c3rkt9v-929ad579514b9f34-driver-svc   ClusterIP   None          <none>        7078/TCP,7079/TCP,4040/TCP   82s

NAMESPACE   NAME                            COMPLETIONS   DURATION   AGE
emr-job     job.batch/00000002uah9c3rkt9v   0/1           2m5s       2m5s
```

一旦這些 Spark 的相關元件被正確啟動，在 EMR Console 中檢視 Job 狀態時，點擊 Job 旁邊的 "View logs" 會跳出彈掉視窗顯示 Spark UI 檢視相關的 Job 詳細資訊 (請注意瀏覽器是否有阻擋彈跳視窗)：

{% include figure image_path="/assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/view-job-on-spark.png" alt="Spark 中顯示的 Job 狀態" caption="Spark 中顯示的 Job 狀態" %}

當 Job 完成運算，你可以在 EMR Console 中得知該任務完成的變化：

{% include figure image_path="/assets/images/posts/2021/05/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour/view-job-completed-on-emr-console.png" alt="檢視 Completed Job" caption="檢視 Completed Job" %}

## 清除資源 Clean up

若你要終止並移除所有資源，可以依照下列順序執行清除：

```bash
# 清除 EKS Cluster
eksctl delete cluster --name emr-eks-demo --region us-east-1

# 清除 EMR Virtual Cluser
aws emr-containers delete-virtual-cluster --id <Virtual Cluster ID>

# 清除 IAM Policy
aws iam delete-policy --policy-arn arn:aws:iam::11122233344:policy/EMR-EKS-JobExecutionRolePolicy
```

## 總結

在這篇內容中，我展示了如何使用 Amazon EMR + Amazon EKS 快速部署一個範例的運行 EMR on EKS 的執行架構，運行大數據運算工作，能夠協助你在配置時具體了解其流程，幫助你在一小時內搭建好 Amazon EMR on EKS 運行 Spark on Kuberentes。

在我實際搭建時，由於涉及不同服務，所以對於剛接觸相關服務的人來說可能十分陌生並且會花費大量時間在閱讀文件，希望上述的內容，同樣也能幫助你成功運行這樣的解決方案，如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道，並且分享給有需要搭建環境的人，幫助他們快速上手這項功能。

## 看更多系列文章

- [一小時內搭建好 Amazon EMR on EKS 運行 Spark on Kuberentes](/run-spark-on-kubernetes-with-amazon-emr-on-eks-in-1-hour)
- [在 Amazon EMR on EKS 使用 AWS Fargate 運行你的 Big Data 大數據 Map Reduce 運算](/run-spark-on-kubernetes-with-amazon-emr-on-eks-using-aws-fargate)
