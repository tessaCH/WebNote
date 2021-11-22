---
title: GitHub - SSH keys
---

###### tags: `GitHub`, `SSH`

# GitHub - SSH keys 連線設置（Mac）
 - [取得SSH金鑰](#取得SSH金鑰)
 - [設置SSH金鑰](#設置SSH金鑰)
 - [設定GitHub的SSH連線機制](#設定GitHub的SSH連線機制)

### 前言
```zsh
$ git clone https://github.com/ooo/xxxxx.git
Cloning into 'xxxxx'...
> Username for 'https://github.com': xxx
> Password for 'https://xxx@github.com': 
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: Authentication failed for 'https://github.com/ooo/xxxxx.git/'
```
因為換了新的MacBook M1，在clone非公開repo時用上方的步驟，驗證完帳號密碼會出現錯誤訊息。
大意就是在2021/8/13開始，不再支援密碼驗證，只能使用個人的token進行驗證…。
所以才有這篇記錄如何使用SSH驗證的方式存取GitHub repo。

***

## 取得SSH金鑰
- 列出.ssh資料夾中的檔案
    `$ ls -al ~/.ssh`
- 如果像下列，表示已有公用SSH金鑰
    ```zsh
    total 24
    drwx------   5 xxx  staff   160 Nov 22 20:43 .
    drwxr-xr-x+ 38 xxx  staff  1216 Nov 22 20:39 ..
    -rw-r--r--@  1 xxx  staff    78 Nov 22 20:43 config
    -rw-------   1 xxx  staff   464 Nov 22 20:40 id_ed25519
    -rw-r--r--   1 xxx  staff    99 Nov 22 20:40 id_ed25519.pub
    ```
 - GitHub支援的公用金鑰種類如下：
    - id_rsa.pub
    - id_ecdsa.pub
    - id_ed25519.pub

    (已有符合種類的公用金鑰，可直接跳到[下一步驟](#設置SSH金鑰))

- 沒有的話會得到下列訊息
    ```zsh
    ls: /Users/xxx/.ssh: No such file or directory
    ```
- 需要產生一組ssh金鑰 (rsa / ecdsa / ed25519皆可用，此處以ed25519為例)
    ```zsh
    $ ssh-keygen -t ed25519 -C "your@email.com"
    Generating public/private ed25519 key pair.
    > Enter file in which to save the key (/Users/xxx/.ssh/id_ed25519):
    [這邊直接按Enter，會自動生成資料夾到預設路徑]
    Created directory '/Users/xxx/.ssh'.
    > Enter passphrase (empty for no passphrase): [設定這組金鑰的密碼]
    > Enter same passphrase again: [再次輸入確認密碼]
    Your identification has been saved in /Users/xxx/.ssh/id_ed25519.
    Your public key has been saved in /Users/xxx/.ssh/id_ed25519.pub.
    The key fingerprint is:
    SHA256:***************************** your@email.com
    The key's randomart image is:
    +--[ED25519 256]--+
    | ***o*           |
    |. ++.oo          |
    |o=***...         |
    |*******o         |
    |o**.o.+ S        |
    |o** *.           |
    |* .o.            |
    | **=.+           |
    |. =o**.          |
    +----[SHA256]-----+
    ```

## 設置SSH金鑰
- 在背景中啟用ssh-agent (如果帳號無管理者權限，可能會需要先啟用root存取權限)
    ```zsh
    $ eval "$(ssh-agent -s)"
    Agent pid 9265
    ```
- 如果使用macOS Sierra 10.12.2以上版本，需修改~/.ssh/config以使ssg-agent能讀取金鑰、並將passphrases設定的密碼存入鑰匙圈(keychain)中
    - 確認是否有~/.ssh/config設定檔
    `$ open ~/.ssh/config`
    - 沒有的話會得到下列訊息
    `The file /Users/xxx/.ssh/config does not exist.`
    - 需要生成該檔案再開啟
    `$ touch ~/.ssh/config`
    - 開啟後，內容應該包含下列文字 (~/.ssh/id_ed25519為私鑰檔案路徑)
        ```zsh
        Host *
          AddKeysToAgent yes
          UseKeychain yes
          IdentityFile ~/.ssh/id_ed25519
        ```
- 確認設定檔無誤，即可將ssh私鑰加入ssh-agent中
`$ ssh-add -K ~/.ssh/id_ed25519`

## 設定GitHub的SSH連線機制
- 確認設定檔無誤，即可將ssh私鑰加入ssh-agent中
    - 執行下列指令，會將公鑰內容存入到剪貼簿中(再command-v即貼上)
    `$ pbcopy < ~/.ssh/id_ed25519.pub`
    - 或執行下述指令，將公鑰內容輸出後，再自行複製
        ``` zsh
        $ cat ~/.ssh/id_ed25519.pub     
        ssh-ed25519 ************************* your@email.com [複製這行]    
        ```
- 登入GitHub頁面，點擊 個人圖像 -> Settings -> [SSH and GPG keys](https://github.com/settings/keys) -> [new SSH key](https://github.com/settings/ssh/new)
    - 將前一步複製的公鑰貼到Key下方的框框，在Title輸入想要的名稱(ex:MacBook)
    - 點擊"Add SSH key"以新增SSH金鑰
    - 可能會跳出GitHub的密碼確認視窗，輸入後即完成新增金鑰

- 允許以ssh方式與GitHub連線
    ``` zsh
    $ ssh -T git@github.com
    The authenticity of host 'github.com (13.114.40.48)' can't be established.
    ECDSA key fingerprint is SHA256:******************************************.
    > Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added 'github.com,13.114.40.48' (ECDSA) to the list of known hosts.
    Hi username! You've successfully authenticated, but GitHub does not provide shell access.   
    ```

- 完成後，需要clone repo時，記得要先選到SSH再複製(選到HTTPS就不是用ssh機制囉)
`git clone git@github.com:xxx/xxx.git`

***

reference:
- https://docs.github.com/en/authentication/connecting-to-github-with-ssh