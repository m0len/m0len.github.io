---
layout: post
title: "开篇"
comments: true
date: 2019-02-24
categories: [Blog]
tags: [jekyll, github, page, domain, cname, seo]
---

**目录**
* ToC
{:toc}
---

> 这是一本记录折腾的笔记本，托管于[ GitHub ](https://github.com/m0len/m0len.github.io)。

## 关于搭建

1. 在这里找一个自己喜欢的模板：http://jekyllthemes.org/
2. Fork 到自己的 GitHub 下
3. 仓库名改成`$USERNAME.github.io`，这就是你未来博客的地址。例如： `m0len.github.io` 
4. 按需修改配置文件 `_config.yml` 
5. 等待

一段时间之后 GitHub 会自动帮你生成博客页面，访问刚刚修改的 `xxx.github.io` 就可以看到。

## 关于 Branch

你可能在网上的教程中见过一些需要使用 `gh-pages` 分支才能生成页面的要求，实测并不是，以下方案都可以：

* 直接使用 `master` 分支
* 使用 `gh-pages` 分支
* 在 `master` 分支里用一个名叫 `docs` 的文件夹
* 使用任何分支然后在 `settings` 里面告诉GitHub

## 关于自定义域名

直接在该仓库里添加一个名叫 `CNAME` 的文件，里面填入你要使用的自定义域名，然后在域名服务商添加一个 `CNAME` 记录指向 `xxx.github.io` 等待生效即可。

## 参考

* [Jekyll](https://jekyllrb.com/)
* [Jekyll GitHub](https://github.com/jekyll/jekyll)
* [Configuring a publishing source for GitHub Pages](https://help.github.com/en/articles/configuring-a-publishing-source-for-github-pages)
* [User, Organization, and Project Pages](https://help.github.com/en/articles/user-organization-and-project-pages)
