---
title: "(2021 年底最新) 我是如何通過 CKA - 認證考試及準備 Certified Kubernetes Administrator"
description: "Certified Kubernetes Administrator (CKA), CNCF (Cloud Native Compute Foundation)"
header:
  og_image: "/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cover.jpeg"
  teaser: "/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cover.jpeg"

invalid_testing_env:
  - url: /assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cka-invalid-environment.jpg
    image_path: /assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cka-invalid-environment.jpg
    alt: "違反考試規定的環境"
  - url: /assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cka-invalid-desk.jpg
    image_path: /assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cka-invalid-desk.jpg
    alt: "違反考試規定的工作桌"

tags: ['Kubernetes', 'certificate', 'cka', 'cncf', 'cloud native']
classes: wide
---

由於我實在是太懶了，年初嚷嚷買的考試給他一個拖到快年底，決定請了幾天特休閉關唸書準備，並且花了一個下午完成了線上考試。我在 Linkedin 分享考到 Certified Kubernetes Administrator 之後，我就收到許多私人訊息，甚至有的還問有沒有試題的 dump (真的有點太莫名其奇妙了)。因此簡單透過自己的經驗，跟有想要準備這個認證而在閱讀這篇文章的你分享一下自己的考試過程。

{% include figure image_path="/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cover.jpeg" alt="Become a Certified Kubernetes Administrator" caption="Become a Certified Kubernetes Administrator" %}

Certified Kubernetes Administrator 是屬於上機類型的考試，也就是會有一定數量的情境題需要在時間內完成，並且需要透過操作實際的 Linux server 設定 Kubernetes Cluster 的一些細節。總結來說，我個人覺得比起工作會遇到的問題，Certified Kubernetes Administrator 上機題目不是說很難且複雜，但也不太容易。

我的做題過程系統還斷了幾次線，不過還是在時間內完成所有題目提早十分鐘交卷，但最後應該是掉了一題 7 分跟一題 4 分的題目，出來的成績總分是 89 分。趁現在還有一些記憶，簡短的透過一個篇幅分享有關考試的細節。

## 什麼是 Certified Kubernetes Administrator (CKA)

Certified Kubernetes Administrator (簡稱 CKA) 是 Cloud Native Computing Foundation (CNCF)，是唯一 Kubernetes 官方唯一認可的技術能力評鑑認證，CKA 認證旨在針對考核成為業界的 Kubernetes 管理員所需的技能。

如果企業想要申請 Kubernetes Certified Service Provider (KCSP, Kubernetes 認證服務提供商)，條件之一是至少需要三名員工擁有 CKA 認證。

考試屬於線上形式，並且全程會有監考官透過視訊鏡頭和監控螢幕進行監考，過程中需要透過命令行 (Command Line / CLI) 的形式完成。

### 什麼是 Cloud Native Computing Foundation (CNCF)

{% include figure image_path="/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cncf-logo.png" alt="Cloud Native Computing Foundation (CNCF)" caption="Cloud Native Computing Foundation (CNCF) - [cncf.io](https://www.cncf.io/)" %}

Cloud Native Computing Foundation (CNCF, 雲原生基金會) 的誕生與 Kubernetes v1.0 版本的釋出有關 (2015 年 7 月 21 日)。在 Kubernetes 釋出的同時，Google 與 Linux Foundation 合作組建了 Cloud Native Computing Foundation (CNCF)，並將 Kubernetes 作為種子技術的一部分。

創始過程包含 Google, CoreOS, Mesosphere, Red Hat, Twitter, Huawei, Intel, Cisco, IBM, Docker, Univa, 及 VMware 等各個廠商貢獻及草擬，至今已經有超過 450 個企業和會員支持。CNCF 旨在基於各個廠商之間成為中立的存在，協助專案開源及推廣，以打造 Cloud Native (雲原生技術) 的生態鏈。

CNCF 至今仍致力於 Github 上的快速成長的開源技術的推廣，例如 Kubernetes、Prometheus、Envoy 等，幫助開發人員更快更好的構建出色的產品。

## 考試基本資料

### 認證有效期

3 年 (CKA and CKAD  Certifications are valid for 3 years [1])

### 費用

英文考試為 `$375.00 USD`、中文版本及中文監考官的的考試是 `¥2088 RMB` (含税)。價格可能會隨著時間上漲及浮動，最新的資訊可以參考[CNCF 的考試公告頁面](https://www.cncf.io/certification/cka/)以掌握最新動態。

### 考試地點和應試要求

考試採線上形式，可以是任何地點，但對於考試環境有特定的規範。通常掌握以下原則都不會有太大的問題：

- 通常是安靜且私人的房間，不可以有其他人於房間內走動，因此咖啡廳等公開場所是不允許的
- 燈光必須明亮能夠清楚提供照明、視訊監考時能看得清應試者的臉和周圍的環境
- 視訊鏡頭拍攝應試者背後不可以有窗戶或是強烈的光源 (避免反光)
- 考試桌面保持整潔，不可以有任何紙張、衛生紙、筆等雜物
- 乾淨的牆面，視訊鏡頭拍攝的四周牆面不可以有紙張或是海報等標記 (一般裝飾通常不是太大的問題，以監考官判定為主)
- **考試過程可以喝水，但是水杯必須是透明的不包含任何標籤，液體也需要是透明的水**
- **應試過程應試者的臉必須置於鏡頭中央，並且全程可見**

(具體的規範可以參考 [1] - What are the Testing Environment requirements to take the exam? )

### 測驗語文、形式、題數及時間

- 考試語言：英文、簡體中文和日文 [1]
- 考試形式：上機測驗
- 考試題數：約 17 題 (包含 7 分和 4 分等不同級距的問題)
- 考試時間：120 分鐘 (Candidates are allowed 2 hours to complete the CKA, CKAD and CKS Exams)
- 及格分數：66% 分通過 (For the CKA Exam, a score of 66% or above must be earned to pass)

## 考試報名

我是考英文並且安排英文考官，介面直接透過 The Linux Foundation 的頁面完成購買:

- [The Linux Foundation - Certification / Certified Kubernetes Administrator (CKA)](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)

完成購買不代表預約考試，買完後還需要透過 Training Portal 並且根據指示跳轉到 PSI 安排考試：

- [Training Portal](https://trainingportal.linuxfoundation.org/learn/)

如果是中文考官可以透過 The Linux Foundation 官方中文頁面並且根據指示進行購買：

- [https://training.linuxfoundation.cn/certificates/1](https://training.linuxfoundation.cn/certificates/1)

購買後可以在一年內安排考試，可以在不同時區跟地區應考，如果第一次沒通過，包含考試有一次免費重考的機會 (Free re-take)。

## 考試實際過程及注意事項

在實際過程中會有幾個重要的注意事項：

- 不可以用手嗚住嘴巴
- 考試過程會開啟麥克風聆聽現場的環境聲音。因此做題過程不可以把題目唸出來、碎碎念 (避免有複誦題目獲得答案的嫌疑)
- 過程中只能開啟瀏覽器並且會共享螢幕、不可以使用任何軟體 (包含筆記本程式)，只能用考試系統於瀏覽器中彈出的 Notepad (用 javascript 彈出來) 進行紀錄
- 為了要控制作答時間，有時候掃過一遍題目覺得太耗時可以加上標記回頭作答，但標記 (Flag) 作答的功能不會保留 (這意味著在斷線之後再回來不會紀錄)。我自己在這邊有點小被雷到，建議紀錄直接透過系統的 Notepad (這個有斷線實測會保存)
- 考試過程必須關閉所有應用程式及瀏覽器分頁。雖然是 Open Book，但只能保留一個考試介面和一個允許頁籤用以查看手冊，包含：
  - `https://kubernetes.io/docs/`
  - `https://github.com/kubernetes/`
  - `https://kubernetes.io/blog/`
  - 這包含不同語言的頁面，例如：`https://kubernetes.io/zh/docs/home/`
  - 參考 What resources am I allowed to access during my exam? [1]

### 考試開始前

建議在開始前至少 1-2 天請根據考試 Checklist 完成每一項檢查，包含：
- 姓名與英文 ID 文件相符 (護照)
- 安裝規定的 Chrome 瀏覽器插件 - Innovative Exams Screensharing (這部分根據考試 Checklist 安裝即可，用於分享電腦的所有螢幕)
- 確保系統合乎考試環境 (穩定的網路)
- 有視訊鏡頭、麥克風 (用筆電有內建的就沒什麼問題，只要注意可以在展示護照的時候可以正確對焦驗證身份)

要準備的紙本材料有：
- 與你註冊英文姓名相同的護照 (Passport) 用以核對身份
- 可以準備有英文姓名的 Secondary ID 以備用查核身份: 信用卡 / Debit Card / 員工識別證 / 學生證 ... etc.
- (參考 What are the ID requirements to take the exam? [1])

建議 15 分鐘前進入考試介面，這幾十分鐘是用來檢查環境的。監考官全程會透過 Chat 與你對談 (英文)，監考官可以看得見你，但是你看不見他，並且只透過 Chat 對話。

過程中監考官會要求需要緩慢的旋轉鏡頭以環視四周環境，在我的考試中，監考官還特地我要求我拍攝桌子底下的空間，以確定沒有任何雜物和紙張。

以下是一些工作區域的注意事項：

- 工作區域和工作桌下的所有物品都要清空
- 可以使用外接螢幕，桌上僅可以有相關的連接線等。這個在監考官透過鏡頭巡視的時候就會一併檢查。以下是一些範例幫助你了解環境的一些注意事項：

{% include gallery id="invalid_testing_env" layout="half" caption="違反考試規定的考試環境 (考試環境背後有會反光的窗戶, 桌上有雜物)" %}

{% include figure image_path="/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cka-validated-desk.jpg" alt="允許的考試環境" caption="允許的考試環境" %}

在考試檢查環境過程考官會要求你分享所有畫面 (包含外接的螢幕)，考試前記得關閉所有應用程式和清空瀏覽器頁面。

如果是 Mac，會要求你按下 `Option` + `Command` + `Esc (Escape)` 以檢視你是不是有開啟其他程式，如果有，會要求你透過該方法關閉額外的應用程式並且只保留 Chrome。

[![檢視 Mac 系統中開啟的應用](/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/mac-force-quit.jpeg)](/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/mac-force-quit.jpeg)

在考試過程需要注意背後是不是有窗戶會影像到鏡頭反光，並且清除桌上和桌下的雜物。我個人在考試過程中把窗簾拉起來，監考官在檢視過程看過確定沒什麼問題。

### 考試中

考試過程會主要使用 Chrome 瀏覽器完成考試，會是一個線上的 Shell 介面可以輸入命令完成題目。考試的視窗會像是以下：

{% include figure image_path="/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/lf-exam-ui.png" alt="CKA - PSI 實際考試介面" caption="CKA - PSI 實際考試介面 ([詳細訊息](https://docs.linuxfoundation.org/tc-docs/certification/lf-candidate-handbook/exam-user-interface))" %}

如果中途遇到網路問題斷線了，考試介面會凍結，這時候建議嘗試透過介面的重整按鈕刷新，不行再重新整理頁面。我斷了快三次線，斷線的時候考官會保留考試時間跟 Session / VM (tmux session 也還會在)，並且等一切確認 (正確分享畫面、視訊等) 才重新釋放考試介面，所以不用太緊張。

要注意考試的環境在不同系統中 Copy-Paste 的按鍵不太一樣 (我使用的是 Mac，所以沒什麼差異)：

- **For Linux**: select text for copy and middle button for paste (or both left and right simultaneously if you have no middle button).
- **For Mac**: ⌘+C to copy and ⌘+V to paste.
- **For Windows**: Ctrl+Insert to copy and Shift+Insert to paste.

考試過程會完全分享螢幕，因此要記得只能開啟一個瀏覽器視窗檢視 Kubernetes 文件 (**要注意 Kubernetes Forum 是不能瀏覽的**)。因為時間有限，建議 yaml 能用 kubectl 產生就用命令產生，文件能複製貼上的就複製貼上。考試環境有提供 tmux，所以我在考試過程斷線的時候十分平滑沒什麼影響，以下是一些 VM 環境中有提供的工具：

- `kubectl` with kalias and Bash autocompletion
- `jq` for YAML/JSON processing
- `tmux` for terminal multiplexing
- `curl` and `wget` for testing web services

要注意考試環境也不允許透過用 vim 等方式在終端介面瀏覽網站 (包含用 wget 獲取一些三方的資源可能也會觸及到考試規範)。

因為全程監考官看得到你但是你看不到他，所有指示和問題都可以透過系統介面的 Chat 視窗對談 (考官不會回答任何跟考試題目相關的疑問)。

過程中可以透過考試介面的 `Exam Control` > `Request Break` (我個人是沒用到，但應該沒記錯) 請求休息去上廁所，但要注意時間會繼續計算。

### 完成考試

考官通常會於剩下 30 分鐘左右透過 Chat 提醒你，不過，如果你跟我一樣遭遇到系統斷線，考官通常會提醒系統中的計時器截止後仍然還會有剩餘的時間。

如果你提早做完，可以透過考試介面的 Exam Control 提早交卷。考試完成後，成績會於 24 小時內通過 Email 通知。因為是上機考試，部分可能會需要人工審核，我自己等候的時間有點長，等到隔天才有成績。這時候就放輕鬆不用傻傻的一直刷了，36 小時後如果還是沒有收到可以透過 PSI 或是發信詢問有關考試的進度。

{% include figure image_path="/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cka-exam-score.png" alt="CKA - 考試結果和分數" caption="CKA - 考試結果和分數" %}

(考試結果如果有通過，會一併核發認證 ID 並且可以在 Exam Checklist 裡面看到總成績。考試結果不會告訴你錯了哪些題目，不過，我們不求 100 分，有求過就好。)

## 考試重點及準備

### 1. 掌握考試大綱

如果你正在準備 Certified Kubernetes Administrator (CKA) 考試，那 Exam Curriculum 絕對是不可以錯過的一項環節。在裡面會定義 CKA 考試於不同 Kubernetes 版本考試項目的權重，並且會隨著時間更新。所以千萬別忘記考前花點時間了解一下考試項目的大綱：

- [CKA Curriculum](https://github.com/cncf/curriculum)

我在 Kubernetes v1.22 開始 troubleshooting 問題的佔比達到 30%，不過對於每天都在做 troubleshooting 工作的我來說，這並不是一個太難的項目 (e.g. [CoreDNS(kube-dns) resolution truncation issue on Kubernetes (Amazon EKS)](/coredns-resolution-truncation-issue-on-kubernetes-kube-dns))。

但如果你不是擅長 troubleshooting 的問題，推薦必須要試著完成一次 etcd backup 及復原的操作、修復 Node 沒辦法加入 Kubernetes cluster 的問題等，因為我在我的考試中還真的遇到了。

### 2. 熟悉基本命令

考試過程中不會有 Racher 之類的 UI 介面可以讓你懶人操作，所以推薦熟悉 kubectl 的操作以及 cheat sheet 上一些有趣的用法：

- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

通常也會需要熟悉一些基本的命令：

- Create Pod / Service / PV / PVC / ConfigMap / Secret / deployment ... etc
- Create service account / RBAC (Role-Based Access Control)
- 知道怎麼設定 Port / 暴露服務
- 知道 Kubernetes DNS 解析機制
- 知道怎麼擴展 Pod 數量 (`kubectl scale`) / Rolling update
- 知道怎麼替換、升級和更新節點 ([`kubectl drain`](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/))
- 查看 container / Pod logs

考試的時候你也可以開啟 kubectl cheat sheet 輔助，就不需要全部記憶。但由於考試過程分秒必爭，我個人覺得 kubectl 工具的 help 輸出說明十分詳細，如果可以的話推薦可以習慣使用 `-h/--help` 直接拉出具體的用例：

```bash
kubectl <ACTION> -h
```

如果你想了解更多，網路上也有 CKA 相關的資源，這是我覺得不錯的文件：

- [Core Concepts](https://github.com/chadmcrowell/CKA-Exercises/blob/master/core_concepts.md)

網路上也有人推薦說要用 `kubectl explain` 來檢視用法，說實話我個人完全沒有用到，如果你習慣怎麼在 Kubernetes docs 上查文件，我覺得這個不是必須的項目：

```bash
kubectl explain <resource>.<key>
```

在檢查 service account 類型的問題，另一個額外的技巧是用 `kubectl auth can-i` 檢查權限：

```bash
kubectl auth can-i create deployments --namespace dev
```

- [Checking API Access](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access)

但如果你喜歡硬派一點也覺得時間很多的話，你可以額外學習 Kubernetes API 的操作和用 curl 直接模擬：

```bash
# Point to the internal API server hostname
APISERVER=https://kubernetes.default.svc

# Path to ServiceAccount token
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

# Read this Pod's namespace
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)

# Read the ServiceAccount bearer token
TOKEN=$(cat ${SERVICEACCOUNT}/token)

# Reference the internal certificate authority (CA)
CACERT=${SERVICEACCOUNT}/ca.crt

# Explore the API with TOKEN
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api
```

- [Accessing the Kubernetes API from a Pod](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/#without-using-a-proxy)
- [Access Clusters Using the Kubernetes API](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)

### 3. 熟悉使用的技術

我在準備階段花了約 2-3 週看完 A cloud guru 的 CKA 課程並且完成所有的練習題目 (有的比較不熟的完成 2-3 次)：

- [Certified Kubernetes Administrator (CKA)](https://acloudguru.com/course/certified-kubernetes-administrator-cka)

基本上題目的範圍都不會偏離考試範圍太遠，只需要熟習及練習做題的速度就可以了，避免在考試答題過程花太多時間一直閱讀文件。很多人都說有空的話可以刷 [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way)，但說實話我自己在準備的過程沒有那麼多時間安排真的去一個一個練習、完成這個額外的工作。考下來我也覺得這也不是一個必須的項目，但如果你熟悉的話，會對 Kubernetes 底層的運作有更深層次的理解。

我個人覺得 killer.sh 的題目特別有用，難度特別高，貼近我自己日常工作會遇到的部分問題，做起來特別有感覺。如果你可以熟悉 killer.sh 的問題並且了解、檢討怎麼作答的話，我認為考試不會有太大的問題，因為考試的難度比這個簡單許多：

- [killer.sh - Kubernetes Exam Simulator](https://killer.sh/)

在 Exam Checklist 上會提供兩次免費的 killer.sh (Kubernetes Exam Simulator) 模擬環境的訪問，所以可以不用先購買。我個人考前 3-4 天才打開來做一次，如果你時間充裕，推薦可以在考前 1-2 週左右打開來練習用好用滿。

### 4. 熟讀文件

我會推薦除了練習命令和做題，剩下的時間就是熟讀 Kubernetes 官方文件 ([https://kubernetes.io/docs](https://kubernetes.io/docs))。熟悉文件提供的搜尋功能，可以仔細的閱讀常見的問題並且參考相關的 yaml 文件、描述，並且知道可以用哪些關鍵字把文件掉出來。

我自己的經驗是這是一個很有效的一個準備方式，在考試過程中可以很快速的透過關鍵字搜尋直接找到我要的答案。如果你習慣透過書籤 (Bookmark) 儲存紀錄，也可以在考試前建立對應的書籤資料夾，在考試過程中於另開的一個分頁中直接從書籤中調閱。

一些基本的使用相關的文件我這邊就不贅述，以下是幾個我在考完後認為值得花點時間閱讀跟時間操作的文件：

- [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
- [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

### 5. 了解 JSON Path & yaml

kubectl 的 selector 通常可以善用 json path 完成許多工作跟篩選，如果你熟悉這個技巧，在考試的時候，遇到要做排序、或是根據題目要求輸出格式，會非常有幫助。如果有時間，我會推薦可以上完在 kodekloud 上約幾十分鐘的免費課程幫助你建立相關的知識：

- [JSON PATH Quiz- FREE COURSE](https://kodekloud.com/courses/json-path-quiz/)

### 6. 其他

基本上以上幾點如果有過一遍我想要及格不會是太大的問題，剩下的就是平常心及多熟悉 CLI 操作了。

以下是一些我在考試過程中的懶人技巧 (我的考試環境是 Kubernetes v1.22，就我的認知下列的技巧於各個版本中不會有太大的差異，但還是可能要注意是否於後續的版本中仍適用)：

#### 在考試過程設定命令快捷 (用 `k` 取代 `kubectl`)

```bash
alias k='kubectl'
alias kn='kubectl -n kube-system'
```

列出所有 namespaces 底下的所有資源

```bash
alias ka='kubectl get all --all-namespaces'

# 或使用 -A 是一樣的效果
k get all -A
```

#### 善用 kubectl 產生 yaml 文件、建立資源

考試的作答時間非常緊湊，推薦可以在練習過程熟悉用 `--dry-run` 直接產生許多不同種類物件的 yaml 描述文件，可以節省許多找文件和複製貼上的時間：

```bash
kubectl run <POD_NAME> --image=<IMAGE_NAME> --restart=Never -n <NAMESPACE>
kubectl expose <deploymentname> --port=<PORT> --name=<SVC_NAME>
```

```bash
kubectl create <....> --dry-run=client -o yaml
```

產生 DaemonSet 可以用 Deployment 修改即可：

```bash
kubectl create deployment --dry-run=client -o yaml
```

#### 熟悉基本的 shell script 和複製貼上 yaml 的技巧

我個人覺得善用 cat 讀取標準輸入 (stdin) 的方式非常有用，我在考試的時候用這個方式無痛複製貼上許多 yaml 文件，例如：

```bash
cat << EOF > pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
EOF
```

如果會一些 shell script 可以在一些設計執行 command 的階段輕鬆的完成任務，但如果會忘記，推薦可以詳細閱讀 Kubernetes 相關的文件並且知道哪裡可以在考試過程中參考，例如：

- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

```yaml
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
```

## 總結

以上就是有關我個人準備 Certified Kubernetes Administrator 考試的簡略分享。在這篇內容中，我與你分享了有關 Certified Kubernetes Administrator 具體的考試注意事項、考試內容和相關的應試細節。並且具體提出幾個考試重點準備相關的 tips，同時列舉了數項實際學習的建議。

希望在閱讀這篇內容的你可以順利完成認證，並且拿到屬於你的電子證書。如果你覺得這樣的內容有幫助，也別忘了在底下按個 Like / 留言讓我知道。

{% include figure image_path="/assets/images/posts/2021/11/cka-cncf-certified-kubernetes-administrator-study-guide/cka-cert.jpeg" alt="Certified Kubernetes Administrator - Eason" caption="Certified Kubernetes Administrator" %}

## References
- [1] [Frequently Asked Questions: CKA and CKAD & CKS](https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks)
- [2] [Important Instructions: CKA and CKAD](https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad)
