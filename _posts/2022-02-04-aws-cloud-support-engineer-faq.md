---
title: "關於 Cloud Support Engineer 職位的常見問題 - AWS Cloud Support Engineer FAQs (Amazon Web Services) "
description: "Amazon (Amazon Web Services / AWS) Cloud Support Engineer 的常見問答，所有你可能會想知道的問題。"
tags: ['amazon', 'aws', 'amazon web services', 'work', 'Cloud Support', 'Cloud Support Engineer']
classes: wide
header:
  og_image: /assets/images/posts/2022/02/aws-cloud-support-engineer-faq/cover.png
  teaser: /assets/images/posts/2022/02/aws-cloud-support-engineer-faq/cover.png
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

有鑒於我實在收到太多有關於職位的詢問，我已經開始懷疑自己到底是公司的 HR 還是 Engineer。但基於這項原因，我注意到許多常見的問題，因此，希望透過簡短的篇幅列舉可能你會想知道的資訊。

{% include figure image_path="/assets/images/posts/2022/02/aws-cloud-support-engineer-faq/cover.png" alt="AWS Support FAQs" caption="AWS Support FAQs ([source](https://aws.amazon.com/premiumsupport/faqs/))" %}

## 概覽

在分享 [Amazon 的 Cloud Support Engineer 到底是在做什麼 (Amazon Web Services / AWS)](/what-is-cloud-support-engineer-doing-in-amazon) 一文後，我開始陸陸續續收到許多私訊，並且收到許多人對於對於內容的反饋，也因為這樣間接也讓更多人更加深入明白對於我們職位的認知。

很多進來我們團隊的新同事都私底下跟透露說他們曾經看過我的內容，讓我著實有點害羞。但能夠幫助到許多當時對於我們團隊職位有興趣的候選人，心裡還是十分感激且開心的。

不過，由於我能從每個詢問中感受到每個人在準備面試的疑問，在不違反 NDA 的情況下，只希望給每個想要加入 AWS Cloud Support 團隊的你能夠有更多的方向，幫助你了解有關職位的內容，甚至能夠了解這份工作是否符合你的期待。

**因此，以下內容均屬於個人自身經驗，不代表官方建議，請參考就好。**

## 常見問題

### Q: 團隊面向的客戶、在做什麼

請參考 [Amazon 的 Cloud Support Engineer 到底是在做什麼 (Amazon Web Services / AWS)](/what-is-cloud-support-engineer-doing-in-amazon)

### Q: 需要會英文嗎？面試需要具備英文能力嗎？程度需要多好？

> **要**、**會**、**True**、**Yes**

Amazon / AWS 是一家跨國的公司，在內部主要書信和溝通語言以英文為主。所以英文能力是必備的，但工程職位的英文能力可以不用說到一定要語言檢定都滿分非常強的程度。

但至少，要有能力不畏懼能跟英文母語者溝通和互動、聽說讀寫，因為除了無時無刻與其他國家的同事互動，也會有接觸英文客戶的機會需要使用全英文溝通 (或是為客戶充當翻譯)。

但如果你在中文團隊，使用英文這樣的佔比可能會是 50/50，工作中不乏還是會有使用中文與團隊成員、客戶進行溝通的機會。

(不過，我個人覺得這不是一個投遞職缺上很大的阻礙，畢竟語言這種東西也是可以練習跟訓練的嘛！)

### Q: 我發現你們的招聘職位有 Cloud Support Engineer 跟 Cloud Support Associate，但要求好像一樣，差別是什麼？

在我們團隊工作內容相同，也會接觸相同的客戶，技術跟軟實力培訓跟資源都是一樣的。

差別在於 Cloud Support Engineer **是定義技術水平在業界比較有工作經驗的工程師，因此會有大量的機會接觸困難且複雜的案例**、工作面向的對象也比較多是有規模的企業或是團隊為主，Performance 要求通常比較高 (簡單來說就是比較操，但相對的，協助複雜案例或是與客戶接觸的外部機會也通常比較多)。

Cloud Support Associate 在進入團隊後，同樣會有豐富的訓練資源幫助你累積足夠的實際客戶案例和經驗。再經過一定的技術考核後，可以經過內部 Promotion (升遷流程) 成為 Cloud Support Engineer。

### Q: 以 Big Data/Networking/Database ... XYZ ，請問面試過程中會問到 Linux (OS) 或是 Networking 相關的問題嗎 (還是偏向 AWS 產品 ABC 呢)？會問到多深入呢？

因為我們面對的客戶包含維運、開發者等不同角色，並且 AWS 的基礎建設離不開這些技術，使得 Linux 及 Networking 在我們團隊許多職缺算是基本的必備能力，但通常不會比專精 Linux 和 Networking 的專家還深入。

基於這項原因，就如同招聘說明和簡介裡面寫的，通常對於基本的作業系統知識、了解 Linux 系統的運作原理、檔案系統、維運相關等知識通常免不了。並且會需要有對基本網路除錯和 L1-L7 網路的理解 (例如：了解 TCP/IP、HTTP/TLS/HTTPS/DNS 及一般網路協定的運作)，每個不同的專業 (i.e. Database, Networking, BigData, Linux, Networking, DevOps ... etc) 基於客戶需求和技術相關性對於這個領域的標準不太一樣。換句話說，專注 Database 專業的工程師跟專注 Linux 領域專業的工程師對於 **Linux** 知識的要求定義可能有所不同。可能 Database 專業的工程師具體了解 Linux 的基本原理、知道一些基本的指令和明白檔案系統、檔案權限管理、basic troubleshooting skill；但 Linux 專業的工程師可能就要非常了解 Linux process 運作、知道如何使用 Linux 的工具更加了解系統效能、知道 kernel dump 怎麼解讀、troubleshooting 等等知識。

**每個專業領域會有基礎需要知道的基本知識，但團隊的技能樹也都是隨著客戶需求在變化，解決的問題也是日益更新。**

面試可能會基於你的工作經歷及背景會有所調整，不過，因為我們就是主要在處理 AWS 產品的問題，因此，你在 AWS 上面可以看到的服務，可以預期都是你未來有機會能碰到的。具備 AWS 產品使用經驗是好的，但如果僅僅只會使用，回到問題排查的角度可能不見得有所幫助 (因為 troubleshooting 和問題排查正是我們團隊每天在做的事情)。

**相信我，我們團隊的每個人也都在不斷的學習、拓展不同領域的知識**。基於這項理由，沒有所謂固定的「面試問題」；但對於作業系統、網路知識的了解通常仍是需要一定的掌握和深入，因為這是 AWS 基礎建設中，基本中的基本。

AWS 的面試會是實際的理論和實務問題，沒有什麼腦經急轉彎的機智問答，並且，技術面試通常十分重視對於知識的理解。不管是誰來問我，我的建議一直都是你可以先從自己擅長的部分開始準備。由於面試的人通常會是領域專家，所以可以預期有時候問題不會是你可能過去所接觸的，這就取決於你如何分析和解決問題的能力。

具體來說，就我的觀點，如果連基本的作業系統、網路概論跟網路排查都不太熟，建議把基礎功打好再試試。

通常「只會使用和操作」的級別距離這個職缺所需要的能力還有一段差距，如果你想知道為什麼這個職位對於很多技術需由上至下到底層的通透理解，只需要設想你會每天[會遇到類似以下各式各樣東西壞掉的情境](/what-is-cloud-support-engineer-doing-in-amazon)：

- “網站掛點, 救命!”
- “資料庫連不上”
- “好像有東西不太對勁並且無法正確運行”
- “為什麼我的應用程式跑了一陣子就會自己 Crash?”
- “為什麼我的生產環境 / 應用無法解析 DNS?”
- “救命, 我的應用程式 / 服務在遭遇大流量的時候會崩潰”

如果是你，會怎麼排查呢？

### Q: 面試怎麼準備 XYZ, tips ...

面試不是考試，而是讓面試者更加了解你的過程。我會推薦根據你選擇的技術領域，準備實際你過去的案例，並且分享相應的實務經驗。另外，有興趣可以根據相關領域找找相應的 AWS 服務和實際案例。不管是否在準備面試，或即使你並沒有要加入我們團隊，我認為這都有助於你更深入了解感興趣的領域，對你的工作、目前在解決的問題可能也會有一定程度的幫助。

AWS 也提供了很多免費的學習資源可以參考 (e.g. [AWS Training](https://www.aws.training/))，你會更加了解這份工作實際會接觸的內容和產品，和未來要學習幫助客戶解決的問題有哪些。

除了技術面向，Amazon 也十分重視 Culture fit，以了解你是否符合公司所想要的人格特質加入我們團隊。上上下下所有的團隊都留著 [Leadership Principles](https://www.amazon.jobs/en/principles) 血液，所有的決策、文化和流程都是基於這些準則實行，某種程度上也影響團隊成員更加負責和不斷推動進步。

因此，我相信你也一定聽過很多傳聞跟分享提到 Amazon 有多重視 Leadership Principles 並且會在面試過程中不斷的一而再、再而三問問你過去處理問題的相關情境。我可以確定這不是都市傳說，面試一定會有 Behavior question，建議可以閱讀 Amazon Leadership Principles 了解公司的文化特質。但這種東西建議根據自身經驗回答就可以了，如果你的做事風格都遵循這樣的準則，很容易在回答中呈現出來。

### Q: 我如果是非資工本科系學生，適合這項職位嗎？

我們團隊很強的成員並非全然都是資工 / CS背景，有化學工程、經濟、物理、甚至是商管背景的同事。

**就我的觀點，領域背景在 Amazon / AWS 不會是太大的阻礙影響到你的職涯發展，全然取決於你自己如何定義，希望這能增加你一點信心。**

### Q: BigData 實際工作內容有與外面資料科學家的相關工作內容相同嗎？(數據分析、資料科學家)

就我所知，BigData Team 日常處理的問題都是大規模集群客戶使用上遭遇的問題 (Hadoop, hive ... etc)。也許可能會在團隊專案上用到數據分析相關的專業解決一些團隊的需求，但不會是單純做資料分析的工作。

有部分原因由於我們團隊面向的對象有部分都是系統維運人員、開發者，所以更多的是維運、Troubleshooting 相關的知識和技能，存在一定的佔比。這通常會需要對於底層運作存在一定程度的了解，才知道怎麼解決這種連客戶也覺得棘手 (~~東西壞掉~~) 的狀況。

### Q: 未來職涯發展

由於這個角色會面向許多不同的對象：產品開發人員、客戶、Technical Account Manager、Trainer、銷售或是其他團隊，這造就這個職位的職涯路徑變得非常多元。

我們有持續深化技術能力成為資深工程師的同事，為客戶帶來更深遠的影響、或是往管理職發展；也同時有轉到其他團隊擔任不同職位的同事 (Solution Architect, Trainer ., Developer ... etc)。在 Amazon 內部，只要有能力並且存在職缺，都能申請其他團隊的機會。

在 Amazon 有趣的事情是，你可以自己定義自己的職涯藍圖，我也看過非 CS 背景的同事從帳務團隊轉到工程師職位的例子 --- **一個非技術職缺轉到技術職缺的案例**。因此這題，我想我並沒有標準答案。

如果你問我，我個人則是選擇 Relocate 到國外的團隊繼續發展並貢獻己力，因此，這取決於每個人的發展和職涯取向，但你絕對能挖掘自己的不同可能性。

## 總結和工商

希望這篇內容能夠對你有所幫助，並且有助於你解答可能對於職缺的相關疑問。

若你對於這樣的工作環境及內容有所興趣並躍躍欲試，我們仍在持續招聘優秀的人才加入我們。我同時也在 NEX Foundation 為串連台灣人才於國際舞台上的職涯發展貢獻己力，你可以透過 NEX Work 附上 CV 提交內推申請 (需註冊登入) 以引薦更多像你這樣的優秀人才，或是透過我的 LinkedIn 與我聯繫。

- [NEX Work：申請內推 (需登入)](https://work.nexf.org/companies/Amazon/referrers/tdG7zR9knf8XnpHzrpTw)

如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道。

## 看更多系列文章

{% include_relative related-post/aws-career.md %}
