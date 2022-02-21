---
title: "AWS Firecracker 論文導讀：一個小孩才做選擇，我兩個全部都要的 VMM"
description: "AWS Firecracker 是一款由 AWS 開源的輕量級虛擬化運行環境。這項技術使得運行的執行環境具備傳統虛擬機的安全性，提供優化的執行效能和資源利用。在這篇內容中，我將與你分享在閱讀完 Firecracker 論文後具體的導讀細節。"
tags: ['amazon', 'aws', 'amazon web services', 'firecracker', 'virtualization', 'Lambda', 'severless', 'AWS Fargate']
classes: wide
header:
  og_image: /assets/images/posts/2022/02/firecracker-paper-reading/cover.png
  teaser: /assets/images/posts/2022/02/firecracker-paper-reading/cover.png
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

AWS 公開 Firecracker 專案已經至少 1-2 年的時間，也許你多少都聽過這項技術。然而，究竟 Firecracker 是什麼，我想可能你也仍然一知半解。有鑒於我認為學術類型的內容有時不容易讓人明白，因此，我希望可以透過以下的篇幅，分享我自己對於閱讀 Firecracker 設計論文的一些理解。

由於我個人沒有受過正式的學術訓練，因此，如果有專家願意提供任何見解，也請不吝給予指正及建議。

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/cover.png" alt="Firecracker 概覽" caption="Firecracker 概覽 ([source](https://firecracker-microvm.github.io/))" %}

## 概覽

在閱讀完 Firecracker 的相關論文後，我對於這個專案的結論，**簡直可以說是一個小孩才做選擇，我兩個全部都要的設計**。

在論文中提到，設計 AWS Firecracker 由於在權衡 Hypervisor-based 和 Linux container 虛擬化技術之間產生的相容性、安全性的優缺點，兩者取其一似乎都無法滿足 AWS 基礎建設所需滿足的工程目標。因此，Firecracker 決定打破這樣的抉擇，擔任起 VMM (Virtual Machine Monitor) 的角色，並且引入相關的各種現有功能機制的優點，以滿足運算虛擬化的設計需要。

> Implementors of serverless and container services can choose between hypervisor-based virtualization (and the potentially unacceptable overhead related to it), and Linux containers (and the related compatibility vs. security tradeoffs). **We built Firecracker because we didn’t want to choose.**

目前 AWS 已經將 Firecracker 導入至兩個公開的無伺服器 (Serverless) 服務：[AWS Lambda](https://aws.amazon.com/lambda/) 及 [AWS Fargate](https://aws.amazon.com/fargate/)，並且支援數百萬的用戶和單月萬億級別 (trillions) 的請求，以下將具體描述更多 Firecracker 相關的細節。

(相關的 Paper 原文和我自己畫的重點請參考[^firecracker-paper-note])

## 基本名詞釋義 (Terminology)

由於 Firecracker 屬於一種作業系統虛擬化技術的延伸，其中涉及 Linux 作業系統虛擬化的諸多細節。因此，首先在閱讀這篇內容之前，必須先了解基本的一些概念和名詞釋義：

### Hypervisor

Hypervisor 可以視為用於管理虛擬機器 (Virtual Machine) 的軟體、系統或是韌體。使用虛擬化技術允許我們在單一個電腦上運行多個不同的系統、甚至是可能不同的作業系統 Kernel，並且將其放置於一個虛擬的運行環境中 (Virtual Machine)。因此，Hypervisor 的目的就是用來管理這些虛擬機器，通常，用來執行一個或多個虛擬機器的電腦稱為宿主機 (Host)，這些虛擬機器則稱為客戶機 (Guest)。

Gerald J. Popek 及 Robert P. Goldberg 在 1974 提出了兩種類型的 Hypervisor [^virtualization-architectures]，分別為 Type 1 和 Type 2：

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/hypervisor-types.png" alt="Hypervisor 類型" caption="Hypervisor 類型 ([source](https://en.wikipedia.org/wiki/Hypervisor))" %}

因為在虛擬機器中，安裝了一個 Guest OS 並不意味著就能直接使用 Host OS 的所有資源 (例如：磁碟寫入、CPU 時間、I/O 等操作)。通常，Hypervisor 會實作「模擬」這些裝置讓 Guest OS 以為能夠使用，但實際上仍交由虛擬化技術實際將這些操作轉譯、排程交給 Host OS 處理。

- **Type 1** 的 Hypervisor 通常需要硬體和 Kernel 支援，因為通常能充分交付使用硬體操作，而無需透過 Hypervisor 為 Guest OS 執行作業系統各種操作的 (syscall) 轉譯。這也通常意味著，執行的效能也比較高。常見的實作如：Xen 和 Linux KVM。
- **Type 2** 軟體會運行於主要的 (Host OS) 並且可能會以一般的軟體應用形式運行，通常執行效率比 Type 1 來得低。常見作業系統層級的軟體類似於 VMWare、Vritual Box Hypervisor 軟體。

### VMM (Virtual Machine Monitor)

基本上與 Hypervisor 相同，如同他的名字一樣 (Virtual Machine Monitor)，VMM 設計的目的就是用於建立、監控、管理和捕捉跑在虛擬機器中的 I/O 操作 (磁碟寫入、網路吞吐等)。

### QEMU

[QEMU](https://www.qemu.org/) 是一個開源的 VMM，由於 QEMU 的架構由純軟體實現，並且處於 Guest machine 與 Host machine 擔任中間者角色，以處理 Guest machine 的相關硬體請求，並由其轉譯給真正的硬體，使得其存在一些效能問題。

### KVM

KVM (Kernel-based Virtual Machine) 是一種 Linux Kernel 支持的虛擬化技術，可以將 Linux Kernel 轉換成一個可用的 VMM 並將系統轉換為 Type 1 (bare-metal) 類型的 Hypervisor，使得你可以在 Linux 系統上運行多個隔離的虛擬環境 (VM)。KVM 一直是 Linux Kernel 設計的一部分，並且存在於主流的 Linux Kernel 版本中。因此，由於屬於 Linux Kernel 支持功能的一部分，通常可以使用接近原生系統的相應執行效能處理對應的 I/O 操作。

### crosvm

[crosvm](https://github.com/google/crosvm) 為 Google 的一項開源專案 (Chrome OS Virtual Machine Monitor)，用於 Chrome OS 執行虛擬化機制的操作，基於 Linux KVM Hypervisor 實現虛擬化技術，並且用於 Android、Chrome OS 為基礎的系統中。與 QEMU 相比，它並不直接模擬實際的硬體裝置，反之，它採用了 Linux 支持的半虛擬化的裝置標準 ([virtio](https://developer.ibm.com/articles/l-virtio/)) 來模擬虛擬機中相關的裝置。Firecracker paper 中具體提到了實作中採用了以 crosvm 作為基礎核心背景修改。

### Cgroup (Control Group)

cgroup 是 Linux Kernel 一項支持的功能，主要可以用來限制運行在容器執行環境中的資源使用 (例如：CPU、Memory 和磁碟讀取寫入等)。cgroup 同時也被大量運用在 Linux container 的技術中，例如：Kubernetes、Docker 等。

在 Firecracker 中，提及了基於不信任 Guest OS 對於資源控制的行為。這是由於 Guest OS 屬於客戶控制的一部分，並無法預期其是否能依照合理的使用行為運行，因此，Firecracker 也採用了 Linux 本身支持的功能及 cgroup 等機制，限制了 VMM 和各個虛擬機器總體可用的資源。

### Seccomp

seccomp 是 Linux Kernel 支持的一項功能，用來限制在容器中運行的 process 可以呼叫的系統方法 (syscall)。可以想像就像是允許使用特定 Linux function 的白名單，在 process 的直接階段僅允許特定系統呼叫操作。

同樣的機制也被實踐在一些容器虛擬化技術中，例如 Docker 定義的預設 [seccomp profile](https://github.com/moby/moby/blob/b0806bdb03f0843267b34f50142d5f52ddd68757/profiles/seccomp/default_linux.go#L51-L390)。

## AWS Firecracker 是什麼

在過去，AWS 主流的 Serverless 服務提供了 AWS 客戶另一項託管運行應用的選擇 - 用戶不需要再自行管理底層運作機器和安全性修補的工作。

最著代表性的 AWS 服務便是 AWS Lambda，如果你不知道 AWS Lambda 是什麼，[AWS Lambda](https://aws.amazon.com/lambda/) 是一種無伺服器 (Severless) 運算服務，使用者可以直接上傳你的程式碼並且選擇對應的規格運行，你無須在煩惱需要使用什麼樣的硬體規格以及為維護工作煩惱。

同時，也提供在大規模的應用情境中隨著用量可以動態擴展的優勢。然而，在 AWS Lambda 服務剛釋出時，其採用了 Linux Container 用以隔離不同客戶的執行環境 (類似於 Docker 相應的技術)，然而，這樣的機制除了可能在客戶使用運行執行環境存在限制 (需使用依賴 Host OS 支持的 Kernel 版本指令集)，也因為同時共用相同的 Kernel，也可能存在部分安全性風險。

> When we first built AWS Lambda, we chose to use Linux containers to isolate functions, and virtualization to isolate between customer accounts. 

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/model-compare.png" alt="虛擬化架構" caption="AWS Lambda 虛擬化架構 (左) 為之前的設計 (右) 為採用 KVM & VMM 技術的設計" %}

因此，在這篇 Paper 中，主要提到了 firecracker 評估使用虛擬化技術設計時存在六項重點考量：

- **Isolation**: 需要具備安全的隔離環境使用戶在運行應用時，能避免資料洩漏、非法提權等安全性問題
- **Overhead and Density**: 為了盡可能使單一機器的硬體資源應用最大化，該技術需要能夠提供運行至少數千個 microVM 執行環境 (Lambda function)
- **Performance**: 執行效能要能貼近使用原生實體機器 (Bare-metal) 的執行效能，換句話說能降低因為硬體指令轉譯的執行時間
- **Compatibility**: 需要能夠讓用戶執行應用時運行 Linux 支持的函式庫和執行檔，以客戶無需進行程式碼修改或是重新編譯
- **Fast Switching**: 能夠盡可能快速的啟動執行環境 (microVM) 和清除執行環境
- **Soft Allocation**: 存在資源動態調整機制，能夠允許虛擬執行環境分配額外的 CPU、Memory 等資源。讓每個應用僅可使用消耗他所需要的資源，而不是有權使用的系統資源

在這樣的條件下，論文 2.1 中的細節便是在具體討論和評估數項現有的虛擬化技術，包含：

- Linux container: Linux Kernel 本身支持的容器化技術，使 Linux process 存在於獨立的執行環境 (namespaces)，並達到 process-level 的隔離，包含 user IDs (uids), process IDs (pids) 及 network interface，並且能夠利用 chroot 機制隔離執行的檔案系統。同時，利用 seccomp-bpf 更可以達到 process 執行系統呼叫的限制 (syscall)。在相關的研究中，一般啟動 Ubuntu Linux (15.04) 版本的安裝需要 224 syscalls 及 52 個獨立的 ioctl 操作。
- Language-Specific Isolation：例如 JVM 透過劃分 Heap size 和虛擬執行環境，於記憶體空間分配支持的虛擬執行環境
- 硬體和 Kernel 支持的主流虛擬化技術：Intel VT-x、KVM、QEMU。在論文中提到常見的 KVM 和 QEMU 組合通常增加了執行虛擬化上的複雜度，由於 QEMU 專案本身包含了大於 140 萬 (1.4 million) 的程式碼，並且至少需要 270 個系統呼叫操作 (syscall)，若使用這項基礎再疊加使用 KVM，則會再另外增加了 120,000 行程式碼。

因此，在評估和主流虛擬化技術比較這樣的背景下，AWS Firecracker 借鑒了許多解決方案而在眾多項目中選擇一個適當的平和。同時，基於 AWS 內部許多團隊，維運基礎架構都採用 Linux 系統，促使 Firecracker 在設計的哲學上的這項決定。更重要的是，Firecracker 更之所以遵循沿用 Linux Kernel 本身就支持的技術，而不是重新實作替代它，正是因為這些功能行之有年，並且具備高質量、成熟的設計 (例如：scheduler、TUN/TAP network interface)，也能讓 AWS 原本的團隊使用熟悉的 Linux 工具和維運流程執行除錯。例如：採用 `ps` 即可列舉機器上運行的 microVM，其他 Linux 本身支持的工具 (`top`、`vmstat` 甚至是 `kill`) 均可以在預期的操作下管理 Firecracker。

基於這項原因，Firecracker 使用了 KVM 作為主要的虛擬化執行基礎，並且實作 VMM (Virtual Machine Monitor) 元件以滿足管理 KVM 執行環境的需要。

> Our other philosophy in implementing Firecracker was to rely on components built into Linux rather than re-implementing our own, where the Linux components offer the right features, performance, and design

### KVM 在這樣的基礎下做了哪些事？

執行硬體層級的虛擬化 (HVM) 及資源分配，例如：CPU、處理記憶體管理、分頁 (Paging) 等。

## AWS Firecracker 實作

在 Firecracker 的實作中，以 Google crosvm 作為基礎，移除了大量不必要的裝置，例如：USB、GPU 以及 9p 檔案系統協議 (Plan 9 Filesystem Protocol)。在這樣的基礎下，Firecracker 以 Rust 語言為主增加了額外約 2 萬行的程式碼；同時修改了約 3 萬行的程式碼並且開源公開。

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/firecracker-layers.png" alt="Firecracker 架構" caption="Firecracker 架構" %}

Firecracker 同時模擬了有限的一些 I/O 裝置，例如：網路卡、磁碟、序列端口 (serial ports)、i8042 支持 (PS/2 鍵盤的控制器)；與 QEMU 相比，QEMU 相對複雜許多，其支持多餘 40 種不同的裝置，包含 USB、影像和音訊裝置。

更細部的設計架構如下：

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/firecracker-design.png" alt="Firecracker 細部設計架構" caption="Firecracker 細部設計架構 ([source](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md))" %}

在 Firecracker 中採用了 virtio 作為網路和磁碟裝置的模擬，其中大約佔 Firecracker 1, 400 行 Rust 程式碼。同時 Firecracker 也提供了 REST API 設計，使其能夠使用 HTTP 的用戶端直接與其互動 (例如：`curl`)。

總結來說，Firecracker 旨在提供以下機制 [^how-firecracker-work] [^firecracker-open-source-innovation]:

- 設定 KVM
- 提供裝置模擬，包含模擬 SSD、NIC (網卡) 等，即使沒有那麼多裝置在實際的硬體 (AWS Lambda Host) 上，使用 virtio 使得其可以虛擬化這些裝置
- 執行環境效能的隔離 (採用 cgroup)
- 為 Serverless 提供優化的效能 (5-8 Mb 的 Linux 啟動時間可以縮短為 100ms 以內、更微小的 Kernel 更可以縮短為 5ms)

### 資源限制 (Rate Limiter)

在 Firecracker 中的硬體裝置涵括了限制配額的機制，包含可以限制 Disk IOPS (I/O Per Second)、PPS (Packets Per Second for network)。在 Firecracker 提供了使用 API 設定 microVM 可用的資源請求，包含 CPU、磁碟 I/O、網路吞吐等。

其資源限制機制採用 `virtio` 本身支持的資源限制功能，以網路裝置來說，可以是以下的配置機制 ([`rx_rate_limiter`](https://github.com/firecracker-microvm/firecracker/blob/f35f324b84ce5c78dcf706cd97117bca41485b34/src/vmm/src/vmm_config/net.rs#L30))：

```bash
PATCH /network-interfaces/iface_1 HTTP/1.1
Host: localhost
Content-Type: application/json
Accept: application/json

{
    "iface_id": "iface_1",
    "rx_rate_limiter": {
        "bandwidth": {
            "size": 1048576,
            "refill_time": 1000
        },
        "ops": {
            "size": 2000,
            "refill_time": 1000
        }
    }
}
```

### 安全性 (Security)

為了實踐安全性的最佳化，Firecracker 在部署階段需要充分避免一些因為 Linux kernel 或虛擬化技術可能帶來潛在的安全性問題，例如：Intel Meltdown、Spectre、Zombieload 等安全性漏洞。因此，在生產環境中，為了解決這項顧慮，Firecracker 實踐了幾項部署重點：

- 關閉 Symmetric MultiThreading (SMT, aka HyperThreading)
- Kernel Page-Table Isolation, Indirect Branch Prediction Barriers, Indirect Branch Restricted Speculation and cache flush mitigations against L1 Terminal Fault
- 啟動部分 Kernel 參數包含 Speculative Store Bypass mitigations
- 關閉 swap 和 samepage merging
- 避免共享檔案 (解決 timing attacks like Flush+Reload and Prime+Probe)
- 以及使用建議的硬體設備以解決 RowHammer 攻擊技術

相關的 Firecracker 生產環境部署建議同時列舉於以下文件中：

- [Production Host Setup Recommendations](https://github.com/firecracker-microvm/firecracker/blob/cd29c64b35f694a5a442a7778b7c1463551bc1e2/docs/prod-host-setup.md)

同時為了避免 Firecracker VMM 執行操作的過程出現任何非預期行為 (例如：安全性漏洞允許植入惡意代碼)，在 Firecracker 中實現了使用另一層沙箱 (Sandbox) 提供額外隔離的保護。在 Firecracker 的設計稱之為 **Jailer**。

雖然是這樣說，但在 Paper 中提到的具體實作，仍為使用 Linux container 提供的技術執行，包含：

- 以 chroot 機制隔離執行的檔案系統
- 以 namepsace 隔離執行環境、pid、network namespaces
- 移除 System privilege (基於 Linux Capabilities 功能 [^linux-capabilities])
- 以 seccomp-bpf profile 設定允許呼叫的 syscall

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/firecracker-jailer.png" alt="Firecracker Jailer" caption="Firecracker Jailer" %}

在 jailer sandbox 配置的 chroot 目錄中，裡面僅包含 Firecracker 編譯的執行檔、`/dev/net/tun`、cgroup 控制檔案和 microVM 所需的資源。並且，預設情況下 seccomp-bpf profile 設定了 24 個系統呼叫操作 (syscalls) 和 30 個 ioctls 操作為白名單。

不過就我的研究，如果我理解正確，似乎 Firecracker 在 seccomp filter 上面在最近的版本多了不少 syscalls 支援:

- [Firecracker default seccomp filters (x86_64)](https://github.com/firecracker-microvm/firecracker/blob/main/resources/seccomp/x86_64-unknown-linux-musl.json)
- [Firecracker seccomp integration test case](https://github.com/firecracker-microvm/firecracker/blob/77fdc6ef390b5e04ae97a4c1b4bf0da541afc7bc/tests/integration_tests/security/test_seccomp.py)

## Firecracker 與 AWS Lambda 架構之間的關係 (High-level architecture)

在 Firecracker 設計出來後，AWS 便逐漸於 AWS Lambda 的底層架構中導入使用。使用 AWS Firecracker 允許 Lambda 的架構在每個執行的節點 (Lambda worker) 運行數千個 microVM。

### Lambda High-level to low-level architecture

AWS Lambda 從上層到下層的架構可以由遠至近如下：

(1) 用戶透過事件經由 Frondend service 觸發 Lambda function (可以是 API Gateway, 其他來源等)，會由 Worker Manager 定義配置部署可用的執行機器 (Lambda Worker)

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/lambda-invoke-flow.png" alt="AWS Lambda 觸發的架構流程" caption="AWS Lambda 觸發的架構流程" %}

(2) 一旦觸發後，Frondend service 交付由 Worker Manager 會遵循調度演算法 (sticky routing) 盡可能將觸發對象黏著在特定的 Lambda Worker 機器上，並且建議觸發的對象 (invoke service) 直接將請求的內容 (payload) 直接轉送到目標的 Lambda worker 機器上，減少觸發上的延遲和來回交互請求 (round-trip)。

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/lambda-event-path.png" alt="AWS Lambda 事件觸發的流程" caption="AWS Lambda 事件觸發的流程" %}

(3) 在每個 Lambda worker 提供了 *slot* 這個抽象物件，該抽象物件即客戶預先載入的 Lambda function 應用程式碼 (Lambda function code)，並且在後面每次觸發的行為上盡可能的重複使用這個執行環境 (slot)

重點在於 Firecracker 於 Lambda Worker 中部署的機制，每個 Lambda Worker 可以視為一個 Bare-metal 的機器，上面運行著 Firecracker VMM 用於管理多個 MicroVM (Lambda function, slot)；每個 microVM 包含了客戶的執行環境 (sandbox) 和客戶的應用程式碼，以及一個 shim control process 用於採用 TCP/IP socket 和 Micro Manager 互動的元件。

(MicroManager 可以視為 Lambda data plane 和 control plane 互動的元件)

> MicroManager provides slot management and locking APIs to placement, and an event invoke API to the Frontend

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/lambda-worker-architecture.png" alt="Lambda Worker 中的細部架構" caption="Lambda Worker 中的細部架構" %}

同時 MicroManager 部分也確保存在小量預先啟動的 MicroVMs，以確保有放置請求的即時需要。這是因為即使 Firecracker 能縮短在 **125ms** 內啟動，這樣的啟動時間可能仍不足以滿足 AWS Lambda 客戶快速啟動擴展的需要，並且可能會部分阻塞用戶的執行請求，因此在實務中，存在類似這樣 pre-warm 的機制。


### Firecracker I/O Path

當 AWS Lambda 中的應用執行寫入操作時 (假設 Guest OS 中的應用希望寫入檔案到磁碟)，此時會交付由 `virtio` driver 處理該操作，並且由 `virtio` driver 將其放置到共享記憶體 (shared memory) 中，並且於系統 Ring Buffer 進行緩衝。然後，Firecracker 將被喚醒執行 I/O 操作，並且將該寫入操作真實的寫進實體磁碟當中。[^deep-dive-into-aws-lambda-security]

### 實際遷移到 AWS Firecracker 的執行

論文中提到 AWS 從 2018 年開始將 AWS Lambda 的客戶從 EC2 運行容器 (per function per container) 的基礎平台轉移到 Firecracker。在遷移過程中，並無可用性中斷、延遲或其他指標層面問題。

不過，在 AWS 內部團隊在遷移過程中，一些小問題也因為這樣的遷移暴露出來，例如前面提到為了安全性考量關閉了 Symmetric MultiThreading (SMT) 機制 (過去的部署中是開啟的)，使得使用 Apache HttpClient 應用執行的行為因為一些執行緒 (Thread) 相關的 bug 也因此暴露，並且存在於過去的 AWS SDK 版本中，需透過修補依賴函式庫解決這項問題。

但在 AWS 內部團隊完成遷移後，便開始實際將外部客戶的相關基礎建設逐步遷移至 AWS Firecracker 為基礎的設施，並且獲得巨大的成功。

同時，有鑒於涉及未來安全性補丁的修復和系統更新，由於傳統使用 `rpm`、`yum` 等套件管理工具進行管理的變因太多，可能導致軟體一致性問題產生，AWS 團隊採用了 immutable infrastructure 的策略來完成這項工作，即透過使用新版本的 AMI (Amazon Machine Image, 用於 EC2 的啟動鏡像) 直接啟動新的 EC2 instances，並且替換舊的 EC2 instances 來完成這項工作。

## Evaluation (性能評估)

在該篇論文中，Firecracker 提供了數項不同測試數據的表現，同時，在 NSDI 2020 會議上也公開了對應的測試數據[^nsdi2020-data]。

下列的測試採用 EC2 `m5d.metal` instance type，其擁有 `Intel Xeon Platinum 8175M` 處理器 (48 cores, hyper-threading disabled)、**384GB RAM** 和 4 個 **840GB 的 NVMe 磁碟**。

在這項測試中 Host OS 使用 `Ubuntu 18.04.2` 以及 Linux kernel `4.15.0-1044-aws` 版本。

這項測試與幾個主要的虛擬化技術執行比較，包含：`Firecracker v0.20.0`、`Pre-configured Firecracker`、`Intel Cloud Hypervisor`、`QEMU v4.2.0`

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/firecracker-evaluation-boot-time.png" alt="Firecracker 啟動時間表現" caption="Firecracker 啟動時間表現" %}

在啟動時間的定義中，啟動時間為 VMM process 執行建立 process 操作 (fork) 並且 Guest Kernel 發起第一個 `init` process 的時間。

從數據顯示預先配置好 IO Port 的 Firecracker 和 Intel Cloud Hypervisor，**兩者環境啟動時間皆優於 QEMU**。然而，要注意的是，上述的測試結果中不包含設置網路裝置，一旦加入網路裝置的設置，Firecracker 和 Cloud Hypervisor 皆會在啟動時間中增加約 **20ms**，然而，QEMU 則是 **35ms**。

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/firecracker-memory-overhead.png" alt="Firecracker 記憶體消耗使用表現" caption="Firecracker 記憶體消耗使用表現" %}

在記憶體的消耗表現上 (Figure 7)，可以觀察到 QEMU 本身需要 128MB 的記憶體、Cloud Hypervisor 則約為 13 MB，然而，Firecracker 僅需約為 3MB 的記憶體消耗。

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/firecracker-io-performance.png" alt="Firecracker 磁碟 I/O 操作效能評估" caption="Firecracker 磁碟 I/O 操作效能評估 (使用 fio)" %}

值得一提的是，在檔案 I/O 操作的表現上 (Figure 8 & Figure 9)，該研究使用 `fio` 執行測試，**明顯可以關注到在硬體資源能夠負荷超過 340,000 read IOPS (1GB/s at 4kB) 的情況下，Firecracker 以及 Cloud Hypervisor  僅可以被限縮使用約 13,000 IOPS (52MB/s at 4kB) 的吞吐效能**。

{% include figure image_path="/assets/images/posts/2022/02/firecracker-paper-reading/firecracker-iperf.png" alt="Firecracker 網路吞吐效能評估" caption="Firecracker 網路吞吐效能評估 (使用 iperf3)" %}

該研究同時也採用了 `iperf3` 執行網路效能的測試 (針對虛擬的 tap 網路介面, MTU 為 **1500 byte**)。**在機器能夠達到單一網路流 44Gb/s 及 46Gb/s (10 個並行傳輸) 的狀況下，Firecracker 僅可達到約 15Gb/s 的吞吐**。然而，QEMU 獲得接近於 Cloud Hypervisor 的測試結果，均能擁有較好的網路吞吐性能，這裡部分歸納結論是由於 `virtio` 的裝置設計實作而產生的限制。

## 研究改進和結論

如同前面性能評估所提及的，基於 `virtio` 的實作因素，這使得 Firecracker 並無法取得以直接存取 PCI 裝置以接近實體機器的 I/O 吞吐性能。使得網路和磁碟 I/O 的效能存在部分限制。

然而，就論該研究的總結，與前面提及的六項主要問題呼應，AWS Firecracker 的技術著實達到其工程上的設計目標，包含：

- **Isolation**: 以 Rust 為基礎的 VMM 允許多個用戶 (multi-tenant) 於單一機器上運行並達到執行環境隔離
- **Overhead and Density**: Firecracker 允許單一硬體資源運行數千個 MicroVMs，以達到節省硬體資源的目的，同時帶來更低的 CPU 和 Memory 資源消耗
- **Performance**: Block IO 以及網路吞吐效能著實存在改善空間，但著實已經滿足 AWS Lambda 及 AWS Fargate 兩者產品所需
- **Compatibility**: Firecracker MicroVMs 運行為修改的 Linux kernel，允許客戶運行相關的應用程式碼，目前尚未發現不允許於 MicroVM 中運行的應用程式
- **Fast Switching**: Firecracker MicroVMs 有較短的啟動時間 (150ms)，並且在多個 MicroVMs 並行啟動的狀況下保持一致的效能
- **Soft Allocation**: Firecracker 測試允許超出 20 倍的資源配置比例，在 AWS 生環境中為允許超出實際 CPU 和 Memory 10 倍的配置比例並未存在問題

## 總結

Firecracker 除了於部分開源專案為虛擬化提供解決方案外 (例如：Kata container)，目前 AWS Firecracker 更是已經導入使用於 AWS 的基礎產品建設中，包含 [AWS Fargate](https://aws.amazon.com/fargate/) 和 [AWS Lambda](https://aws.amazon.com/lambda/)。

這樣的基礎設施改進同時也為客戶帶來更大的優勢，借助 Firecracker 的設計，使得 AWS Fargate 將運算定價折扣甚至達到 50% 的成本優化 ([AWS Fargate Price Reduction – Up to 50%](https://aws.amazon.com/blogs/compute/aws-fargate-price-reduction-up-to-50))。

基於對於這樣的技術感興趣，我著實閱讀了有關 Firecracker 整篇論文和參考部分 Firecracker 專案，歸納出上述的內容，並且花了一些時間整理這項導讀，更多訊息可以參考：

- [Firecracker: Lightweight virtualization for serverless applications](https://github.com/0xlen/paper-reading/blob/master/firecracker-lightweight-virtualization-for-serverless-applications.pdf)
- [Firecracker project page](https://firecracker-microvm.github.io/)
- [Firecracker source code](https://github.com/firecracker-microvm/firecracker)

希望透過這樣的導讀，能夠有助於你更加了解 AWS Firecracker 這項技術。

如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道。

## 延伸資源

- [AWS re:Invent 2018: REPEAT 1 A Serverless Journey: AWS Lambda Under the Hood (SRV409-R1)](https://www.youtube.com/watch?v=QdzV04T_kec)
- [AWS re:Invent 2019: REPEAT 1 A serverless journey: AWS Lambda under the hood (SVS405-R1)](https://www.youtube.com/watch?v=xmacMfbrG28)
- [Firecracker: A Secure and Fast microVM for Serverless Computing (Orelly)](https://www.youtube.com/watch?v=PAEMGa-i2lU)
- [Firecracker: Lightweight Virtualization - Opportunities and Challenges (2020)](https://www.youtube.com/watch?v=ADOfX2LiEns)
- [Deep Dive into firecracker-containerd (2019)](https://www.youtube.com/watch?v=0wEiizErKZw)
- [Extending containerd - Samuel Karp & Maksym Pavlenko, Amazon (2019)](https://www.youtube.com/watch?v=9avPJL9Zqso)
- [Deep Dive into firecracker-containerd - Mitch Beaumont (AWS)](https://www.youtube.com/watch?v=2rwYZdVPN4g)

## References

[^firecracker-paper-note]: Alexandru Agache, Marc Brooker, Andreea Florescu, Alexandra Iordache, Anthony Liguori, Rolf Neugebauer, Phil Piwonka, Diana-Maria Popa. (2020). [Firecracker: Lightweight virtualization for serverless applications](https://github.com/0xlen/paper-reading/blob/master/firecracker-lightweight-virtualization-for-serverless-applications.pdf)
[^virtualization-architectures]: Gerald J. Popek, Robert P. Goldberg. (1974). [Formal requirements for virtualizable third generation architectures](https://dl.acm.org/doi/10.1145/361011.361073)
[^how-firecracker-work]: [How AWS’s Firecracker virtual machines work](https://www.youtube.com/watch?v=BIRv2FnHJAg)
[^firecracker-open-source-innovation]: [AWS re:Invent 2019: Firecracker open-source innovation (OPN402)](https://www.youtube.com/watch?v=yDplzXEdBTI)
[^linux-capabilities]: [Linux Capabilities and Seccomp](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/container_security_guide/linux_capabilities_and_seccomp)
[^deep-dive-into-aws-lambda-security]: [AWS re:Invent 2020: Deep dive into AWS Lambda security: Function isolation](https://www.youtube.com/watch?v=FTwsMYXWGB0)
[^nsdi2020-data]: [Firecracker benchmarking code and data](https://github.com/firecracker-microvm/nsdi2020-data)

