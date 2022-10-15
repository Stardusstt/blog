---
title: 連筆電都懶得帶? 那就在 Android 上跑 VS Code 吧! | Termux , PRoot , VS Code Server
date: 2022-08-14 02:13:59
tags:
    - Android
    - Termux
    - Linux
    - Visual Studio Code
categories:
    - Android
keywords:
    - Android
    - VS Code Server
    - Termux
    - Linux
    - PRoot
description: 
cover: https://i.imgur.com/5kqtPtu.png
---


# 前言

最近發現隨著東西越看越多，有些東西還是會忘，雖然說有加書籤整理，但這樣翻起來還是有點雜，
所以決定來寫文章，當作筆記紀錄，由於是一開始，就挑個輕鬆有趣的主題 ?

至於為甚麼要在手機上跑 Visual Studio Code ?
很簡單，就是懶，廢話不多說讓我們來開始吧!


# 簡介

由於本篇是屬於輕鬆有趣的文章，所以跳過原理部分。

本次主要會用到的東西
* [Termux](https://github.com/termux/termux-app) - Android 上的終端模擬器
* [PRoot Distro](https://github.com/termux/proot-distro) - wrapper for proot
* [Visual Studio Code Server](https://code.visualstudio.com/docs/remote/vscode-server)

原理是藉由在 Termux 上使用 proot 安裝 Linux，並在 Linux 上安裝 Visual Studio Code Server，
然後由手機的瀏覽器 (e.g. Chrome) 顯示。


# Termux 環境建置

## 安裝 Termux

由於 Play Store 的版本[已經棄用](https://github.com/termux/termux-app#google-play-store-deprecated)，我們可以選擇 [F-Droid](https://f-droid.org/en/packages/com.termux/) 或 [Github](https://github.com/termux/termux-app) 的版本

> **注意:** Github 版本為 [debuggable](https://github.com/termux/termux-app#github)，與 F-Droid 不通用。

## 存取手機的儲存空間

為了讓我們可以[從 Termux 存取手機的儲存空間](https://wiki.termux.com/wiki/Internal_and_external_storage)，需要進行設定，
打開 Termux，輸入指令，執行後會跳出 Android 的詢問，同意即可

```bash
termux-setup-storage
```
![termux-setup-storage](https://i.imgur.com/uwjzLjS.jpg)

## 更新

這裡我習慣用 apt

```bash
apt update
```
![apt update](https://i.imgur.com/6M5AvDn.jpg)


```bash
apt upgrade
```
記得輸入 y 後繼續

![apt upgrade](https://i.imgur.com/cGkmD8N.jpg)

<br/>

途中可能會跳出設定檔版本衝突，如果你沒有做甚麼更改，用 `package maintainer's version` 即可，輸入 Y

![configuration file](https://i.imgur.com/I3cvIiW.jpg)


## 安裝 OpenSSH

```bash
apt install openssh
```
![apt install openssh](https://i.imgur.com/v3AOTht.jpg)



# 配置 SSH server，從電腦打指令 (可選)

為了方便，我們可以配置 [SSH server](https://wiki.termux.com/wiki/Remote_Access)，從電腦上打指令比較方便(本文設備在同一個 local network 下)
當然，你也可以在手機上處理

## 配置密碼

等一下從電腦登入會用到

```bash
passwd
```
![passwd](https://i.imgur.com/8xGBZEu.jpg)

## 啟動 OpenSSH server

`啟動` OpenSSH server

```bash
sshd
```

若要`關閉` OpenSSH server

```bash
pkill sshd
```

## 確認使用者

確認要登入的使用者

```bash
whoami
```
![whoami](https://i.imgur.com/t2CJp0o.jpg)

## 確認手機 IP

確認手機的 LAN IP，可輸入 `ifconfig`，你也可以從其他地方看

```bash
ifconfig
```

## 從電腦登入

Windows 10 1809 後已經支援 openssh
從電腦登入，Termux 預設的 `SSH port` 為 `8022`

```powershell
ssh -p 8022 user@hostname_or_ip
```
![ssh](https://i.imgur.com/f8QXT28.png)



# Linux 環境建置

## 安裝 proot-distro

安裝 [proot-distro](https://github.com/termux/proot-distro)

```bash
apt install proot-distro
```
![apt install proot-distro](https://imgur.com/CCLrsuS)

## proot-distro list

列出可以用的 Linux distribution、安裝狀態等資訊

```bash
proot-distro list
```
![proot-distro list](https://i.imgur.com/qkZ4u8y.png)

## 安裝 Linux 發行版

安裝你想要的 Linux 發行版，這裡我選擇 ubuntu

```bash
proot-distro install ubuntu
```
![proot-distro install ubuntu](https://i.imgur.com/g8rUA3h.png)
![proot-distro install ubuntu](https://i.imgur.com/UCsrcnn.png)

## 登入

```bash
proot-distro login ubuntu
```
![proot-distro login ubuntu](https://i.imgur.com/7CKXSQC.png)

## 更新

依照慣例，先更新一下

```bash
apt update && apt upgrade
```
![apt update && apt upgrade](https://i.imgur.com/Bewfflj.png)

## 安裝 wget

安裝 wget，稍後會用到

```bash
apt install wget
```
![apt update && apt upgrade](https://i.imgur.com/QoclSV6.png)



# 配置 Visual Studio Code Server

## 安裝 Visual Studio Code Server

安裝 [Visual Studio Code Server](https://code.visualstudio.com/docs/remote/vscode-server)

```bash
wget -O- https://aka.ms/install-vscode-server/setup.sh | sh
```
![install VS Code Server](https://i.imgur.com/4prJUF3.png)

## 列出可用指令

我們可以用 `--help` 列出可用指令，這裡擷取一部份

```bash
code-server -h
```
![code-server -h](https://i.imgur.com/xUjwgk6.png)
![code-server -h](https://i.imgur.com/IQfzrz0.png)

## serve-local

這次我們要使用的是 `serve-local`，因為 VS Code Server 的服務還是在 private preview，要填表單申請
而且 local 畢竟比較穩定，當然你還是可以去[申請](https://code.visualstudio.com/docs/remote/vscode-server#_how-can-i-get-access-to-the-vs-code-server)

一樣列出可用指令，這裡擷取一部份

```bash
code-server serve-local -h
```
![serve-local -h](https://i.imgur.com/t5Wdd4L.png)
![serve-local -h](https://i.imgur.com/i2WYy43.png)

我們可以自行指定 port , bound interface，VS Code 版本等等...
interface 部分，我自己測試，如果讓他 bind 0.0.0.0 從外部(非 localhost)直接連，
會有部分東西顯示不出來(e.g. extension's details page)，所以還是走預設讓他 bind 127.0.0.1

## 啟動 Visual Studio Code Server

版本我選 `insiders`，首次啟動會要你同意授權條款，同意即可

```bash
code-serve serve-local --quality insiders
```
![code-serve serve-local --quality insiders](https://i.imgur.com/Ry8EWWX.png)

如果要停止，使用 `ctrl+c`

## 進入 Web UI

在瀏覽器開啟那串 URL 即可

![URL](https://i.imgur.com/swVyozA.png)

{% gallery %}
![Web UI](https://i.imgur.com/Ig3JZHm.jpg)
{% endgallery %}



# 補充 - 從電腦連手機的 Visual Studio Code Server

為了方便，你可以選擇在電腦上處理配置，但由於前面提到的原因，直接連會有些顯示問題，
這裡我們使用 SSH Tunneling 解決

## Local Port Forwarding

電腦再開一個 Terminal，使用中保持其存在，從前面得知預設 port 為 `8000`

Local Port Forwarding 指令語法
```powershell
ssh -L [bind_address:]<port>:<host>:<host_port> <SSH Server>
```

但別忘了 Termux 預設的 SSH port 為 `8022`

![Local Port Forwarding](https://i.imgur.com/sKsB7Kv.png)

## 進入 Web UI

在瀏覽器開啟那串 URL 即可

![URL](https://i.imgur.com/UqOJfGB.png)



# 補充 - 全螢幕

目前找到比較簡單的方法，但要注意產出的桌面捷徑的 `URL` 是固定的，如果 `tkn` 改變了，捷徑會失效

{% gallery %}
![fullscreen](https://i.imgur.com/RGunNs2.jpg)
{% endgallery %}



# 補充 - 安裝 C,C++,debug 相關工具

基本上就跟在 Linux 安裝一樣，既然都裝了就順便寫一下

## 安裝

```bash
apt install build-essential gdb
```

## 確認 g++ 版本

```bash
g++ --version
```
![g++ --version](https://i.imgur.com/Tf9rAXO.png)



# 補充 - 安裝 Python

## 安裝

```bash
apt install python3
```
![apt install python3](https://i.imgur.com/SCDsTE8.png)

## 確認 Python 版本 

```bash
python3 --version
```
![python3 --version](https://i.imgur.com/eQJvTi6.png)



# 補充 - 存取手機儲存空間

我們可以看到，ubuntu 這邊是自己的空間

![ls -a](https://i.imgur.com/XfAShrG.png)

往上層走可以看到手機儲存空間 `sdcard`

![cd ..](https://i.imgur.com/pWuN4LQ.png)

建議檔案還是放在 ubuntu 的空間下，(可建個資料夾整理)，避免 coding 時出現權限等奇怪的問題，要拿檔案時再複製



# Demo 展示

## 手機

{% gallery %}
![demo_phone](https://i.imgur.com/YrVGB3e.jpg)
![demo_phone](https://i.imgur.com/d0yBLKh.jpg)
![demo_phone](https://i.imgur.com/MV6tkyk.jpg)
{% endgallery %}

## 電腦

{% gallery %}
![demo_phone](https://i.imgur.com/ZhQL2jT.png)
![demo_phone](https://i.imgur.com/bLJTyxA.png)
{% endgallery %}



# 結語

可能有人會問為甚麼不用 VNC，一來是 proot 有效能折損，不如直接在手機瀏覽器跑，
再者是因為 Android 12，使用 VNC 等比較吃 CPU 東西的時候，Termux 可能會遇到 `Signal 9` 的問題
* [Android Phantom, Cached And Empty Processes](https://gist.github.com/agnostic-apollo/dc7e47991c512755ff26bd2d31e72ca8)
* [issue #2366 comment-1009269410](https://github.com/termux/termux-app/issues/2366#issuecomment-1009269410)


話說不知不覺就寫了一大篇，如果想再更方便一點也可以加個藍芽鍵盤，或是拿平板，
Visual Studio Code 的 extensions 大部分都可以正常安裝，用作簡單的 coding，還是蠻方便的



# References

* https://github.com/termux/termux-app
* https://wiki.termux.com/wiki/Internal_and_external_storage
* https://wiki.termux.com/wiki/Remote_Access
* https://github.com/termux/proot-distro
* https://code.visualstudio.com/docs/remote/vscode-server
* https://johnliu55.tw/ssh-tunnel.html
* https://code.visualstudio.com/docs/cpp/config-linux
















