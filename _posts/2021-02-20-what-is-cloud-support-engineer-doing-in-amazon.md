---
title: "Amazon 的 Cloud Support Engineer 到底是在做什麼 (Amazon Web Services / AWS) "
description: "Amazon (Amazon Web Services / AWS) 的 Cloud Support Engineer 到底是在做什麼"
tags: ['amazon', 'aws', 'amazon web services', 'work', 'Cloud Support', 'Cloud Support Engineer']
classes: wide
header:
  og_image: /assets/images/posts/2021/02/what-is-cloud-support-engineer-doing-in-amazon/amazon-bear-chen.png
  teaser: /assets/images/posts/2021/02/what-is-cloud-support-engineer-doing-in-amazon/amazon-bear-chen.png
---

AWS Cloud Support Engineer 主要面向的客戶受眾便是使用 AWS (Amazon Web Services) 服務的客戶，並為這些客戶提供訂閱制的支持計畫。(沒錯，也就是說，我們是付費的服務。) 

由於這項服務屬於訂閱制，費用可以單月從 **Developer Support Plan (29 USD)** 至 **Enterprise Support Plan (15,000 USD)** 以上不等。因此，在我的工作裡，我每天會接觸各式各樣規模的客戶。小至獨立、新創開發團隊，大至市值規模全球前 10 大的公司都有。

> (有的客戶可能這輩子只開一台 EC2 Instance，但同時，有的客戶可能單一個 AWS Account、單一個 AWS Region，就擁有數百至數千個大規格的 EC2 Instance)

{% include figure image_path="/assets/images/posts/2021/02/what-is-cloud-support-engineer-doing-in-amazon/amazon-bear-chen.png" alt="我敬重的同事 Teddy Chen" caption="我敬重的同事 Teddy Chen" %}

## 面對的客戶

{% include figure image_path="/assets/images/posts/2021/02/what-is-cloud-support-engineer-doing-in-amazon/support-plan-price.png" alt="AWS Support Plan 定價" caption="AWS Support Plan 定價 - [來源: 比較 AWS Support 計劃](https://aws.amazon.com/tw/premiumsupport/plans/)" %}

15,000 USD 光換算台幣就已經 40 幾萬，什麼服務都還沒有用就先燒一堆錢，聽起來好像傻子才去買 Enterprise Support。但很多時候，我們的客戶會十分願意花錢買我們的 Enterprise Support，你可以繼續往下看，就知道原因是什麼。

首先，AWS Support 針對不同級別的客戶提供了不同程度的響應時間：

{% include figure image_path="/assets/images/posts/2021/02/what-is-cloud-support-engineer-doing-in-amazon/support-plan-type.png" alt="AWS Support Plan SLA" caption="AWS Support Plan SLA - [來源: 比較 AWS Support 計劃](https://aws.amazon.com/tw/premiumsupport/plans/)" %}

以 Premium Support 服務來說 (也就是 Cloud Support Engineer 的工作)，一旦購買 Support Plan，最大的優勢就是你可以開 Case 進來詢問我們團隊任何有關 AWS 相關的技術問題，並會有對應的專家提供解答。但這時候購買什麼 Support Plan 的差異就出現了，如果你是 Enterprise Support 的客戶，其回答的優先權絕對是很高的，即是只是詢問一般性指導問題 (e.g. A 服務怎麼使用、使用 B 有沒有什麼需要注意的細節)，服務 SLA 最久的保證時間是 24 小時內，你一定可以獲得回覆。此外，針對嚴重的問題，會有更快的響應速度。

若屬於最嚴重的案例 (業務關鍵系統當機，通常是遭遇 service outage)，在客戶開啟案例的同時，就會有資深的技術支持工程師在 15 分鐘內響應協助客戶解決。並且持續追蹤、提供相應的技術建議，以幫助快速定位問題，並盡快幫助客戶恢復系統業務的工作。

另外，若是簽訂 Enterprise Support，Enterprise Support Plan 的客戶會擁有專有的 Technical Account Manager，不斷地幫助客戶向 AWS 內部團隊反饋問題 (像是客戶的專屬窗口)、Solution Architect Team 也會協助客戶檢視系統架構設計的可行性。

同時還會有 Concierge 團隊幫助你怎麼優化每個月的帳單、提供怎麼訂閱 RI (Reserved Instance) 的優化建議，對於 500 人以上的大規模企業，借助 AWS 的資源，總體可以幫助客戶節省非常多的費用。

並且，這某種程度上幫助降低客戶端維運團隊的複雜性，專注在發展重要功能，並交付 AWS 的專業團隊及經驗快速幫助定位、協助解決很多超難又複雜的問題，並從中與我們合作互相學習成長。這正是許多客戶選擇購買 Enterprise Plan，以幫助他們加速業務成長的主要原因。

## Cloud Support Engineer 的日常

Meet our Global Mandarin Premium Support Team at AWS:

(以下影片有點久遠，不過可以大略了解這份職位的特性)

{% include video id="EgSHXUmiuDk" provider="youtube" %}

(What's it like to work at Amazon Web Services?)

{% include video id="GC3bWcFFZTo" provider="youtube" %}

在我撰寫這篇內容的同時，AWS 已經擁有超過 100 種不同的雲端服務及相關的解決方案產品。由於不同產品有其複雜性，因此，Cloud Support Engineer 的工作主要是協助客戶在使用這些服務時所遭遇的問題給予相關的技術建議及指導。

同時，在工作中也會需要協同不同團隊 (Development Team、Solution Architect Team、Account Team / Account Manager / Technical Account Manager) 亦或者是其他團隊，幫助解決客戶所遭遇的問題。**由於 Cloud Support Engineer 最主要負責的技術是 AWS 相關的產品，每個工程師都會是相關服務的領域專家，因此，有的時候也可能是在協助 Amazon 內部其他團隊在使用 AWS 相關服務的問題。**

**在你第一天入職時，Amazon 就會有非常完善的技術訓練計畫，並且擁有累積多年的實務經驗，你還會有專屬的 Mentor。因此在前幾個月你會非常專注學習相關的知識，學習過去不同 Case 的處理方式，並在幾個月後完全的有能力處理不同規模客戶的案例。**

這些案例問題涵蓋的範圍眾多，除了一般可能詢問怎麼設定、API 怎麼使用、客戶剛入門的級別 (我們稱之為很甜的案例)，**大部分情況，我們處理的問題通常都不是 "Happy Case"，客戶會問的問題也都不是寫在文件裡面的。大部分時間我們都在挖掘真實世界的問題並且給予實質建議**，比如：

- "服務中斷"
- "網站掛點, 救命!"
- "資料庫連不上"
- "好像有東西不太對勁並且無法正確運行"
- "為什麼我的應用程式跑了一陣子就會自己 Crash?"
- "為什麼我的生產環境 / 應用無法解析 DNS?"
- "如何升級的時候不要有服務中斷?"
- "為什麼我的生產環境 / Cluster遷移到 AWS 就不能工作?"
- "救命, 我的應用程式 / 服務在遭遇大流量的時候會崩潰"

在我的工作上，得需要在短時間內學習新技術的技能，並且持續不斷的學習 (由於 AWS 團隊很多，產品迭代速度很快，往往我們都還不會之前客戶就已經在問了)。同時，很多情況客戶會混合不同服務一起使用，亦或者是大規模的集群有東西壞掉。因此，你必須要有能力剖析在複雜架構下，真正問題的核心點是什麼，並且有效進行問題的定位。

同時，很多情況下，**基於資料安全性及隱私權政策，往往我們都不會有權限存取客戶的資料、了解他實際運行的邏輯是什麼。這也意味著：我們沒有權限 SSH/登入 到客戶的環境裡面看設定、不知道客戶的 Code 是跑什麼、不確定客戶到底連接了哪些服務。所以你必須在這種情況下，還得知道如何有效的幫助客戶解決問題。**

但我們的團隊在上述的情況下，還可以明確地告訴你要收集哪些東西，然後開啟以下對話 ...：
- 哦！我從你收集回來的 kernel dump 確定是你的 Python 應用程式在 UTC 時間 XX 有問題，可能是有 memory leak，你要不要檢查一下
- 哦！你抓回來的封包明顯看得出來你的應用程式主動發 FIN 關閉連接耶，估計是超出應用程式可允許的等待時間。所以這不是 MySQL Server 關的，你要不要確認一下你語言 / Library 預設用的 timeout 時間?
- 哦！你的 Kubernetes Node 一直是 NotReady 是吧？從你收集回來的東西很明顯就是 kubelet 壞了啦！你說怎麼壞的？從我的分析可以注意到 Disk performance 不太 OK，你是不是跑了 I/O Bound 的應用啊？ .... 你說你不知道開發又寫了什麼埋坑讓你不能下班？好吧，所以你可以試試 .... etc

(我們通常不會跟客戶這樣說話，但大意大概就是如此)

**很多人可能會以為這份工作就是一般的客服人員 / 低階接線生，就我在這個團隊幾年下來，我只能說這是大大的誤解，並且抱持偏見的角度在批判這份專業性的工作。**

因此，要用其他方式描述我們的工作內容，我一直認為類似於**「急診室的醫生」**是滿貼切的說法。在客戶遭遇系統嚴重影響 (比如系統中斷、故障、service outage) 的情況。**醫生也是秉持其專業需要在短時間內判斷下一步的動作是什麼，以盡快的緩解病患所遭遇的問題，並且適當的安排正確的處理優先順序** (所以為什麼在忙碌的急診室要排隊看發燒)。在我們的工作中，也會根據客戶所遭遇的問題嚴重等級正確區分 Priority，**一直以來都視客戶關鍵性業務系統當機為優先，以幫助降低客戶遭遇系統中斷而導致營收受損的影響。**

在我們的工作中，提供了 24 小時 x 7 天 x 365 的工作模型，這意味著 Premium Support Team 在無時無刻都會有人在線上協助客戶所遭遇的問題。基於這種工作模型，我們團隊屬於跨國的工作型態，在全世界各地都有相應的團隊執行交接。因此在台灣的下午及晚上，我們會將業務移轉至歐洲及美洲時區的同事執行跟進。

我的工作中會使用中文及英文兩種不同語言協助客戶的問題。由於用中文在 AWS 服務客戶仍然是一個很稀缺的技能 **(在看這篇的你如果有訂閱 AWS Support Plan，請記得我們支持用中文開 Case，只需要在標題加上 [Mandarin] 即可！)**，所以一大部分我的團隊會協助使用中文的客戶解決他們所遭遇的問題。

另一部分仍是協助全球世界各地使用英文為主的客戶、工程師亦或是 Amazon 內部的團隊。

### 特殊活動監控

遇到 Enterprise 客戶有大型活動或是線上業務特別熱鬧的時候 (例如：Black Friday、雙 11、搶票活動 .... 等等)，這時候就是我們會特別祈禱別發生鯊魚咬斷電纜之類的事件。有時候對於非預期的大規模流量導致任何非預期情況發生時，資深的 Cloud Support Engineer 就會進場幫忙排除可能的問題是哪些、釐清可能觸及到的系統限制、幫助客戶緩解問題，降低停機時間的影響。

### Training & SME

Cloud Support Engineer 擁有完善的訓練計畫，並且有豐富的資源及經驗使得你可以接受到非常深入的技術訓練 **(強度頗高)**。在我們的組織中，同時也有 **Subject Matter Expert (SME)** 的角色存在，很多 Cloud Support Engineer 都會很努力的爭取該項角色，其代表的是某些特定 AWS 服務的領域專家，並且分佈在全球世界各地 (但一個服務如果越新，全球不到 5 位 SME 是有可能的)。

不過你通常也不需要擔心沒有人可以為你提供相關的建議。這使很多情況我們會需要跟不同區域的外國同事交談，以討論深入的技術問題。

一個 Cloud Support Engineer 要成為 SME 有一定的門檻，但總體而言，在成為 SME 前必須要累積一定數量及技術水平的 Show Case (實際的客戶案例)，並進行至少由兩位現有專家的案例審查。通過之後，會另行安排技術面試 (時間可能是半夜或是早上，通常是配合其他時區的專家)。面試的考官除了有現有專家外，還會有開發那個服務的資深開發工程師加入，全程以英文對話來測驗你對於服務的了解及深度，最終再由考官們投票看看是否一致通過。這過程真的是有很多曲折離奇的故事能寫，**但我相信 Amazon 對於工程師的技術標準要求仍是有一定的水準，所以絕對不是一個純接線生這麼簡單。**

Cloud Support Engineer 如果成為某個領域的專家，通常也會想辦法貢獻自己的知識 (透過內部文件、製作訓練教材或是幫忙其他團隊審核公開發佈的內容)，並也有機會成為 Trainer，參與跨國訓練其他地區同事的機會。

### 奧客與第三方軟體

這一定每個行業都會有的，在我的工作中，常會收到負面情緒的客人通常都是購買 Developer Support Plan 的客戶，常常會挨著說每個月要充值 29 USD 太貴、問一般性問題都只能回 Email、能不能直接登入/SSH 到我的機器幫我設？

就我的觀察，這種客戶通常也是寧可花時間耗，而不是想解決問題。通常也不願意花力氣學習 AWS，並總希望有人幫忙幹到好。這種客戶典型會問的問題也像是：為什麼我的 EC2 關了還是會一直重開，你們是不是亂動我的帳戶！！！！ 

(結果進場看個 3 秒就注意到客戶完全不知道自己在幹嘛開了 EC2 Auto Scaling)。

> 小知識：AWS 非常重視 Secuirty & Privacy，特別關注客戶存放在雲端上資料的機密性及安全性。AWS Support 沒有權限更改客戶的資料/SSH 進入到客戶的機器，我們光要看客戶一些你都覺得沒什麼的配置 (比如這個帳戶在某個區域開了幾台 EC2)，都有嚴謹的稽核制度。

通常處理一個案例，我們都得想辦法引導並還得拜託客戶提供及收集一些資訊，以幫助我們確認下一步的動作或是進行分析。對於手裡握著核彈的嬰兒來說，還是建議找找 Partner 提供的 Support 計畫吧。

此外，還有些客戶會很喜歡詢問第三方軟體的一些問題 (比如 Terraform)。由於有些第三方軟體並不是我們能夠協助，也不是由 AWS 開發，這通常已經超出 AWS 所能協助的範圍。如果你跑來問 AWS Support，我們只能竭盡所能回答。然後你總是發現，即使我們可能看一眼知道怎麼修並且提供一些可能的建議，我們只能再無奈地請你關閉 Case，並請你去找 Partner (如果是 AWS 有合作的夥伴並且涵蓋在支持清單，我們會協助轉交至對應的團隊)。

但畢竟我們並不是該產品的專家，如果你遇到 Bug，我們真的無法負責，這是一種心有餘但力不足的哀傷。

## 技術挑戰

其中，日常工作中會出現的情況之一，是我們會需要跟 AWS 開發團隊合作，以改進產品上所遭遇的問題。在我的工作上，很多時候可能是客戶遭遇了服務功能不支持、或是產品有一些已知問題。如果 Cloud Support Engineer 想到能提供其他解決方法，我們會建議客戶採取相應的行動，例如我之前在網站上發佈的幾篇內容，都是我們客戶實際遭遇，並且提供一些可用案例的例子：

- [CoreDNS(kube-dns) resolution truncation issue on Kubernetes (Amazon EKS)](/coredns-resolution-truncation-issue-on-kubernetes-kube-dns/)
- [[AWS][EKS] Zero downtime deployment(RollingUpdate) when using ALB Ingress Controller on Amazon EKS](/zero-downtime-deployment-when-using-alb-ingress-controller-on-amazon-eks-and-prevent-502-error/)

上述過程涵蓋的技術分析只是部分的案例，對於我們內部工程師在分析問題時，這通常是我們會需要從中挖掘的深度，並在你分析後發現是一些坑、或是一些不同技術之間互相依賴關係所產生觸及令人意想不到的限制，還需要嘗試提供可用建議給客戶以解決他們所遭遇的問題。但如果是要建議開發團隊修復的已知問題，同樣也會提供類似上述的分析報告，並且協同開發團隊在下一個版本中修復。

這種長期在挖掘我們家產品的 Bug，每次甩分析報告過去建議他們怎麼修 Bug 的模式，其實已經某種程度上形成良好的循環，讓我們贏得開發團隊的信任，跟開發達成良好的合作關係。(我甚至也認識很多很猛的同事是直接幫忙貢獻代碼的)

在我的團隊中，能參與這樣的過程並能從中學習成長，實在是非常有成就感的一件事情。

**因此，很多情況下，我們會需要學習用宏觀的角度分析並且切入問題核心，並且給予正確建議，而非只看單一問題亂槍打鳥，這是我認為這份工作最困難也最有趣的地方。**

## 跟開發 (SDE) 差在哪

我們跟開發團隊相同共享存取服務原始程式碼的權限。所以有的時候，比較資深或是厲害的工程師，會協助開發找出應用程式的 Bug，並且指出要修正的細節。即使在我的團隊，仍有很多同事過去的背景從事開發、維運等不同角色。但在篩選標準上，技術的水平仍與 Amazon 開發團隊擁有著一致的標準，只是這個工作角色對於編程 (Coding / Programming) 的能力並不是必須。

但往往有時候擁有開發的經驗，在執行這份工作上，也能幫助你更加有效的定位問題的可能 (若屬於應用程式的問題的話)。**與開發團隊最大的差異便是我們的工作不是在寫 code、做 feature，而是在解技術問題、每天都在 troubleshooting！(有的開發團隊工程師甚至不見得知道怎麼 troubleshooting，需要我們耐心引導協助客戶調查核心問題，這是也最有趣的部分)**

並且工作型態固定，偏向 work-life balanced。

(但這並不代表你不能開發，很多 Cloud Support Engineer 還是會自己兼著做很多好用的工具，內部還是有很多專案能幫忙做的)。

## 總結

即使擔任雲端技術支持工程師約 1-2 年，我仍然對每天幫助人們應對棘手的技術挑戰感到興奮。我還是為了有機會每天學習這些實際案例而高興，並且能夠想辦法幫助不同產業客戶很多很偉大的業務達到成功 (大規模數據運算, 購物, 手持應用, Streaming, Travling, 加密貨幣交易 ... 等)，確保客戶的服務運作在 AWS 上面保有高可用性和可靠性，以持續能運作 24 小時 x 365 天。最有趣的特別是面對一些未知問題最終發覺是個 Bug (不管是 K8s, Docker ...)。

在 AWS / Amazon，在全球真的有很多優秀的人、超級酷的 Manager 還有超強的同事。在這裡工作就像是玩一場又一場遊戲，這個遊戲需要我們花很多力氣分享很多分析報告並且嘗試打爆每一個看起來不是那麼友善的問題，只為了提供更好的產品跟技術服務給到客戶。我很高興能夠參與超級多場類似這種遊戲，並且能跟很多優秀的工程師和開發團隊合作。

如果你正在疑惑 Cloud Support Engineer 是什麼樣的工作，希望這篇的內容能夠有助於你了解我們的工作日常。

若你對於這樣的工作環境及內容有所興趣並躍躍欲試，我們仍在持續招聘優秀的人才加入我們，你可以附上 CV 並透過我的 LinkedIn 與我聯繫。

## 看更多系列文章

- [我在 Amazon 學到的 10 件事](/ten-thing-I-learned-in-amazon)
- [我是如何在還沒畢業就錄取並進入到 Amazon 工作](/how-am-I-get-into-amazon-before-graduate)
- [Amazon Cloud Support Engineer 到底是在做什麼 (Amazon Web Services / AWS)](/what-is-cloud-support-engineer-doing-in-amazon)
- [我是如何在一年內通過 AWS 五大核心認證](/how-i-pass-aws-all-five-certificate-within-one-year)
