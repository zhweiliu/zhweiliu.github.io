---
title: "Hugo-PaperMod theme 設定"
date: 2023-01-09T14:27:41+08:00
categories: ["Hugo", "PaperMod"]
keywords: ["PaperMod", "theme"]
summary: |
  這篇文章整理了個人使用 PaperMod theme 的設定
---
*這篇文章整理了個人使用 PaperMod theme 的設定*

## What is Hugo-PaperMod ?

> [Hugo PaperMod](https://github.com/adityatelange/hugo-PaperMod) is a theme based on hugo-paper. The goal of this project is to add more features and customization to the og theme.

Hugo-PaperMod 使用 `yml/yaml` 格式提供所有示例，作者認為 `yaml` 的格式會比 `toml` 容易閱讀。

---

### 安裝方式

Hugo 官方建議採用 submodule 的方式來安裝 hugo themes；
使用 submodule 來安裝 Hugo-PaperMod theme

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```

若要更新 Hugo-PaperMod theme :
```bash
git submodule update --remote --merge
```


Hugo-PaperMod 也提供基本的 [config.yml](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation#sample-configyml) 和 [page.md](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation#sample-pagemd)


---

## Features

Features 設置參考來源
[架站║ Hugo部落格與PaperMod主題](https://www.lilmp.com/2022-06-22/hugo-papermod-blog/)

### 建立 Archive

利用指令在 `content` 下建立 `archive.md`
```bash
hugo new content/archive.md
```

編輯 `archive.md` 內容
```yml
---
title: "Archive"
layout: "archives"
url: "/archives/"
summary: archives
---
```

在 `config.yml` 的 `menu` 區塊設置 archive 按鈕
```yml
menu:
  main:
    - identifier: archives
      name: 📚 Archives
      url: /archives/
      weight: 10
```

---

### 建立 Search Page

在 `config.yml` 加入以下設定
```yml
outputs:
  home:
    - HTML
    - RSS
    - JSON # is necessary
```

利用指令在 `content` 下建立 `search.md`
```bash
hugo new content/search.md
```

編輯 `search.md` 內容
```yml
---
title: "Search" # in any language you want
layout: "search" # is necessary
url: "/search/"
# description: "Description for Search"
summary: "search"
placeholder: "placeholder text in search input box"
---
```

對於不希望被 search page 搜尋到的文章，可以在文章開頭的 `archtype` 加入以下設定

```yml
searchHidden: true
```

#### Setup Post Keywords

在文章開頭的 `archtype` 加入以下設定
```yml
keywords: ["keyword 1", "keyword 2", ...]
```

#### Customizing Fusejs Options

PaperMod 選用 `Fusejs` 作為 search 元件，參考[Fusejs 參數項](https://fusejs.io/api/options.html)來自定義 search page 功能，如
```yml
params:
  fuseOpts:
    isCaseSensitive: false
    shouldSort: true
    location: 0
    distance: 1000
    threshold: 0.4
    minMatchCharLength: 0
    keys: ["title", "permalink", "summary", "content"]
```

### Code Block 設定 max-height
在 `assets/css/common/post-single.css` 添加下面內容
```css
.post-content pre code {
    max-height: 30em;
}
```


---

## Shortcode

>Shortcode 可以看作是「一小塊 HTML 程式片段」，與 Hugo Template 不同的是，前者通常運用在「插入特定用途」、「重複使用」的片段語法到 markdown 內容中，而後者則是作為 markdown content 的外殼載體、或是佈局規劃等，用以構成我們最後呈現的視圖頁面 (View)。 — [來源:iT邦](https://ithelp.ithome.com.tw/articles/10249470)

更詳細的 shortcode 設定範例可以參考 [Day 22. Hugo Shortcode 介紹](https://ithelp.ithome.com.tw/articles/10249470) 和 [Hugo博客自定义shortcodes](https://www.sulvblog.cn/posts/blog/shortcodes/)

以下僅整理我目前有使用的

### PDP Shortcode

在 `layouts/shortcodes/` 目錄下建立一個新文件 `pdf.html`，貼上下列內容
```html
<!DOCTYPE HTML>
<html lang="en">
<head>
    <style type="text/css">
        #googleslides_shortcodes {
            padding-bottom: 66%;
            position: relative;
            display: block;
            width: 100%;
            border-bottom: 5px solid;
        }
        #googleslides_shortcodes iframe {
            position: absolute;
            top: 0;
            left: 0
        }
    </style>
    <title></title>
</head>
<body>
<div id="googleslides_shortcodes">
    <iframe id="googleSlideIframe"
            width="100%"
            height="100%"
            src="{{ .Get "src" }}"
            frameborder="0"
            allowfullscreen="" >
    </iframe>
</div>
</body>
</html>
```

**使用方式**
```
{a{<pdf src="pdf網址 | 站內 pdf 位址">}}
# 使用的時候把字母 a 去掉；這邊加入 a 是防止被識別生效
# 把 pdf file 放在 static/{目錄名}/ 下，可以直接使用 /{目錄名}/{檔案名}.pdf 指定使用站內 pdf
```

{{<pdf src="/files/common_linux_command.pdf">}}


### Youtube Shortcode

在 `layouts/shortcodes/` 目錄下建立一個新文件 `youtube.html`，貼上下列內容
```html
<!DOCTYPE HTML>
<html lang="en">
<head>
    <style type="text/css">
        .youtube_shortcodes {
            position: relative;
            width: 100%;
            height: 0;
            padding-bottom: 66%;
            margin: auto;
            overflow: hidden;
            text-align: center;
        }
        .youtube_shortcodes iframe {
            position: absolute;
            width: 100%;
            height: 100%;
            left: 0;
            top: 0;
        }
    </style>
    <title></title>
</head>
<body>
<div class="youtube_shortcodes">
    <iframe
            class="youtube-player"
            type="text/html"
            width="640"
            height="385"
            src="https://www.youtube.com/embed/{{ index .Params 0 }}?autoplay=0"
            style="
                 position: absolute;
                 top: 0;
                 left: 0;
                 width: 100%;
                 height: 100%;
                 border:0;"
            allowfullscreen frameborder="0">
    </iframe>
</div>
</body>
</html>
```

**使用方式**
```
{a{<youtube _XbJhL7WsE8>}}
# 使用的時候把字母 a 去掉；這邊加入 a 是防止被識別生效
# _XbJhL7WsE8 是 Youtube 分享鏈結中的最後一段識別碼
{{< figure src="/files/youtube_share_link.png" width="50%" >}}
```
{{<youtube _XbJhL7WsE8>}}


### Blog 站內文章
在 `layouts/shortcodes/` 目錄下建立一個新文件 `innerlink.html`，貼上下列內容

```html
<div style="height: 200px;margin: 1em auto;position: relative;
        box-shadow: 0 2px 4px rgb(0 0 0 / 25%), 0 0 2px rgb(0 0 0 / 25%);
        border-radius: 15px;padding: 23px;max-width: 780px;background: var(--entry);">
    {{ $url := .Get "src" }}
    {{ with .Site.GetPage $url }}
    <div style="font-size: 22px; font-weight: 600">
        <a target="_blank" href="{{ .Permalink }}" style="box-shadow: none">{{ .Title }}</a>
    </div>
    <span style="font-size: 14px; color: #999">
        Date: {{ .Date.Format ( default "2006-01-02") }}
        {{ if .Params.categories }}&nbsp;
        Categories:
        {{ range .Params.categories }}
            #{{ . }}&nbsp;
        {{ end }}
    </span>
    <div style="font-size: 14px; line-height: 1.825;max-height: 75px; overflow: hidden;margin-top: 5px;">
        {{ .Summary | plainify}} ......
    </div>
    {{ end }}
    {{ end }}
</div>
```

**使用方式**
```
{a{<innerlink src="posts/hugo/installation.md">}}
# 使用的時候把字母 a 去掉；這邊加入 a 是防止被識別生效
# 直接指定 content/posts/ 下的文章路徑，結尾要加上 .md 檔名

# 卡片獲取的是文章的 summary 內容，默認長度是 70 個中文字
```

{{<innerlink src="posts/hugo/installation.md">}}

---

## Hugo Front Matter 參數說明

[Content Summaries](https://gohugo.io/content-management/summaries/)
> Hugo generates summaries of your content.
With the use of the .Summary page variable, Hugo generates summaries of content to use as a short version in summary views.

---

## 將目錄 (ToC) 改到側邊
參考 [Hugo博客目录放在侧边 | PaperMod主题](https://www.sulvblog.cn/posts/blog/hugo_toc_side/)

>文章内容仅限于PaperMod主题，对于其他主题仅供参考

### 修改 ToC
首先找到 `layouts/partials/toc.html` ， 更換檔案內容如下
```html
{{- $headers := findRE "<h[1-6].*?>(.|\n])+?</h[1-6]>" .Content -}}
{{- $has_headers := ge (len $headers) 1 -}}
{{- if $has_headers -}}
<aside id="toc-container" class="toc-container wide">
    <div class="toc">
        <details {{if (.Param "TocOpen") }} open{{ end }}>
            <summary accesskey="c" title="(Alt + C)">
                <span class="details">{{- i18n "toc" | default "Table of Contents" }}</span>
            </summary>

            <div class="inner">
                {{- $largest := 6 -}}
                {{- range $headers -}}
                {{- $headerLevel := index (findRE "[1-6]" . 1) 0 -}}
                {{- $headerLevel := len (seq $headerLevel) -}}
                {{- if lt $headerLevel $largest -}}
                {{- $largest = $headerLevel -}}
                {{- end -}}
                {{- end -}}

                {{- $firstHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers 0) 1) 0)) -}}

                {{- $.Scratch.Set "bareul" slice -}}
                <ul>
                    {{- range seq (sub $firstHeaderLevel $largest) -}}
                    <ul>
                        {{- $.Scratch.Add "bareul" (sub (add $largest .) 1) -}}
                        {{- end -}}
                        {{- range $i, $header := $headers -}}
                        {{- $headerLevel := index (findRE "[1-6]" . 1) 0 -}}
                        {{- $headerLevel := len (seq $headerLevel) -}}

                        {{/* get id="xyz" */}}
                        {{- $id := index (findRE "(id=\"(.*?)\")" $header 9) 0 }}

                        {{- /* strip id="" to leave xyz, no way to get regex capturing groups in hugo */ -}}
                        {{- $cleanedID := replace (replace $id "id=\"" "") "\"" "" }}
                        {{- $header := replaceRE "<h[1-6].*?>((.|\n])+?)</h[1-6]>" "$1" $header -}}

                        {{- if ne $i 0 -}}
                        {{- $prevHeaderLevel := index (findRE "[1-6]" (index $headers (sub $i 1)) 1) 0 -}}
                        {{- $prevHeaderLevel := len (seq $prevHeaderLevel) -}}
                        {{- if gt $headerLevel $prevHeaderLevel -}}
                        {{- range seq $prevHeaderLevel (sub $headerLevel 1) -}}
                        <ul>
                            {{/* the first should not be recorded */}}
                            {{- if ne $prevHeaderLevel . -}}
                            {{- $.Scratch.Add "bareul" . -}}
                            {{- end -}}
                            {{- end -}}
                            {{- else -}}
                            </li>
                            {{- if lt $headerLevel $prevHeaderLevel -}}
                            {{- range seq (sub $prevHeaderLevel 1) -1 $headerLevel -}}
                            {{- if in ($.Scratch.Get "bareul") . -}}
                        </ul>
                        {{/* manually do pop item */}}
                        {{- $tmp := $.Scratch.Get "bareul" -}}
                        {{- $.Scratch.Delete "bareul" -}}
                        {{- $.Scratch.Set "bareul" slice}}
                        {{- range seq (sub (len $tmp) 1) -}}
                        {{- $.Scratch.Add "bareul" (index $tmp (sub . 1)) -}}
                        {{- end -}}
                        {{- else -}}
                    </ul>
                    </li>
                    {{- end -}}
                    {{- end -}}
                    {{- end -}}
                    {{- end }}
                    <li>
                        <a href="#{{- $cleanedID -}}" aria-label="{{- $header | plainify -}}">{{- $header | safeHTML -}}</a>
                        {{- else }}
                    <li>
                        <a href="#{{- $cleanedID -}}" aria-label="{{- $header | plainify -}}">{{- $header | safeHTML -}}</a>
                        {{- end -}}
                        {{- end -}}
                        <!-- {{- $firstHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers 0) 1) 0)) -}} -->
                        {{- $firstHeaderLevel := $largest }}
                        {{- $lastHeaderLevel := len (seq (index (findRE "[1-6]" (index $headers (sub (len $headers) 1)) 1) 0)) }}
                    </li>
                    {{- range seq (sub $lastHeaderLevel $firstHeaderLevel) -}}
                    {{- if in ($.Scratch.Get "bareul") (add . $firstHeaderLevel) }}
                </ul>
                {{- else }}
                </ul>
                </li>
                {{- end -}}
                {{- end }}
                </ul>
            </div>
        </details>
    </div>
</aside>
<script>
    let activeElement;
    let elements;
    window.addEventListener('DOMContentLoaded', function (event) {
        checkTocPosition();

        elements = document.querySelectorAll('h1[id],h2[id],h3[id],h4[id],h5[id],h6[id]');
         // Make the first header active
         activeElement = elements[0];
         const id = encodeURI(activeElement.getAttribute('id')).toLowerCase();
         document.querySelector(`.inner ul li a[href="#${id}"]`).classList.add('active');
     }, false);

    window.addEventListener('resize', function(event) {
        checkTocPosition();
    }, false);

    window.addEventListener('scroll', () => {
        // Check if there is an object in the top half of the screen or keep the last item active
        activeElement = Array.from(elements).find((element) => {
            if ((getOffsetTop(element) - window.pageYOffset) > 0 && 
                (getOffsetTop(element) - window.pageYOffset) < window.innerHeight/2) {
                return element;
            }
        }) || activeElement

        elements.forEach(element => {
             const id = encodeURI(element.getAttribute('id')).toLowerCase();
             if (element === activeElement){
                 document.querySelector(`.inner ul li a[href="#${id}"]`).classList.add('active');
             } else {
                 document.querySelector(`.inner ul li a[href="#${id}"]`).classList.remove('active');
             }
         })
     }, false);

    const main = parseInt(getComputedStyle(document.body).getPropertyValue('--article-width'), 10);
    const toc = parseInt(getComputedStyle(document.body).getPropertyValue('--toc-width'), 10);
    const gap = parseInt(getComputedStyle(document.body).getPropertyValue('--gap'), 10);

    function checkTocPosition() {
        const width = document.body.scrollWidth;

        if (width - main - (toc * 2) - (gap * 4) > 0) {
            document.getElementById("toc-container").classList.add("wide");
        } else {
            document.getElementById("toc-container").classList.remove("wide");
        }
    }

    function getOffsetTop(element) {
        if (!element.getClientRects().length) {
            return 0;
        }
        let rect = element.getBoundingClientRect();
        let win = element.ownerDocument.defaultView;
        return rect.top + win.pageYOffset;   
    }
</script>
{{- end }}
```

然後確認 `layouts/_default/single.html` 是否有引入使用 `toc.html` 。 這邊預設是有引入的，作者寫出來是防止有人自定義文件名稱，導致設定失敗

```html
{{- if (.Param "ShowToc") }}
{{- partial "toc.html" . }}
{{- end }}
```

### 修改 CSS
找到 `css/extended/blank.css` ， 更換檔案內容如下
```css
:root {
    --nav-width: 1380px;
    --article-width: 650px;
    --toc-width: 300px;
}

.toc {
    margin: 0 2px 40px 2px;
    border: 1px solid var(--border);
    background: var(--entry);
    border-radius: var(--radius);
    padding: 0.4em;
}

.toc-container.wide {
    position: absolute;
    height: 100%;
    border-right: 1px solid var(--border);
    left: calc((var(--toc-width) + var(--gap)) * -1);
    top: calc(var(--gap) * 2);
    width: var(--toc-width);
}

.wide .toc {
    position: sticky;
    top: var(--gap);
    border: unset;
    background: unset;
    border-radius: unset;
    width: 100%;
    margin: 0 2px 40px 2px;
}

.toc details summary {
    cursor: zoom-in;
    margin-inline-start: 20px;
    padding: 12px 0;
}

.toc details[open] summary {
    font-weight: 500;
}

.toc-container.wide .toc .inner {
    margin: 0;
}

.active {
    font-size: 110%;
    font-weight: 600;
}

.toc ul {
    list-style-type: circle;
}

.toc .inner {
    margin: 0 0 0 20px;
    padding: 0px 15px 15px 20px;
    font-size: 16px;

    /*目录显示高度*/
    max-height: 83vh;
    overflow-y: auto;
}

.toc .inner::-webkit-scrollbar-thumb {  /*滚动条*/
    background: var(--border);
    border: 7px solid var(--theme);
    border-radius: var(--radius);
}

.toc li ul {
    margin-inline-start: calc(var(--gap) * 0.5);
    list-style-type: none;
}

.toc li {
    list-style: none;
    font-size: 0.95rem;
    padding-bottom: 5px;
}

.toc li a:hover {
    color: var(--secondary);
}
```