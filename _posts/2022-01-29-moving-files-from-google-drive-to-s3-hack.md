---
title: "3 個步驟不用下載檔案從 Google Drive 直接備份到 Amazon S3"
description: "Google Drive 教育帳號即將取消無上限儲存空間政策，你還在努力的下載檔案轉移備份嗎？如果你正在尋找非手動檔案搬家的解決方式，這篇內容我將會與你分享簡單的步驟，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份到 Amazon S3。"
tags: ['google', 'aws', 's3', 'python']
classes: wide
header:
  og_image: /assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/cover.png
  teaser: /assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/cover.png
---

繼[上篇分享搬家到 Dropbox 的步驟](/moving-files-from-google-drive-to-dropbox-hack)，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份。這則內容將使用 Amazon S3 作為主要的範例。由於 Google Drive 教育帳號即將取消無上限儲存空間政策，你還在努力的下載檔案轉移備份嗎？如果你正在尋找非手動檔案搬家的解決方式，這篇內容我將會與你分享簡單的步驟，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份。

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/cover.png" alt="3 個步驟不用下載檔案從 Google Drive 直接備份到 Amazon S3" caption="3 個步驟不用下載檔案從 Google Drive 直接備份到 Amazon S3" %}

{% capture Notes  %}
**在這篇文章中你將知道**

- 了解不用下載檔案從 Google Drive 直接備份到 Amazon S3 的另一個方法
- 學習應用 AWS SDK for Python 的功能轉移檔案和了解相關範例的演示
- 了解其他的應用
{% endcapture %}

<div class="notice--info">{{ Notes | markdownify }}</div>

## 概覽

為了達到不必從 Google Drive 下載檔案，並將檔案轉移備份的目的，這個步驟分為幾個階段：

- 擁有其他雲端儲存空間的帳戶
- 設置 Google Colab
- 無需下載，使用對應支持的 AWS SDK 直接上傳搬移檔案

在上篇內容中，提到了使用 Dropbox 作為主要的範例，如果你在尋找 Dropbox 相關的細節，有興趣可以參考[上篇](/moving-files-from-google-drive-to-dropbox-hack)的內容以獲得更多資訊。在這則內容終將圍繞 AWS 作為主要的環境。

### Google Colaboratory (Google Colab)

Google Colab - Colaboratory 簡稱“Colab”，是 Google Research 團隊開發的一款產品。使用 Colab 可以通過瀏覽器直接編寫和執行任意 Python 代碼。常被用來適合機器學習、數據分析和教育目的。

從技術上說，Colab 是一種託管式 Jupyter 筆記本服務。用戶無需進行設置，就可以直接使用。

簡而言之，Google Colab 提供了可用的運算資源，因此，在這個工作的執行階段，本篇內容將透過 Colab 所提供的功能協助檔案轉移的工作。

#### 使用限制

Colab 的資源使用並沒有保證一定的 CPU、Memory 資源量，但對於這項用應用於遷移的工作上能更簡單地完成這項工作。唯獨要注意的是，[一個 Colab 的虛擬執行環境最多執行到 12 小時。](https://research.google.com/colaboratory/faq.html)

### Amazon S3 (Simple Storage Service)

Amazon Simple Storage Service (Amazon S3) 是 Amazon Web Service 提供的資料儲存服務，基本上你可以把它視為一種雲端儲存的解決方案。最大的特點就是 S3 沒有放置容量的上限，費用則會依照你的儲存容量和儲存的資料選用計費 - **也就是你放多少資料，就收多少錢**。如果對於 Amazon S3 的定價想了解更多，可以參考以下頁面及工具試算：

- [Amazon S3 定價](https://aws.amazon.com/tw/s3/pricing)
- [AWS Pricing Calculator - 費用試算](https://calculator.aws/#/createCalculator/S3)

S3 服務將資料存放在名為 Bucket 的對象中，可以想像就是一個獨立的硬碟，並且可以選擇不同的區域。

用戶可以選擇離自己地理位置區域較近的資料中心建立這個資源，一個 AWS 用戶可以建立多個 S3 Bucket 儲存資料。

## 執行步驟

### Step 1: 擁有一個 AWS 帳號

一切的前提是你要先有一個 [AWS 帳戶](https://aws.amazon.com/) 能夠操作 AWS 資源。

### Step 2: 建立 S3 Bucket 並且設定 IAM User 並且賦予必要的權限

接下來你就可以透過訪問 S3 Console 建立 Amazon S3 bucket，一旦擁有帳號並且開通後，可以透過以下連結訪問 Amazon S3 Console：

- [S3 Management Console](https://s3.console.aws.amazon.com/s3/home)

點擊 **Create Bucket** 可以進行 S3 Bucket 的建立操作：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/s3-console.png" alt="S3 主控台介面" caption="S3 主控台介面" %}

輸入預期的 S3 Bucket 名稱 (命名為唯一識別的名稱，不可與其他使用者重複) 和區域：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/create-s3-bucket.png" alt="建立 S3 Bucket" caption="建立 S3 Bucket" %}

> 根據我的測試，Google Colaboratory 似乎會選擇在執行環境中選擇離你較近的機房作為執行環境的首要選擇。為了提高檔案從 Google Drive 傳輸到 Amazon S3 搬家的效率，建議選擇離自己地理位置近的地方建立 S3 Bucket，也許能降低一部分的傳輸延遲。
>
> 如果是亞洲地區的用戶通常是日本東京 (`ap-northeast-1`)、新加坡 (`ap-southeast-1`) 或是澳洲 (`ap-southeast-2`)

一旦建立好 S3 Bucket 之後，基本上你就可以透過網頁介面上傳檔案了。

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/bucket-interface.png" alt="在 S3 主控台介面中可以直接上傳檔案" caption="在 S3 主控台介面中可以直接上傳檔案" %}

但為了實現我們無需下載檔案搬遷的目標，我們需要透過程式化的方式來運作上傳的邏輯，如此一來，才能在 Google Colab 中運行我們的應用執行相關的操作。

因此，在接下來的步驟中，我們將透過建立一個可供 Python 程式應用的使用者憑證，其中透過 IAM User 的方式設定以賦予必要的權限。

IAM (Identity Access Management) 是一個 AWS 整合的主要服務，可以想像是作業系統的權限控制方法，讓您能夠安全地控制對 AWS 資源的存取，IAM 能夠控制每個用戶存取不同 AWS 資源的各種權限。

#### 建立 IAM User

要建立 IAM User，可以透過在上方搜尋列尋找 IAM 連結到對應的服務主控台頁面： 

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/navigate-to-iam.png" alt="IAM Console" caption="IAM Console" %}

點擊 "Add IAM User" 進入建立 IAM User 的步驟並且輸入必要資訊：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/add-iam-user.png" alt="新增 IAM User" caption="新增 IAM User" %}

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/iam-user-programmatic-access.png" alt="輸入 IAM User 名稱和類型" caption="輸入 IAM User 名稱和類型" %}

- **User name**: IAM User 的唯一識別名稱，可以自行命名，不能與其他使用者重複
- **Select AWS Credential type**: 為了要用於 Python 應用程式操作的憑證，這裡選擇 **Access Key**，用於產生純文字類型的憑證

(**但請注意不要把這組憑證外流，比如放到 GitHub，否則可能其他人都能任意存取甚至修改你 AWS 資源的資料**)

#### 權限設定

下一步就是選擇這個 IAM Policy，用於限制這個 IAM User 到底只能操作、存取哪些 AWS 資源和類型 (例如：是僅可讀，還是也包含寫入？)。預設情況下，IAM User 不指定任何 IAM Policy 等於沒有任何權限操作資源。

如果要賦予操作 S3 Bucket 的權限，最簡單的方式是使用預先定義好的 `AmazonS3FullAccess` 策略，這個策略會開放 IAM User 操作整個 AWS 帳號底下的 S3 Bucket 資源 (包含可讀、可寫)：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/iam-user-attach-policy.png" alt="附加 `AmazonS3FullAccess` 策略" caption="附加 `AmazonS3FullAccess` 策略" %}

若要限制 IAM User 只能操作單一個 S3 Bucket，可以透過近一步進行進階的設定，以下是範例的 JSON 格式的策略：

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowOperateBucket",
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::google-drive-backup-example/*"
        }
    ]
}
```

#### 完成建立並且記錄 IAM 

根據建立流程檢視 IAM User 的資訊沒有問題，就可以點擊 Create User 完成建立：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/iam-create-user.png" alt="檢視 IAM User 建立資訊" caption="檢視 IAM User 建立資訊" %}

建立環節中最後會產生一組隨機的憑證 (Access Key ID & Secret Access Key)，**由於該憑證只會產生一次，一離開這個頁面無法在取得，請紀錄這組憑證的資訊並妥善留存**：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/review-access-secret-key.png" alt="獲得 IAM User 的憑證資訊 (Access Key ID & Secret Access Key)" caption="獲得 IAM User 的憑證資訊 (Access Key ID & Secret Access Key)" %}

建立完成後你就可以在 IAM User 的介面找到剛建立完成的用戶，至此 IAM 相關的設定就告一個段落：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/review-iam-user.png" alt="檢視 IAM User" caption="檢視 IAM User" %}

{% capture notice %}
**請注意不要把這組憑證外流，比如放到 GitHub，否則可能其他人都能任意存取甚至修改你 AWS 資源的資料**
{% endcapture %}

<div class="notice--danger">{{ notice | markdownify }}</div>

如果你策略也可以在 IAM User 建立完之後附加 (選擇使用 "Add Permissions" 指定 Managed Policy 或是 Add Inline Policy)：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/add-inline-policy.png" alt="附加 IAM 策略" caption="附加 IAM 策略" %}

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/add-inline-policy-example.png" alt="附加 IAM 策略" caption="附加 IAM 策略" %}

如此一來，你就可以在 Python 程式碼中，使用 SDK 提供的方法帶入產生的 access token 初始化 S3 用戶端：

```python
import boto3

# Create an S3 client
s3 = boto3.client('s3')
```

### Step 3: 建立 Google Colaboratory 並且在 CoLab 執行環境中使用 AWS SDK for Python 轉移資料

下一步就是可以直接在目標的 Google Drive 帳號中運行 Colab 環境。CoLab [支持掛載 Google Drive](https://colab.research.google.com/notebooks/io.ipynb#scrollTo=u22w3BFiOveA) 在執行環境中當成本地端硬碟直接使用，透過這種方式能夠直接在 CoLab 環境中指定要執行備份的目錄工作。

透過在要轉移的 Google Drive 中新增一個 Google Colaboratory 的應用選項，可以直接啟用一個執行環境：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/add-colab.png" alt="新增 Google Colab 應用" caption="新增 Google Colab 應用" %}

如果沒有看到 Colab 選項的話，可以透過連結更多應用的選項安裝 Google Colaboratory：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/connect-more-app.png" alt="在 Google Drive 連結更多應用" caption="在 Google Drive 連結更多應用" %}

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/install-colab.png" alt="在 Google Drive 安裝 Google Colaboratory" caption="在 Google Drive 安裝 Google Colaboratory" %}

透過 PyPi，可以直接安裝 AWS SDK for Python (boto3)：

```python
!pip install boto3
```

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/install-boto3.png" alt="安裝 Boto3" caption="安裝 Boto3" %}

接下來就可以直接透過 Colab 掛載 Google Drive：

```python
from google.colab import drive
drive.mount('/gdrive')
```

點擊運行就能將 Google Drive 掛載 (`/gdrive/MyDrive`)，並且授與必要的權限：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/mount-drive.png" alt="授與存取 Google Drive 的權限" caption="授與存取 Google Drive 的權限" %}

掛載後就能透過 AWS SDK 提供的 API 直接執行檔案上傳到 S3，並且透過 Colaboratory 的執行環境可以跑在雲端的背景中執行，完全不需要下載檔案並且將檔案從 Google Drive 搬家到 Amazon S3。

```python
import boto3

s3_client = boto3.client('s3')
response = s3_client.upload_file(file_name, bucket, object_name)
```

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/result.png" alt="執行範例" caption="執行範例" %}

如果你熟悉 AWS CLI 的操作，甚至可以直接在 Colab 環境使用 AWS CLI 直接做出更多不同的操作和應用：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-s3-hack/cli-result.png" alt="CLI 執行範例" caption="CLI 執行範例" %}

唯獨要注意的是，如果是部分 Google Drive 特定的檔案格式 (e.g. Google doc .gdoc ... 等等)，因為在在 Colab 用 Python 的檔案讀取操作會遭遇失敗，部分這類的檔案無法透過 Colab 直接轉移，可以另外透過 Python 應用掃描出來之後，在 Google Drive 搬到一個統一的目錄手動執行壓縮下載的操作 (Google Drive 通常會轉換到在一般環境中讀取的格式)。

{% capture CallToAction %}

如果你有興趣想了解實作的內容

你可以透過以下的方法填入自定義的金額 Buy Me A Coffee

{% include buy-coffee type="ecpay" item="GDriveS3SampleCode" %}

你會獲得贈送的簡單範例程式並且寄送到你的 Email，了解如何直接上傳整個目錄底下的資源，並幫助你快速的瞭解如何使用

{% endcapture %}

<div class="notice--primary">{{ CallToAction | markdownify }}</div>

## 總結

在這篇內容中，我與你分享了非手動檔案搬家的解決方式。在這篇內容中，提幾了幾個簡單的步驟，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份。並且使用 Google Colaboratory 執行 Amazon S3 遷移作為範例。

如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道。

## References

- [1] [AWS SDK for Python - S3](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#s3)


