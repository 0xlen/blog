---
title: "沒有時間跟硬碟容量備份？3 個步驟不用下載檔案從 Google Drive 直接備份到 Dropbox"
description: "Google Drive 教育帳號即將取消無上限儲存空間政策，你還在努力的下載檔案轉移備份嗎？如果你正在尋找非手動檔案搬家的解決方式，這篇內容我將會與你分享簡單的步驟，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份。"
tags: ['google', 'dropbox', 'python']
classes: wide
header:
  og_image: /assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/cover.png
  teaser: /assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/cover.png
---

> Google Drive 教育帳號即將取消無上限儲存空間政策，你還在努力的下載檔案轉移備份嗎？如果你正在尋找非手動檔案搬家的解決方式，這篇內容我將會與你分享簡單的步驟，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份。

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/cover.png" alt="3 個步驟不用下載檔案從 Google Drive 直接備份到 Dropbox" caption="3 個步驟不用下載檔案從 Google Drive 直接備份到 Dropbox" %}

## Google Drive 儲存空間政策異動

[Google Workspace for Education 公告了即將採用全新儲存空間政策，並且於 2022 年 7 月生效，全教育機構僅可共用 100TB](https://support.google.com/a/answer/10431555?hl=zh-Hant)。邁進 2022 年，免費版的 Google Drive 帳號又只能存放 15 GB 的資料，可能正在閱讀這篇內容的你正在煩惱如何有效率的將資料轉移至其他雲端儲存，畢竟假如直接透過 Google Drive 網頁版下載後上傳，會花非常多時間以及電腦的本機容量。

如果你正在尋找非手動檔案搬家的解決方式，這篇內容我將會與你分享一些簡單的步驟，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份。

## 概覽

為了達到不必從 Google Drive 下載檔案，並將檔案轉移備份的目的，這個步驟分為幾個階段：

- 擁有其他雲端儲存空間的帳戶
- 設置 Google Colab
- 無需下載，使用對應支持的 SDK 直接上傳搬移檔案

### Google Colaboratory (Google Colab)

Google Colab - Colaboratory 簡稱“Colab”，是 Google Research 團隊開發的一款產品。使用 Colab 可以通過瀏覽器直接編寫和執行任意 Python 代碼。常被用來適合機器學習、數據分析和教育目的。

從技術上說，Colab 是一種託管式 Jupyter 筆記本服務。用戶無需進行設置，就可以直接使用。

簡而言之，Google Colab 提供了可用的運算資源，因此，在這個工作的執行階段，本篇內容將透過 Colab 所提供的功能協助檔案轉移的工作。

#### 使用限制

Colab 的資源使用並沒有保證一定的 CPU、Memory 資源量，但對於這項用應用於遷移的工作上能更簡單地完成這項工作。唯獨要注意的是，[一個 Colab 的虛擬執行環境最多執行到 12 小時。](https://research.google.com/colaboratory/faq.html)

## 執行步驟

### Step 1: 擁有一個 Dropbox 帳號

因為 Dropbox 提供的 [Python SDK](https://www.dropbox.com/developers/documentation/python#overview) 相對友善許多，因此這篇內容以 Dropbox 遷移的過程。首先，你會需要一個 Dropbox 帳號。

{% capture JoinDropbox %}

**現在新用戶註冊 Dropbox 立即取得免費的額外空間**

![dropbox-logo](/assets/images/posts/2017/01/apache-and-php-upload-large-file/box.png){: .align-center}

<a href="https://www.dropbox.com/referrals/AADNvoU8p9INJLaRJM0B_ZE1M2xRY33OD2U?src=global9" class="btn btn--inverse btn--x-large align-center">點我立即取得額外的儲存空間</a>
{% endcapture %}

<div class="notice--primary">{{ JoinDropbox | markdownify }}</div>

### Step 2: 在 Dropbox App Console 新增設定 Application 獲取可用的應用程式憑證

為了使用 Dropbox API，你同時會需要設定一個可用的 Application 允許授權存取你 Dropbox 中的資料，並且賦予必要的權限。可以透過以下連結訪問 Dropbox App Console：

- [Dropbox App Console](https://www.dropbox.com/developers/apps)

點擊 "Create App" 並且輸入必要資訊：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/app-console.png" alt="Dropbox App Console" caption="Dropbox App Console" %}

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/create-app.png" alt="建立 Dropbox App" caption="建立 Dropbox App" %}

- **Choose the type of access you need**: 可以選擇只允許訪問特定的 App Folder 或是 Full dropbox。因為我們要用來搬家，所以這裡選擇 **Full dropbox**。
- **Name**: 應用程式的唯一識別名稱，不能與其他使用者重複，如果已經被註冊系統會提示。

如此一來你就可以獲得一個可用的應用了，預設情況下，你可以透過 **"Generate Access Token"** 的按鈕取得臨時性的憑證，並且可以在 Dropbox Python 程式碼中，使用 SDK 提供的方法帶入產生的 access token 初始化 Dropbox 用戶端：

```python
dbx = dropbox.Dropbox('YOUR_ACCESS_TOKEN')
```

由於憑證預設是 4 小時到期，到期後會需要重新產生。因此，如果需要的執行時間長一點，可以在 **"Access token expiration"** 選擇 **"No expiration"**。

(**但請注意不要把這組憑證外流，否則可能其他人都能任意存取甚至修改你 Dropbox 中的資料**)

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/get-access-token.png" alt="產生存取憑證 (Access token)" caption="產生存取憑證 (Access token)" %}

#### 權限設定

由於在設計 Python 上傳檔案的應用中，操作中包含了檔案操作的行為，因此，別忘記到 **"Permissions"** 頁面中選擇需要的檔案寫入的權限 (`files.content.write`)：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/configure-permission.png" alt="設定必要的檔案寫入權限" caption="設定必要的檔案寫入權限" %}

如果沒有設定，很可能會在執行上傳相關的方法 (`files_upload()`, `upload_session`) 可能會遭遇到以下的錯誤訊息：

```python
BadInputError: BadInputError('xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx', 'Error in call to API function "files/upload_session/start": Your app is not permitted to access this endpoint because it does not have the required scope \'files.content.write\'. The owner of the app can enable the scope for the app using the Permissions tab on the App Console.')
```

### Step 3: 建立 Google Colaboratory 並且在 CoLab 執行環境中使用 DropBox Python SDK 轉移資料

下一步就是可以直接在目標的 Google Drive 帳號中運行 Colab 環境。CoLab [支持掛載 Google Drive](https://colab.research.google.com/notebooks/io.ipynb#scrollTo=u22w3BFiOveA) 在執行環境中當成本地端硬碟直接使用，透過這種方式能夠直接在 CoLab 環境中指定要執行備份的目錄工作。

透過在要轉移的 Google Drive 中新增一個 Google Colaboratory 的應用選項，可以直接啟用一個執行環境：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/add-colab.png" alt="新增 Google Colab 應用" caption="新增 Google Colab 應用" %}

如果沒有看到 Colab 選項的話，可以透過連結更多應用的選項安裝 Google Colaboratory：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/connect-more-app.png" alt="在 Google Drive 連結更多應用" caption="在 Google Drive 連結更多應用" %}

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/install-colab.png" alt="在 Google Drive 安裝 Google Colaboratory" caption="在 Google Drive 安裝 Google Colaboratory" %}

接下來就可以直接透過 Colab 掛載 Google Drive：

```python
from google.colab import drive
drive.mount('/gdrive')
```

同時也能透過 PyPi 安裝 Dropbox SDK：

```python
!pip install dropbox
```

點擊運行就能將 Google Drive 掛載 (`/gdrive/MyDrive`)：

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/colab-mount-example.png" alt="在 Colab 掛載 Google Drive" caption="在 Colab 掛載 Google Drive" %}

掛載後就能透過 Dropbox 提供的 API 直接執行檔案上傳，並且透過 Colaboratory 的執行環境可以跑在雲端的背景中執行，完全不需要下載檔案並且將檔案從 Google Drive 搬家到 Dropbox。

{% include figure image_path="/assets/images/posts/2022/01/moving-files-from-google-drive-to-dropbox-hack/execution-example.png" alt="執行範例" caption="執行範例" %}

## 總結

在這篇內容中，我與你分享了非手動檔案搬家的解決方式。在這篇內容中，提幾了幾個簡單的步驟，幫助你不必從 Google Drive 下載檔案，並將檔案轉移備份。並且使用 Google Colaboratory 執行 Dropbox 遷移作為範例。

如果你覺得這樣的內容有幫助，可以在底下按個 Like / 留言讓我知道。

## References

- [1] [Dropbox for Python](https://dropbox-sdk-python.readthedocs.io/en/latest/api/dropbox.html?highlight=upload)
- [2] [Data transport limit](https://www.dropbox.com/developers/reference/data-transport-limit)


