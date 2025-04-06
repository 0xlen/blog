---
title: "在 AWS 上打造 Serverless (無伺服器) 雲端綠界支付金流 (ECPay)"
description: "金流系統一直是許多線上服務、電商所必備的功能之一；常見的使用方式往往是透過第三方金流服務提供商對應的模組實現 (例如：OpenCart, WooCommerce)，以串接金流系統實現購物結帳的機制。為了實現在 AWS 上串接綠界金流 (ECPay) 提供信用卡付款機制，並且簡化管理和維護流程，以下內容以 Serverless 技術作為背景，簡介在 AWS 上實作的相關細節。"
tags: ['aws', 'amazon web services', 'Lambda', 'Lambda Function', 'Serverless', 'Serverless Application Model', 'SAM']
header:
  og_image: /assets/images/posts/2022/05/serverless-ecpay-aws/create-payment-flow.png
  teaser: /assets/images/posts/2022/05/serverless-ecpay-aws/create-payment-flow.png
classes: wide
---

金流系統一直是許多線上服務、電商所必備的功能之一；常見的使用方式往往是透過第三方金流服務提供商對應的模組實現 (例如：OpenCart, WooCommerce)，以串接金流系統實現購物結帳的機制。

即使是設計自己的金流系統串接，想要將這些金流服務串接應用在雲端服務上，對於不熟悉使用雲端技術的用戶來說，仍然需要花一些時間摸索以達到這項目的。

為了實現在 AWS 上串接綠界金流 (ECPay) 提供信用卡付款機制，並且簡化管理和維護流程，以下內容以 Serverless 技術作為背景，簡介在 AWS 上實作的相關細節。

## 什麼是 Serverless？為什麼要用 Serverless 技術？

無伺服器運算 (Serverless) 概念提出了拋棄舊有傳統管理 Server ，在過去，你需要維護及管理運行你應用程式的基礎運算系統；Serverless 提出以平台即服務（PaaS）的運作模式，提供簡單且容易操作的微型的架構，使得你不需要部署、組態或管理伺服器，只需要運用 Serverless 相關的解決方案，將你的程式碼推送至相關平台，運行所需要的伺服器服務皆由雲端平台來提供。

自 2014 年 AWS 推出 Serverless 服務以來，已經儼然成為一項 IT 部署解決方案中熱門的運行架構；學會使用 Serverles，將幫助你更容易且快速地推行不同類型的應用，將你的想法付諸於實際實現。

在過去，如果要運作相關的金流服務，我可能會需要開啟一台虛擬機器 24 小時的提供商業邏輯的運作，並且，可能會因為一些非預期狀況而多許多而外的工作，例如：突然暴增的訂單請求、過高的使用負載等原因影響到業務。往往程式開發出來只是個起點，後面的系統維護工作才是更大的挑戰。

選擇使用 Serverless 架構設計的考量之一，便是考量部署、組態或管理伺服器的長遠維護性，尤其對於金流這種關鍵業務來說，更是至關重要。

## 怎麼在 AWS 上與綠界支付科技串接

一般來說，在 AWS 上要建立 Serverless 為基礎架構的應用程式，通常涉及幾種不同的關鍵服務；以這項支付金流系統為例，我採用了以下的 AWS 服務

### AWS Lambda

是一種無伺服器的運算服務，可讓您執行程式但不必佈建或管理伺服器、建立工作負載感知叢集擴展邏輯、維護事件整合或管理執行階段。使用 Lambda，您可以透過虛擬方式執行任何類型的應用程式或後端服務，全部無需管理。在這篇內容中，我使用了 Lambda Function 以推送訊息至 Amazon SNS 以發佈檔案更新。 Amazon CloudWatch

### API Gateway

為 AWS 提供的託管服務，可以讓開發人員輕鬆地建立、發佈、維護、監控和保護任何規模的 API。API 可作為應用程式的「前門」，以便從後端服務存取資料、商業邏輯或功能。使用 API Gateway 時，您可以建立 RESTful API 等應用程式。API Gateway 支援無伺服器工作負載和 Web 應用程式。API Gateway 可以用以負責處理有關接受和處理多達數十萬個並行 API 呼叫的所有工作，包括流量管理、CORS 支援、授權和存取控制。API Gateway 沒有最低費用或啟動成本。您要為收到的 API 呼叫和資料傳輸量支付費用。

### Serverless Application Model (SAM)

為了更容易實現在 AWS 上運作串接綠界金流支付並且以 Serverless 架構運作的目標，我使用了 AWS [Serverless Application Model](https://aws.amazon.com/serverless/sam/) (簡稱為 SAM) 為開發流程的重要工具，用以建置 Serverless 應用服務。

Serverless Application Model (SAM) 提供了一系列以簡易描述的方法，提供你用以更容易，在很多情況下，你可能無需非常熟悉不同 AWS 服務的設置，即可透過 SAM 建立無伺服器應用程式。

為了幫助你快速了解 Serverless Application Model (SAM) 的運作機制以及簡介，以下簡短 10 分鐘的影片分享了其運作流程的機制：

{% include video id="HYtyzT-4Mqc" provider="youtube" %}

### 運作流程與架構概覽 (Architecture Overview)

{% include figure image_path="/assets/images/posts/2022/05/serverless-ecpay-aws/create-payment-flow.png" alt="以綠界支付科技為基礎的 Serverless 雲端金流概覽 (建立訂單)" caption="以綠界支付科技為基礎的 Serverless 雲端金流概覽 (建立訂單)" %}

上述的使用者流程描述了用戶及各個彼此 AWS 服務之間的運作關係，以建立訂單為例，我們可以藉由綠界支付 (ECPay) 開放的對應 SDK 實際在 AWS 中設計屬於其結帳流程的相關操作，透過 API Gateway 建立一致的對外 API 接口，並且實作建立訂單訊息的 Python 應用程式，並且將其透過 Serverless Application Model 提供的 CLI 工具 (SAM CLI) 將應用部署至 AWS Lambda 上運作。

在這種運作架構下，我們只需要專注設計結帳流程和用戶流程的設計，其餘的服務運作機制，均可以交付由 AWS Serverless 相關的解決方案滿足業務實作需求。

如果你對於具體的實作內容有興趣，可以透過下列的連結獲取更多資訊：

{% capture CallToAction %}
### 學習在一天內於 AWS 雲端搭建 Serverless 架構的金流服務

從 Zero 到 Hero，學習 AWS 入門知識並深入了解、應用 Serverless 相關的服務及架構，同時學會在 AWS 上使用不同的解決方案實踐無伺服器技術 (Serverless)，運行屬於你的雲端金流系統

<a href="https://academy.easontechtalk.com/aws-serverless-ecpay" class="btn btn--inverse btn--x-large">點擊獲取更多資訊</a>
<a href="/courses" class="btn btn--x-large">延伸學習</a>
{% endcapture %}

<div class="notice--success">{{ CallToAction | markdownify }}</div>

## 總結與延伸學習

本篇內容簡介了以 AWS Serverless 為基礎架構設計金流應用程式的實作流程，以及提及部分在 AWS 上實現串接綠界支付科技的對應機制，並且分享一項參考架構。如果你對於本篇具體的實作內容有興趣，可以利用以下連結獲取完整的內容：

- [打造 Serverless (無伺服器) 雲端金流](https://academy.easontechtalk.com/aws-serverless-ecpay)

如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道。
