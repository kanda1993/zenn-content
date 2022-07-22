---
title: "GitHubActionsを使って、Readme.mdからdocxに変換"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHubActions","pandoc","readme"]
published: true
---

# 目的
GitHub上で職務経歴としてREADME.mdを公開している。
転職サイト等にアップロードする場合に、ファイル化する必要がある。
更新するごとに手動でドキュメント化するのが煩わしいのを解決したい。

# 使用技術
  * GitHubActions
  * [pandoc]("https://github.com/jgm/pandoc") : README.mdから変換するツール
  * Docker

# 実施方法
1. GitHubリポジトリにREADME.mdを作成(変換対象)
2. GitHubActionsを開き`new workflow`を選択
3. Simple workflowを`Configure`する
4. 下記コードをコピーし保存する。<br>
   完了
```
name: Crete Doc

on: push
jobs:
  convert_via_pandoc:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
      - run: |
          mkdir output
      - uses: docker://pandoc/core:2.9
        with:
          args: README.md -s -o output/resume.docx
      - uses: actions/upload-artifact@master
        with:
          name: resume
          path: output/resume.docx
```

# 出力結果の取得方法

1. Actionsを選択
　 実行済みの結果を表示
!["GitHubActionsから取得する際のフロー1"](/images/articles/create-resume/1.png)

2. Artifactsに作成されたファイルがある為、DLできます。
!["GitHubActionsから取得する際のフロー2"](/images/articles/create-resume/2.png)

# 解説(初心者向け)
`runs-on: ubuntu-18.04`   : GitHubActionの内容を動かす環境
`- uses: actions/checkout@v2` : 該当リポジトリのソースを、runs-on環境にcheckoutする
`- uses: docker://pandoc/core:2.9` : 
dockerコンテナを起動
→ docker imageはpandocが公式用意したもの
`args: README.md -s -o output/resume.docx` : 変換対象　変換先/変換後ファイル名.形式
`- uses: actions/upload-artifact@master` : GitHubActionsのartifactとして指定ファイルをアップロードする。

# よく分からなかったけれど、実施したい方
1. 下記リポジトリをclone
https://github.com/osusyami/resume
2. README.mdをご自身の経歴に置き換えてください。

# 現状の課題
pdfが作成不可
原因：pandoc公式が用意しているdocker imageに「pdflatex」が入っていない
→ 自分でDocker imageを作成すれば解決すると思われる。