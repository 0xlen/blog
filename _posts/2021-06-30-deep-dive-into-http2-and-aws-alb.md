---
title: "[AWS] 排除 Application Load Balancer 使用 HTTP/2 健康檢查 (Health Check) 失敗"
description: ""
tags: ['aws', 'amazon web services', 'EC2', 'Elastic Compute Cloud', 'amazon', 'ELB', 'ALB', 'Load Balancer', 'Elastic Load Balancer', 'HTTP2']
classes: wide
header:
  og_image: /assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/cover.png
  teaser: /assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/cover.png
---

自 2020 年 10 月，AWS Elastic Load Balancer --- Application Load Balancer (ALB) [1] 開始支援 End-to-End HTTP/2 協定，這意味著你可以透過 Application Load Balancer 幫助你處理 HTTP/2 版本的請求。

當你在建立 Target Group 時，可以選擇相應的使用版本 (Protocol Version)，並且採用 HTTP/2 轉送：

{% include figure image_path="/assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/select-protocol-version.png" alt="於 ELB 建立 Target Group 時選擇使用 HTTP/2" caption="於 ELB 建立 Target Group 時選擇使用 HTTP/2" %}

這篇內容我將會與你分享我在工作中處理案例時，使用該功能所遭遇的問題，並提及相關的技術細節。

## 問題描述

我嘗試在我的環境中建立了一個 Application Load Balancer (ALB)，並且，使用範例應用程式 [2] 為 ALB 轉發提供服務。

我在我的環境中，使用了以下設定：

- Protocol Version: HTTP2
- 為 ALB 設定健康檢查 (Health Check)

但應用程式始終無法通過 ALB 的健康檢查 (一直為 Unhealthy 狀態)：

{% include figure image_path="/assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/http2-target-failed-alb-health-check.png" alt="選擇使用 HTTP/2 無法通過健康檢查 (Health Check)" caption="選擇使用 HTTP/2 無法通過健康檢查 (Health Check)" %}

然而，使用 HTTP1 時，能夠正確通過健康檢查。同時，使用 `--http2` 測試也可以返回相應的內容：

```bash
$ curl --http2 172.31.18.7 -vvv -s > /dev/null

* Rebuilt URL to: 172.31.18.7/
*   Trying 172.31.18.7...
* TCP_NODELAY set
* Connected to 172.31.18.7 (172.31.18.7) port 80 (#0)
> GET / HTTP/1.1
> Connection: Upgrade, HTTP2-Settings
> Upgrade: h2c
> HTTP2-Settings: AAMAAABkAARAAAAAAAIAAAAA
>
< HTTP/1.1 200 OK
< Server: gunicorn/19.9.0
< Date: Thu, 08 Jul 2021 21:15:29 GMT
< Connection: keep-alive
< Content-Type: text/html; charset=utf-8
< Content-Length: 9593
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Credentials: true
<
{ [9593 bytes data]
* Connection #0 to host 172.31.18.7 left intact
```

## 問題研究

### Client 端行為 (curl)

問題回到 curl 提供的 `--http2` 參數，文件 [3] 中提及了該參數可以幫助你啟用 HTTP/2：

> curl offers the --http2 command line option to enable use of HTTP/2.

從請求行為中，也可以注意到該行為的請求：

```bash
$ curl --http2 172.31.18.7/get -vvv

*   Trying 172.31.18.7...
* TCP_NODELAY set
* Connected to 172.31.18.7 (172.31.18.7) port 80 (#0)
> GET /get HTTP/1.1
> Host: 172.31.18.7
> User-Agent: curl/7.61.1
> Accept: */*
> Connection: Upgrade, HTTP2-Settings
> Upgrade: h2c
> HTTP2-Settings: AAMAAABkAARAAAAAAAIAAAAA
>
< HTTP/1.1 200 OK
< Server: gunicorn/19.9.0
< Date: Wed, 30 Jun 2021 11:00:10 GMT
< Connection: keep-alive
< Content-Type: application/json
< Content-Length: 304
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Credentials: true
<
{
"args": {},
"headers": {
  "Accept": "*/*",
  "Connection": "Upgrade, HTTP2-Settings",
  "Host": "172.31.18.7",
  "Http2-Settings": "AAMAAABkAARAAAAAAAIAAAAA",
  "Upgrade": "h2c",
  "User-Agent": "curl/7.61.1"
},
"origin": "172.31.18.7",
"url": "http://172.31.18.7/get"
}
* Connection #0 to host 172.31.18.7 left intact
```

從 curl 的行為中，可以注意到 `--http2` 其實是採用基於新增 HTTP/1.1 Upgrade Header 的行為，並且由 Client 發起了 `Upgrade: h2c` 的 HTTP Header。若 Server 端支援 HTTP/2，一般情況下，將預期返回使用 HTTP 101 (Switching Protocols)，告知 Client 可以使用 HTTP/2 協定互動 [4]。

在與熟悉網路的同事和專家們討論後，提及在 RFC 7540 [5] 文件中定義了發起 HTTP/2 協定的行為和限制：

- **(A) Negotiating HTTP/2 via a secure connection with TLS and ALPN**
  - (在 TLS 交握的過程中，於 TLS extension (TLS-ALPN) 帶入 "h2")
- **(B) Upgrading a plaintext connection to HTTP/2 without prior knowledge**
  - (在明文傳輸中以 HTTP/1.1  傳送 Upgrade Header，表明使用 HTTP/2 傳輸)
- **(C) Initiating a plaintext HTTP/2 connection with prior knowledge**
  - (直接在明文傳輸環境中發起 HTTP/2 連線)

同時，HTTP/2 的設計中，改進了使用單一個 TCP Connection，並且採用 Binary Framing，將過去 HTTP/1.1 相關包含的 Header 以二進位方式編碼，以作為主要 HTTP 請求的機制：

{% include figure image_path="/assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/http2-in-one-slide.png" alt="HTTP2 in one slide" caption="HTTP2 in one slide - ([source](https://docs.google.com/presentation/d/1r7QXGYOLCh4fcUq0jDdDwKJWNqWK1o4xMtYpKZCJYjM/present?slide=id.p19))" %}

所以，截至目前為止，可以確定 Client 端的行為 (curl) 採用 `--http2` 參數所執行的測試方式，在明文傳輸中，**會以 HTTP/1.1  傳送 Upgrade Header，表明使用 HTTP/2 傳輸，並不意味著確定應用程式完全支持 HTTP/2**。

### Application Load Balancer (ALB) 使用 HTTP/2 的請求行為

接下來讓我們來進一步了解為什麼採用 HTTP/2 之後，Health Check (健康檢查) 會失敗，以及探討 ALB 在使用 HTTP/2 相關的請求機制。在我的環境中，請求行為如下：

![ALB 及應用彼此的請求行為](/assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/alb-http2-architecture.png){: .align-center}


在收集相關的封包後，可以確認 Application Load Balancer 在主動發起 Health Check 檢查 HTTP 請求時，會主動使用 (C) 的方法，直接在明文傳輸環境中發起 HTTP/2 連線：

{% include figure image_path="/assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/connection-preface-from-alb-packet.png" alt="HTTP/2 Connection Preface" caption="HTTP/2 Connection Preface" %}

同時，從封包中可以注意到 Application Load Balancer 在建立連線時於連線標頭主動提及的協議版本。這如同在在 RFC 7540 [5] - 3.5 中提及的規範，如果用戶端已經知道目標對象將採用 HTTP/2 版本，在建立連線時，每個連線會採用發起 Connection Preface 以作為連線協議版本。用戶端將需要在發起以 HTTP/2 的連線時，發起以下內容：

> The client connection preface starts with a sequence of 24 octets, which in hex notation is:
```text
0x505249202a20485454502f322e300d0a0d0a534d0d0a0d0ao
```

其 16 進制轉換為以下標頭：

```text
PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n"
```

比對封包中的內容後 (如同圖：HTTP/2 Connection Preface)，可以確認 Application Load Balancer 進行 Health Check 的行為。

至此，目前可以確定，在 Application Load Balancer 已經得知目標採用 HTTP/2 版本的情況下，將採用第三種方式，即發送 Connection Preface 與目標對象 (這個範例為 172.31.8.7) 建立 HTTP/2 連接，以發送 Health Check (健康檢查) 請求。

## 解決方案

回到 Client (curl) 提供的對應行為，在 curl 提供的方法中，列舉了以下兩種不同的請求方法 [3]：

> 1. `--http2` command line option to enable use of HTTP/2.
> 2. `--http2-prior-knowledge` command line option to enable use of HTTP/2 without HTTP/1.1 Upgrade.

如同前面提及，在使用參數 `--http2` 時 (並非 Prior Knowledge 直接採用 HTTP/2 連線)，若服務器不支持 HTTP/2，根據 RFC 7540 [5]，Client 發送 Upgrade: "h2c" 後，應用程式仍然可以使用 HTTP/1.1 回應。

從 Application Load Balancer 所附帶的方法，我們可以確定其包含 Connection Preface (可用於明文傳輸並基於 Prior Knowledge，以 HTTP/2 直接連線)。

因此，從 curl 提供的 `--http2-prior-knowledge`，可以間接模擬 Application Load Balancer 執行 Health Check (健康檢查) 的行為。

從我的應用程式測試結果中，可以注意到，我的應用程式其實是無法正確支持使用 HTTP/2 協定的行為，這通常涉及應用程式使用不支援對應 HTTP/2 協議相關的網路函式庫：

```bash
$ curl --http2-prior-knowledge 172.31.18.7/get -vvv

*   Trying 172.31.18.7...
* TCP_NODELAY set
* Connected to 172.31.18.7 (172.31.18.7) port 80 (#0)
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Send failure: Broken pipe
* Failed sending HTTP2 data
* nghttp2_session_send() failed: The user callback function failed(-902)
* Connection #0 to host 172.31.18.7 left intact
curl: (16) Send failure: Broken pipe
```

不過，nginx 在 1.9.5 版本開始，正式支援 HTTP/2 [6]，因此，為了驗證是否與應用程式設計有關，我在我的環境使用了使用了 nginx 1.12.2，並且於設定檔中啟用了 HTTP/2 module：

```bash
bash-4.2# nginx -v
nginx version: nginx/1.12.2

bash-4.2# cat /etc/nginx/nginx.conf
http {
    server {
        listen 80 http2 default_server;
        listen [::]:80 http2 default_server;
        ...
    }
}
```

在啟用後，可以從 curl 的測試結果中確認 `--http2-prior-knowledge` 正確被支援：

```bash
$ curl --http2-prior-knowledge http://172.31.18.7 -vvv -I
...
* * Using HTTP2, server supports multi-use
* * Connection state changed (HTTP/2 confirmed)
* * Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* * Using Stream ID: 1 (easy handle 0x25d0140)
* > HEAD / HTTP/2
* > Host: 172.31.18.7:80
* > User-Agent: curl/7.61.1
* > Accept: */*
* >
* * Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
* < HTTP/2 200
* HTTP/2 200
* < server: nginx/1.12.2
* server: nginx/1.12.2
* < date: Sat, 10 Jul 2021 19:33:24 GMT
* date: Sat, 10 Jul 2021 19:33:24 GMT
* < content-type: text/html
* content-type: text/html
* < content-length: 3520
* content-length: 3520
* < last-modified: Wed, 28 Aug 2019 19:52:13 GMT
* last-modified: Wed, 28 Aug 2019 19:52:13 GMT
* < etag: "5d66db6d-dc0"
* etag: "5d66db6d-dc0"
* < accept-ranges: bytes
* accept-ranges: bytes
*
* <
* * Connection #0 to host 172.31.18.7 left intact
```

同時也可以確認應用程式能夠正確通過由 Application Load Balancer 發起的 Health Check 請求：

```bash
bash-4.2# tail access.log
172.31.0.26 - - [10/Jul/2021:19:31:14 +0000] "GET / HTTP/2.0" 200 3520 "-" "ELB-HealthChecker/2.0" "-"
172.31.31.230 - - [10/Jul/2021:19:31:14 +0000] "GET / HTTP/2.0" 200 3520 "-" "ELB-HealthChecker/2.0" "-"
172.31.0.26 - - [10/Jul/2021:19:31:19 +0000] "GET / HTTP/2.0" 200 3520 "-" "ELB-HealthChecker/2.0" "-"
172.31.31.230 - - [10/Jul/2021:19:31:19 +0000] "GET / HTTP/2.0" 200 3520 "-" "ELB-HealthChecker/2.0" "-"
172.31.0.26 - - [10/Jul/2021:19:31:24 +0000] "GET / HTTP/2.0" 200 3520 "-" "ELB-HealthChecker/2.0" "-"
...
```

{% include figure image_path="/assets/images/posts/2021/06/deep-dive-into-http2-and-aws-alb/nginx-http2-pass-health-check.png" alt="Nginx 使用 HTTP/2 通過 Health Check" caption="Nginx 使用 HTTP/2 通過 Health Check" %}

至此，可以確定 Health Check (健康檢查) 失敗的主要原因，涉及應用程式本身並未正確設計以回應 HTTP/2 協議的支持。並且，可以從數項測試和實際連線數據封包中具體驗證這項細節。

在修正使用支持的應用程式 (Nginx) 後，Application Load Balancer 中的 Health Check (健康檢查) 指標正確檢測其通過。

## 總結

在這篇內容中，我與你分享了在 Application Load Balancer 中使用 HTTP/2 相關的請求機制。並且進一步分析了用戶端 (curl) 和 Application Load Balancer 本身的行為，同時列舉了使用 nginx 啟用 HTTP/2 Module 修正的範例設定。

如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道。

## References

- [1] [New – Application Load Balancer Support for End-to-End HTTP/2 and gRPC](https://aws.amazon.com/blogs/aws/new-application-load-balancer-support-for-end-to-end-http-2-and-grpc/)
- [2] [httpbin.org](https://nghttp2.org/httpbin/#/)
- [3] [Curl - HTTP/2 with curl](https://curl.se/docs/http2.html)
- [4] [Wiki - HTTP/1.1 Upgrade header](https://en.wikipedia.org/wiki/HTTP/1.1_Upgrade_header)
- [5] [RFC 7540](https://datatracker.ietf.org/doc/html/rfc7540#section-3.2)
- [6] [NGINX Open Source 1.9.5 Released with HTTP/2 Support](https://www.nginx.com/blog/nginx-1-9-5/)
