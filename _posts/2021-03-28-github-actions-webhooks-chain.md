---
layout: post
title: "GitHub Actions 和 Webhooks 的触发"
comments: true
date: 2021-03-28
description: ""
categories: [Gimmick]
tags: [github, git, actions, webhooks]
---

**目录**
* ToC
{:toc}
---

## 需求一：由 Action 触发 Webhook

* 使用 `distributhor/workflow-webhook`
* 地址：https://github.com/distributhor/workflow-webhook

示例：

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Invoke deployment hook
        uses: distributhor/workflow-webhook@v1
        env:
          webhook_type: 'json-extended'
          webhook_url: ${{ secrets.WEBHOOK_URL }}
          webhook_secret: ${{ secrets.WEBHOOK_SECRET }}
          data: '{ "weapon": "hammer", "drink" : "beer" }'
```

* `WEBHOOK_URL` 和 `WEBHOOK_SECRET` 分别为 Webhook 地址和密码，均在仓库的 `secrets` 中设置。

## 需求二：由仓库 A 的 Action 触发仓库 B 的 Action

### 仓库 B 的设置

* 在仓库 B 的 `.github/workflows/Action.yml` 中添加一个触发条件 `repository_dispatch`

```yml
on:
  repository_dispatch:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * *"
```

### 仓库 A 的设置

* 在仓库 A 的 `.github/workflows/Action.yml` 添加以下片段

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger auto update actions
          run: |
            curl -XPOST -u "${{secrets.PAT_USERNAME}}:${{secrets.PAT_TOKEN}}" -H "Accept: application/vnd.github.everest-preview+json" -H "Content-Type: application/json" ${{secrets.PAT_ADDR}} --data '{"event_type": "build_application"}'
```

* `PAT_USERNAME` 为仓库 B 用户的用户名；
* `PAT_TOKEN` 为仓库 B 用户的 Personal Access Token；
* `PAT_ADDR` 形如 `https://api.github.com/repos/xxx/yyyy/dispatches`，`xxx` 为仓库 B 用户名，`yyyy` 为仓库 B 的名称；
* 以上变量在仓库 A 的 `secrets` 中设置。

### 引用
1. https://github.community/t/triggering-actions-by-other-repository-webhooks/16295/5
2. https://github.community/t/triggering-by-other-repository/16163