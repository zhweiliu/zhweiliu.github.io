---
draft: true
title: "GitHub Pages 設定 Custom Domain ( Go Daddy )"
date: 2023-01-05T22:42:35+08:00
categories: ["Hugo"]
keywords: ["Hugo", "GitHub Pages", "Custom Domain"]
summary: |
  這篇文章主要提供 GitHub Pages 設定 custom domain 的過程
---
*這篇文章主要提供 GitHub Pages 設定 custom domain 的過程*

**GitHub Pages** 提供了可以設定 **custom domain** 的方法，因此想要透過 custom domain 設定，將 GitHub Pages 的 URL 更改為自己的 domain。

## 申請 Domain
有很多 provider 提供註冊 domain 的服務，如 [Google Domain](https://domains.google/intl/zh_tw/)、[Go Daddy](https://tw.godaddy.com) 等等。

我自己是在 GoDaddy 上購買 Domain ， 因此這篇文章記錄的是在 Go Daddy 上的設定方式。

---

## GitHub Pages Custom Domain
根據 GitHub Pages 的[文件](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain)説明，設定 subdomain 的方式相對簡單明白

> *GitHub Pages Repository > Settings > Pages (左側選單) > Custom Domain > 輸入 subdomain > Save*

這時會看到 Custom Domain 下方出現正在驗證 domain 的字樣，等 DNS Record 設定好之後，幾分鐘內就會出現驗證通過的綠色字樣。

---

## GoDaddy DNS Record 設定

在 GoDaddy 的 DNS 紀錄頁面，新增一筆 `C Name` Record，給予想要的 subdomain 名稱，如: `www`  、 `blog` ，內容值填入 GitHub Pages URL ，如 : `zhweiliu.github.io.` ，TTL 時間可以留預設值，之後儲存以新增紀錄即可

![CNAME Record](/images/hugo/custom_domain/CNAME_Record.png)

---

## Enable GitHub Pages Enforce Https

在 terminal 中使用指令確認 DNS Record 的設置是否讓 GitHub Pages Custom Domain 生效
```bash
$ dig blog.zhengweiliu.com
```

若設置已經生效，你會看到如下圖所示的 **Answer Section**
![](/images/hugo/custom_domain/dig_subdomain.png)

> 將 blog.zhengweiliu.com 置換為你設定 subdomain  
假設你設定 CNAME 得名稱為 www ， 你購買的 domain name 是 example.com ，那你需要 dig www.example.com

回到先前 GitHub Pages Custom Domain 設定的頁面，刷新網頁後將 **Enforce HTTPS** 打勾，以確保 `http://` 開頭的請求都能夠被強制轉為 `https://`

![GitHub Pages Custom Domain Setup](/images/hugo/custom_domain/github_pages_setup_custom_domain.png)

---