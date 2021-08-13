---
title: "[AWS] 為 CloudWatch Log Group 自動設定日誌保留期限 (Retention)"
description: "當使用新版本的 AWS Console 建立 CloudWatch Log Group 時，其預設允許選擇對應的保留期間 (Retention)。然而，若涉及部署和配置的自動化，我將在這篇內容與你分享可用的解決方案"
tags: ['aws', 'amazon web services', 'CloudWatch', 'CloudWatch Log', 'amazon', 'Lambda', 'Lambda Function']
classes: wide
header:
  og_image: /assets/images/posts/2021/07/set-up-cloudwatch-log-group-retention-period-automation/create-cw-log-group.png
  teaser: /assets/images/posts/2021/07/set-up-cloudwatch-log-group-retention-period-automation/create-cw-log-group.png
---

日誌 (Log) 儲存一直是在資訊稽核中十分重要的一環，在 AWS 中，提供了 CloudWatch Logs 作為監控、存放不同 AWS 資源輸出的日誌檔案 (例如：EC2 等) [1]。

當使用新版本的 AWS Console 建立 CloudWatch Log Group 時，其預設允許選擇對應的保留期間 (Retention)。然而，若涉及部署和配置的自動化，我將在這篇內容與你分享可用的解決方案。

{% include figure image_path="/assets/images/posts/2021/07/set-up-cloudwatch-log-group-retention-period-automation/create-cw-log-group.png" alt="於 AWS Console 建立 CloudWatch Log Group 時選擇日誌保留時間 (Retention setting)" caption="於 AWS Console 建立 CloudWatch Log Group 時選擇日誌保留時間 (Retention setting)" %}

## 問題描述

預設情況下，若你使用的 AWS 服務資源存放日誌的行為 (例如：Amazon ECS、Amazon EKS、CodeBuild ... 等)、或是採用 `CreateLogGroup` API [2] 建立 CloudWatch Log 幫助你存放日誌，很有可能建立出來的 CloudWatch Log Group 日誌的存放時間會是「永遠 (Never Expire)」。

然而，一切事情可能都涉及到費用和運算成本的考量，即使 CloudWatch Log Group 每 GB 的「日誌儲存費用」十分便宜 [3]。但也許隨著組織規模的成長，日誌的儲存規模到達 PB 量級似乎也不是特別遙不可及的事情。伴隨著團隊的增加和規模的擴大，使用 AWS 進行產品遞交的成員也逐漸增加、使用了更多服務、開啟了更多的資源，相對的，可能間接增加了數以萬計的 CloudWatch Log Group。

如果這個時候，當你發現 CloudWatch Log Group 已經超過 100, 000 以上，甚至已經多到 100 頁 AWS Console 都塞不下的時候，你還發現每一個 CloudWatch Log Group 都是 "Never Expire"，經年累月下，可能從公司草創到的 Application Log 都還留在 CloudWatch Log Group 裡面。直到帳單上 CloudWatch Log 的費用又成長到超出預期，勢必得在合規的要求下，為這些日誌儲存期限手動一個一個設定到合乎預期的保留範圍。

於是你開始思考：**是否能在 CloudWatch Log Group 被建立好的同時也自動設定對應的保留期限？**

## 解決方案

答案是有的，即使在 `CreateLogGroup` API [2] 不支持更改保留時間 (Retention Period) 的情況下，仍有幾種方式可以在建立過程中達到類似的需求：

### A. 使用 AWS CloudFormation

> 什麼是 AWS CloudFormation?
>
> [AWS CloudFormation 是一個管理工具並且能夠使用通用的語法幫助你描述並且部署 AWS 相關的基礎建設和資源。](/using-aws-lambda-and-amazon-sns-to-get-file-change-notifications-from-aws-codecommit/)

如果你很習慣使用 CloudFormation 建立和管理 AWS 資源，你可以選擇使用 [AWS::Logs::LogGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-logs-loggroup.html#aws-resource-logs-loggroup-syntax.yaml)，幫助您建立和管理 CloudWatch Log Group 資源。

該類別提供了對應的屬性 (`RetentionInDays`) 可以設置 CloudWatch Log Group 預設的日誌保留時間：

```
RetentionInDays

The number of days to retain the log events in the specified log group. Possible values are: 1, 3, 5, 7, 14, 30, 60, 90, 120, 150, 180, 365, 400, 545, 731, 1827, and 3653.
```

其 CloudFormation template 在編寫過程中，可以描述如下，以設置 CloudWatch Log Group 資源 (myLogGroup) 預設的日誌保留時間為 7 天：

```yaml
Resources:
  myLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      RetentionInDays: 7
```

### B. 選擇使用 PutRetentionPolicy API [4] 幫助修改其屬性

#### AWS CLI

CloudWatch Log 同時提供了 `PutRetentionPolicy` API [4] 以幫助你修改特定 CloudWatch Log Group 的日誌保留期限 (Retention)。這意味著，您可以採用 AWS CLI 提供的對應以下命令並且設計你的自定義的腳本 (Shell Script) 定期幫助您更新 (例如使用 CronJob)。

AWS CLI 提供了以下命令 (為 CloudWatch Log Group: `my-logs` 設定日誌保留時間為 5 天) [5]：

```bash
aws logs put-retention-policy --log-group-name my-logs --retention-in-days 5
```

#### CloudWatch Event Rule + Lambda

亦或者，你可以基於 CloudWatch Log Group 建立事件，採用 Lambda Function 實現該實作。為了監控相關的事件，通常可以採用 CloudWatch Event 規則 (CloudWatch Event Rule) [6] 以捕捉相關的事件，並且透過其觸發定義的 Lambda Function 執行自定義操作，在這樣的架構下，其可能執行流程如下：

{% include figure image_path="/assets/images/posts/2021/07/set-up-cloudwatch-log-group-retention-period-automation/cwe-update-cw-log-retention-diagram.png" alt="CloudWatch Event Rule + Lambda 執行流程" caption="CloudWatch Event Rule + Lambda 執行流程" %}

- (1) 建立 CloudWatch Log Group (使用 `CreateLogGroup` API)
- (2) 觸發 CloudWatch Event Rule
- (3) 觸發自定義的 Lambda Function
- (4) 於 Lambda Function 中使用 `PutRetentionPolicy` API 幫助您修改 CloudWatch Log Group 對應的 Retention policy

要實作上述的流程，其需要步驟如下：

##### 1. 建立 IAM Role

不管是自行建立對應的 IAM Role 或是採用 AWS Lambda 建立過程中產生的對象，一般來說，其都需要允許操作 `logs:PutRetentionPolicy` 的權限，以確保 Lambda Function 在呼叫過程具備足夠的權限操作 CloudWatch Log Group 資源更新，例如：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowUpdateLogRetentionPolicy",
            "Effect": "Allow",
            "Action": "logs:PutRetentionPolicy",
            "Resource": "*"
        }
    ]
}
```

##### 2. 建立 Lambda Function

建立 Lambda [7] 以確保 CloudWatch Event 在觸發時能夠執行更改對應 CloudWatch Log Group 的日誌保留時間，我使用了簡易的 Python 應用程式碼完成這項工作：

```python
import json
import boto3

client = boto3.client('logs')

def getCloudWatchLogGroupName(event):
    if 'logGroupName' in event:
        return event['logGroupName']
    else:
        return event['detail']['requestParameters']['logGroupName']


def lambda_handler(event, context):
    RetentionDays = 7
    CloudWatchLogGroupName = getCloudWatchLogGroupName(event)

    print('[DEBUG] Updating Log Group {0}, retention: {1}'.format(CloudWatchLogGroupName, RetentionDays))

    response = client.put_retention_policy(
        logGroupName=CloudWatchLogGroupName,
        retentionInDays=RetentionDays
    )

    return response
```


##### 3. 設定 CloudWatch Event Rule

- 至 CloudWatch Console 並選擇對應的區域，例如: [https://ap-northeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-1#rules:](https://ap-northeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-1#rules:)
- 點擊 `Create rule`
- 於 `Event Source` 選擇 `Event Pattern`
  - Service Name: `CloudWatch Logs`
  - Event Type: `AWS API Call via CloudTrail`
- 選擇 `Specific operation(s)`: `CreateLogGroup`

例如以下的 CloudWatch Event Rule：

```
{
  "source": [
    "aws.logs"
  ],
  "detail-type": [
    "AWS API Call via CloudTrail"
  ],
  "detail": {
    "eventSource": [
      "logs.amazonaws.com"
    ],
    "eventName": [
      "CreateLogGroup"
    ]
  }
}
```

於 `Targets` 點擊 `Add Targets` 並且選擇前面 2. 建立的 Lambda Function 以確保其正確觸發。

至此，一旦完成建立上述流程，在建立 CloudWatch Log Group 後，便能正確的觸發 Lambda Function 並且修改 CloudWatch Log Group 對應的 Retention Period。

## 總結

在這篇內容中，我與你分享為 CloudWatch Log Group 自動設定日誌保留期限 (Retention)。並且進一步分享了一項自動化架構的實現流程，同時列舉了具體細節和範例設定。

如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道。

## References

- [1] [What is Amazon CloudWatch Logs?](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
- [2] [CreateLogGroup API](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_CreateLogGroup.html)
- [3] [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/)
- [4] [PutRetentionPolicy](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_PutRetentionPolicy.html)
- [5] [AWS CLI - aws logs ut-retention-policy](https://docs.aws.amazon.com/cli/latest/reference/logs/put-retention-policy.html)
- [6] [什麼是 Amazon CloudWatch Events](https://docs.aws.amazon.com/zh_tw/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html)
- [7] [Create a Lambda function with the console](https://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html)
