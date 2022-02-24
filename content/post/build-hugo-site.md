---
title: "Build Hugo Site"
date: 2022-02-24T03:37:48+08:00
draft: false
tags: ["hugo", "全自動部署", "github", "action", "github pages", "web"]
categories: ["架設hugo靜態網站"]
banner: https://d33wubrfki0l68.cloudfront.net/c38c7334cc3f23585738e40334284fddcaf03d5e/2e17c/images/hugo-logo-wide.svg

---

# 建立全自動部屬的hugo靜態網站
本篇最後成果會是只要pushcommit後，就會自動部屬到git-pages上，直接進行blogger的更新。
* 發布文章流程:
> 寫文章->git push->blogger看成果
* 開發及部屬流程:
> 寫文章->本地hugo server預覽->gitpush到gihutb上->github-action被觸發建構hugo靜態網站到git-pages->成功發布文章
## 大綱
這篇文章內容:
1. 安裝hugo
2. 創建一個hugo專案
3. 兩個github repo
   1. repo: blogger(包含兩個分支)
      1. branch: blogger本體分支
      2. branch: 佈署分支
   2. repo:theme
      * branch: 對喜歡的theme fork, 對於主題的修改都會新增在這邊。
4. github pages 
   * github寄放靜態網站功能
5. github action
   * 用於建構並佈署
## 安裝hugo

詳細內容可以看[官網](https://gohugo.io/getting-started/installing/)
1. 設定[chocolatey](https://chocolatey.org/install):
用系統管理者開啟powershell後輸入
``` 
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
2. 安裝hugo
```
choco install hugo -confirm
```
## 創建hugo專案
``` 
REM 創建專案
hugo new site quickstart

REM 使用submodule管理theme，路徑放置在themes/ananke
cd quickstart
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke

REM 在config.toml上補上theme = "ananke"
echo theme = "ananke" >> config.toml

REM 寫文章，預設路徑在content資料夾下
hugo new posts/my-first-post.md

```

會出現這樣的檔案內容，本地伺服器會忽略draft的設定，當要放到真的網站上的時候要把**draft設定成false**
```
---
title: "My First Post"
date: 2022-02-24T23:42:50+08:00
draft: true
---
your content
```
你也可以選擇使用其他[hugo的themes](https://themes.gohugo.io/), config.toml的theme名稱記得要改成你資料夾的名稱themes/**ananke**



啟動hugo server:
``` 
hugo server -D
```
從輸出訊息可以找出網址位址
``` 
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```
___

以上都是官網簡單的教學，接下來的步驟基本上建議參考官網上的資訊，這邊紀錄其他比較細節的東西。

英文:https://gohugo.io/hosting-and-deployment/hosting-on-github/

基本上是建議先使用`https://<USERNAME|ORGANIZATION>.github.io/`去創建你的github

## github
### git repo架構:
   1. repo: blogger 文章存放用的: `https://<USERNAME|ORGANIZATION>.github.io/`
         * main: 存放你的hugo文章本體的分支
         * pg-pages: 當你push文章到main後，觸發git action部屬的分支
   2. repo: github hugo themes : 先使用現有的`https://github.com/theNewDynamic/gohugo-theme-ananke.git`
      * 未來有修改theme架構或是圖片的時，可以自己fork一個theme，並將修改push到github上

#### blogger repo
創建github repo後照著quick setup將你本地hugo資料夾用git init後並推送到github上
修改
1. **main分支**

[修改config.toml的baseURL](https://gohugo.io/hosting-and-deployment/hosting-on-github/#change-baseurl-in-configtoml)

baseURL: `https://<USERNAME>.github.io/<REPOSITORY_NAME>`

``` 
echo "this is for hugo content" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/<USERNAME>/<USERNAME>.github.io.git
git push -u origin main
```
2. **pg-pages分支**
```
## 建立空分支
git checkout --orphan gh-pages
# 清除分支初始檔案
git reset --hard
## 建立
echo "this is for depoly branch" > "README.md"
git add .
git commit -m "gh-pages first commit"
git push -u origin gh-pages
```

> [創建空分支: git --orphan](https://stackoverflow.com/questions/13969050/creating-a-new-empty-branch-for-a-new-project)


### github-action
基於Hugo setup去製作github action，

[官網說明:](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action)
本地端建立`.github/workflows/gh-pages.yml`檔案，可以發現jobs->deploy 使用了setup Hugo的action

這邊的github_tokeh先在個人的setting中連結`https://github.com/settings/tokens`選擇workflow後產生token，請記好你的token，複製起來放到你的`https://github.com/<USERNAME>/<USERNAME>.github.io.git/settings/secrets/actions`

```
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

切回main分支開始處理hugo設定，後續就不太需要管gh-pages了，他會被自動部屬。
```
git checkout main

# commit 後push到main分支
git add .github/workflows/gh-pages.yml
git commit -m "add github action scripts"
git push -u origin main
```

### github-pages
請到`https://github.com/<USERNAME>/<USERNAME>.github.io.git/settings/pages` 將你的 branch改成gh-pages

## 結論

本篇主要說明github分支建立方式，透過main分支做hugo設定，並透過gh-pages用於自動部屬。
依照本篇設定完之後後續在main分支上透過
1. `hugo new posts/post_name.md`撰寫文章
2. 寫完後執行
```
git add .
git commit -m "message"
git push -u origin main
```
3. 等待git action執行完成，blog就會更新了
4. 後續建議可以自己fork一個theme，然後把自己的theme git origin更新，就可以客製化你的網站了。
5. 在theme/你的theme資料夾下更新url
` git remote set-url origin git@github.com:user/your_theme.git`
6. 在修改theme資源並commit後，你必須同時更新blogger main分支下的.gitmodules並push到github上，這樣github上的部屬才會生效。
## 參考資源: 
https://gohugo.io/getting-started/external-learning-resources/

其中簡單點的可以參考MikeDane去看一下基本架構
https://www.youtube.com/watch?list=PLLAZ4kZ9dFpOnyRlyS-liKL5ReHDcj4G3&v=qtIqKaDlqXo