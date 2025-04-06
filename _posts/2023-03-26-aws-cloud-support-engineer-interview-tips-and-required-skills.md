---
title: "在 AWS Cloud Support Engineer 面試中脫穎而出：必備的技術能力和重要特質"
description: "如果你正準備面試 AWS Cloud Support Engineer 的職位，在這篇文章中，我將與你分享我對於 AWS Cloud Support Engineer 所需的必備技能和特質，幫助你更好地了解這份職位且思考自己目前所具備的能力是與該團隊所交集，讓你在面試中脫穎而出。"
tags: ['amazon', 'aws', 'amazon web services', 'work', 'Cloud Support', 'Cloud Support Engineer']
classes: wide
header:
  og_image: /assets/images/posts/2023/03/aws-cloud-support-engineer-interview-tips-and-required-skills/cover.jpg
  teaser: /assets/images/posts/2023/03/aws-cloud-support-engineer-interview-tips-and-required-skills/cover.jpg
---

如果你正準備面試 AWS Cloud Support Engineer 的職位，在這篇文章中，我將與你分享我對於 AWS Cloud Support Engineer 所需的必備技能和特質，幫助你更好地了解這份職位且思考自己目前所具備的能力是與該團隊所交集，讓你在面試中脫穎而出。

{% include figure image_path="assets/images/posts/2023/03/aws-cloud-support-engineer-interview-tips-and-required-skills/cover.jpg" alt="在 AWS Cloud Support Engineer 面試中脫穎而出：必備的技術能力和重要特質" caption="在 AWS Cloud Support Engineer 面試中脫穎而出：必備的技術能力和重要特質 (source: [Unsplash](https://unsplash.com/photos/7aakZdIl4vg))" %}

詳細內容可以參考以下我的團隊針對 AWS Cloud Support Engineer (雲端工程師) 職涯分享會中所提到的具體細節，裡面也包含了部分面試流程和一些小秘訣幫助你可以更好地掌握我們團隊所看重的能力：

{% include video id="Q0qkKhxAo04" provider="youtube" %}

**請注意本篇內容純屬個人觀點**，會分享這些內容的目的是希望有興趣投遞 AWS Cloud Support Engineer 能夠了解具體應該提升哪些必備技能，也同時分享一些日常為客戶解決技術問題時非常重要的能力。**寫這篇內容的目的不是要幫助大家 Crash 考題，也不代表任何官方的參考指南。**

即使你面試時展現把考題背熟的能力，進來團隊終究會在實際面對客戶問題時怕得要死，因為客戶問題往往都是沒有被定義清楚但又希望你能給予解答。如果只會背誦這些內容，仍然無法實際為客戶解決任何問題。

另外團隊召聘所看重的核心技能也可能隨時間變化，面試也不是一般考試，重要的是了解你能為團隊貢獻什麼樣的技能，看的是不同維度的全面評估。因此以下內容屬於我個人在團隊的經驗和處理客戶案例認為非常重要的必備技能，僅供參考。

如果你還不是很清楚 AWS Cloud Support Engineer 在做什麼，我十分推薦你可以參考我在 AWS 職涯系列的相關文章，以幫助你逐步建立對於這個職位的認識：

{% include_relative related-post/aws-career.md %}

## 基本技術能力 (Fundamental Technical Skill)

很多應徵者把這份工作當成一般設定環境的 IT Helpdesk 或是只是單純的客戶服務職位，以為遵循 Runbook 就能解決大部分工作上的問題。但實際 AWS Support 做的工作與一般公司的 IT Support 會與想像中有蠻大的差異，即使工作上以 Ticket 形式與客戶互動，但角色仍偏向顧問服務性質，直接被拉進客戶會議直接一個人打十個討論問題更是你都可能會遇到的情境，我會建議在應徵這份工作前可以有個心理準備。

我常觀察到很多候選人即使在 IT 界從業多年，對於很多基本的知識都有很大的落差 (例如：我聽過有人說用 `ping` 可以測網站的 Port 80 看網站是不是掛了)。這種現象在只專注做開發相關工作的工程師身上尤為明顯 (嚴格上來說很多軟體工程師職位都是在「實作」，面對的很多產品規格都已經在現有的封裝函式庫或是公開解決方案的 API 上定義能夠直接套用，所以可能也沒太多機會思考這種這麼核心底層的問題)。

不論是開發人員或是系統維運人員，可能在高階應用和實作上能夠滿足「使用者」身份角度的需求，所以也不需要對於基礎知識有太深入的了解，使得當東西壞掉或問題情境複雜時便束手無策 (也是感謝這些人才讓我有飯吃)。

但 AWS Cloud Support Engineer 就像是醫生這項職業，醫生必須根據病人描述的徵狀跟現有資訊提出正確的診斷步驟，用正確的診斷工具 (例如：聽診器、X 光)，最後開出正確的藥來緩解病人的症狀；工程師在協助 Troubleshooting 問題時也需要依照自己對於客戶提出問題的背景，知道要收集什麼資訊進行分析、用什麼樣正確的工具。

**甚至有時候客戶給的資訊還是錯的，這就十分仰賴對於基礎知識至上層的全面了解，提供正確的排查方向，否則就只是在亂查一通。**

### 網路概論 (Networking)

網路概論基本包含：

- 基本的 TCP/IP 協議運作
- OSI 網路七層的基本架構 (由於 AWS Cloud Support Engineer 主要負責的產品就是 "Web Services"，所以 L4-L7 的討論不少，延伸知識例如常見的 Route、SSL、HTTP 等基本認識)
- IPv4 的組成 (學習 AWS VPC 時對於 Subnet 的基本認識尤為重要)
- DNS 的運作及除錯
- HTTP/SSL/TLS
- VPN (如果招聘團隊不著重處理網路相關的產品，則不會是討論的重心)
- 熟悉不同協議中對應的網路除錯工具和實際排查案例

DNS 算是基本中再基本到不行的必備知識，我自己個人倒是遇過很多連基本 DNS 協議都不太熟的候選人 (當然也有些從事 IT 工作的客戶也不是那麼熟)，最常見的問題就是 DNS 查詢的具體流程、DNS 協議的組成和問題除錯。

遇到問題除錯的場景或是系統故障就將矛頭直接指向應用程式或是服務端，壓根沒有想到實際造成問題的其實是 DNS 不正確設定或是一些非預期行為導致。

關於網路概論，有太多的免費資源可以參考，甚至有一些很不錯的參考資源可以具體幫助你了解。可能光以下的連結要全盤了解就讀不完了，我這裡就不一一列舉：

- [What is DNS? - Introduction to Domain Name System](https://www.youtube.com/watch?v=e2xLV7pCOLI)
- [High network performance](https://hpbn.co/)

請注意問題的深度仍取決招聘團隊所看重的技術能力，如果是專注網路相關 AWS 服務的團隊，則可能在網路的部分就會更加深入；但對於其他專業團隊來說，由於更多的重心在於協助特定領域的 AWS 產品，則具備基本網路問題排查能力即可滿足協助客戶的情境。

### 作業系統 (Operating System)

由於我個人不太熟 Windows，為避免誤人子弟，這邊就僅列舉我認為非常實用的 Linux 資源，以及基礎到不能再基礎的檔案系統章節 (其實把鳥哥所有章節認真讀完並且實作，可能就足以面對 60-80% 有關 Linux 維運的情境)：

Linux (file system/operation/administration knowledges)
- [Linux 的檔案權限與目錄配置](https://linux.vbird.org/linux_basic/centos5/0210filepermission-centos5.php)
- [Linux 檔案與目錄管理](https://linux.vbird.org/linux_basic/centos5/0220filemanager-centos5.php)
- [Linux 磁碟與檔案系統管理](https://linux.vbird.org/linux_basic/centos5/0230filesystem-centos5.php)

如果你完全並未具備這方面的經驗，搭建一個 Web Service (HTTP) 涵蓋網路概論至作業系統基本維運操作過程中所必備的知識都是必須的。

## 重要特質

以下列舉幾個我認為應徵 AWS Cloud Support Engineer 所需具備的幾項重要特質：

### 客戶服務技能和溝通能力

AWS Cloud Support Engineer 的主要工作是為客戶提供協助，並且常常會需要將複雜的技術問題拆解成能理解的步驟。讓客戶甚至是其他團隊 (例如開發團隊) 能夠清楚地知道如何排查問題、修正哪些錯誤。**需要能夠清晰、明確地傳達信息，解釋問題和解決方案。**

與一般軟體開發工作不同的是，AWS Cloud Support Engineer 由於需要經手系統故障排查的情境，不免會接受客戶環境上的壓力，例如有時候客戶在系統故障影響到營收的同時，充滿緊張和壓力的情境下只希望能盡快把問題解決 (~~急急急~~)，原本這些專業的 IT 人員也會瞬間變得很不理智。

試想下你剛進問題現場才短短的 5 分鐘，進行 Live troubleshooting 的同時試著釐清問題檢查每一個項目，並且引導客戶執行正確的步驟確認 (因為有時候客戶給的資訊是錯的)。但有時候客戶就是會覺得你在浪費他的時間，具備保持冷靜和耐心特質的重要性在這種情境下就特別顯著。我個人自己聽過的就有：
- *你到底會不會*
- *這沒必要看，我現在只想要他恢復*
- *你看這沒用啊，你行不行啊，我檢查過了這沒有問題*
- *跟你溝通沒有意義，請找 ECS 專家來跟我溝通*

客戶有情緒但你不能帶著情緒協助，因為這樣就是大家都一起 Panic (~~大家一起急急急~~)。我在這份工作確實也學習到很多溝通軟技巧，如果再拉一堆不相干的人進來，我的經驗是這通常只會把問題攪得更亂，對問題調查沒有太大的幫助。

你可以想想自己過去是否有類似的經驗，談論如何與其他人有效地溝通，如何解決複雜的技術問題以及如何處理緊急情況。

### 解決問題的能力

AWS Cloud Support Engineer 需要有效、精確地識別和解決問題。基本的邏輯和良好的分析能力是必須的，並能夠迅速掌握問題的本質。同時，你需要能夠綜合多個方面的信息，從而定位出問題的癥結點。

簡單來說就是排查問題的過程邏輯要對、能夠正確分析問題、使用正確的方法和工具，知道當問題發生時要如何排查、為什麼用這些工具、為何查 A 不是查 B。比如網站連不上為什麼是用 ping 而不是其他工具、用 ping 返回的結果代表什麼、正確獲取結果後調查的方向是什麼？。

而不是收集到一堆無用的資訊胡亂瞎猜，將問題弄得更發散 (反面案例即是前面提到使用 `ping` 檢查 Port 80 能不能正常連通)。

### 主動學習、問題研究能力和自我提升

AWS 產品不斷推陳出新，基本上已經學不完，這項工作不得不跟隨客戶的快速腳步不斷地持續的學習和自我提升。

由於這份工作的角色也從原本用戶端 (使用者) 變成解決問題的角色，在你的專業領域中也必須擁有深入研究問題的能力，當客戶拋出未知且未定義清楚的問題，你通常才能具體的給予明確的排查方向。

## 專業技術能力 (Professional Technical Competency)

由於各個專業領域都有各自側重的項目，例如，專注 Database 專業的工程師跟專注 Linux 領域專業的工程師對於 Linux 知識的要求定義可能有所不同。可能 Database 專業的工程師具體了解 Linux 的基本原理、知道一些基本的指令和明白檔案系統、檔案權限管理、基本問題排查即可；但 Linux 專業的工程師可能就要非常了解 Linux process 運作、知道如何使用 Linux 的工具更加了解系統效能、知道 kernel dump 怎麼解讀、troubleshooting 等等知識。[^aws-support-faq]

每個專業領域會有基礎需要知道的基本知識，但團隊的技能樹也都是隨著客戶需求在變化，解決的問題也是日益更新。以下分享一些我認為可能對於所有專業團隊來說都十分有幫助的學習資源：

### AWS 相關的知識

AWS Cloud Support Engineer 的工作是支援 AWS 的客戶，你需要熟悉 AWS 的服務和產品，並能夠協助客戶解決他們遇到的問題。同時，你需要知道如何設定和管理 AWS 環境，以及如何進行故障排除。因此如果你對 AWS 技術有著深刻的了解，這會對於你在入職之後非常有幫助。

- [AWS Training](https://www.aws.training/): 包含了各式免費 AWS 訓練的資源
- [AWS Educate](https://aws.amazon.com/education/awseducate): AWS Educate 是給予大專院校學生或是任何有意展開雲端旅程的任何人，免費學習和進行 AWS 服務使用試驗的計畫，裡面有一些 Training 通常也直接包含了 Cloud Support Associate 角色的相關訓練材料，能幫助你更加了解這份職位的工作內容和必備技能

其他專業技術領域則參考招聘簡介中所提到的對應專業技能，不同技術團隊所重視的技能樹基於專注的產品多少都有些不同，可以透過產品頁面大致了解相關的細節：

- [All AWS Products]https://aws.amazon.com/products/

### Container / DevOps / Deployment 領域

對於整個 AWS Support，我可能還稱不上非常了解，但如果是 DevOps 和容器技術相關領域的團隊，個人對於該領域小有心得還能分享點東西。

我所在的團隊大部分涵蓋 AWS 服務包含以下：

- Elastic Container Service (ECS)
- Elastic Kubernetes Service (EKS)
- AWS Fargate
- Amazon Elastic Container Registry (ECR)
- CloudFormation
- AWS Batch
- AWS AppMesh
- AppRunner
- CodeCommit
- CodeBuild
- CodePipeline
- CodeDeploy
- CodeStar
- Cloud Map
- CodeArtifact
- Cloud Development Kit (CDK)
- Amazon Managed Service for Prometheus (AMP)
- Elastic Beanstalk
- AWS OpsWorks
- Artifact
- X Ray
- DevOps Guru
- CodeGuru

目前我的團隊一個人可能可以支持到快 40 個不同的 AWS Service，上述服務基本上都是客戶如果拋出問題來我都會有機會協助。

有鑒於我們團隊也在積極尋求合適的人才，以下是我們團隊十分看重的技術經驗和能力，部分附上一些你可以參考的學習資源：

Linux
- Container and Virtualization features
- Container Networking
- Performance analysis (I/O, process state, CPU, memory)

Kubernetes / Docker
- [CERTIFIED KUBERNETES ADMINISTRATOR (CKA)](https://www.cncf.io/certification/cka/)
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [learnk8s](https://learnk8s.io/)
- [Docker official document](https://docs.docker.com/)
- [Kubernetes official document](https://kubernetes.io/)

CI/CD
- 部署策略
- 基本的版本控制方法
- Infrastructure as Code (IaC)

## 總結

在這篇內容中，簡述了有關 AWS Cloud Support Engineer 必備的技術能力、特質和相關可以參考的學習材料。如果你是正在考慮加入 AWS Cloud Support Engineer 團隊，希望這些內容能夠更加幫助你建立更多認識。

這篇內容也更像是我自己對於 AWS Cloud Support Engineer 技術職位所具備的長遠學習路徑有個基本的指南，並且幫助你思考如何在你的問題中使用具體的案例正確的展現這些能力。

{% include_relative related-post/easontechtalk-community-tw-ad.md %}

## 看更多系列文章

{% include_relative related-post/aws-career.md %}

## References

[^aws-support-faq]: [關於 Cloud Support Engineer 職位的常見問題 - 請問面試過程中會問到多深入呢？](/aws-cloud-support-engineer-faq/#q-%E4%BB%A5-big-datanetworkingdatabase--xyz-%E8%AB%8B%E5%95%8F%E9%9D%A2%E8%A9%A6%E9%81%8E%E7%A8%8B%E4%B8%AD%E6%9C%83%E5%95%8F%E5%88%B0-linux-os-%E6%88%96%E6%98%AF-networking-%E7%9B%B8%E9%97%9C%E7%9A%84%E5%95%8F%E9%A1%8C%E5%97%8E-%E9%82%84%E6%98%AF%E5%81%8F%E5%90%91-aws-%E7%94%A2%E5%93%81-abc-%E5%91%A2%E6%9C%83%E5%95%8F%E5%88%B0%E5%A4%9A%E6%B7%B1%E5%85%A5%E5%91%A2)
