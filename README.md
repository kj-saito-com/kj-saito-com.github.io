📘 サブサイトの追加方法（GitHub Pages + minima テーマ）
このリポジトリでは、1リポジトリで複数の独立したサブサイトを管理できます。
各サブサイトは Jekyll + minima テーマで構築され、GitHub Pages に自動でデプロイされます。

✅ 1. サブサイト用ディレクトリを作成
例：spring_boot_guide

bash
コピーする
編集する
mkdir spring_boot_guide
✅ 2. サブサイトの _config.yml を作成
yaml
コピーする
編集する
# spring_boot_guide/_config.yml
title: Spring Boot Guide
description: Spring Boot の実践ガイド
theme: minima
baseurl: ""
✅ 3. サブサイトの index.md を作成
markdown
コピーする
編集する
---
layout: default
title: Spring Boot Guide TOP
---

# Spring Boot Guideへようこそ！

このサイトではSpring Bootの実用的な使い方を紹介します。
✅ 4. GitHub Actions にビルドステップを追加
.github/workflows/pages.yml の build: ステップに以下を追記：

yaml
コピーする
編集する
- name: Build spring_boot_guide site
  run: |
    bundle exec jekyll build \
      --source spring_boot_guide \
      --destination _site/spring_boot_guide
これにより、spring_boot_guide/ 以下が /spring_boot_guide/ URLで公開されます。

✅ 5. コミット＆Pushすれば完了！
bash
コピーする
編集する
git add .
git commit -m "Add new subsite: spring_boot_guide"
git push origin main
GitHub Actions により自動でビルド → gh-pages ブランチにデプロイされます。

🌐 アクセスURLの例
サブサイト名	URL例
ルートサイト	https://<ユーザー名>.github.io/
Javaカリキュラム	https://<ユーザー名>.github.io/java_sql_spring_curriculum/
Spring Bootガイド	https://<ユーザー名>.github.io/spring_boot_guide/

📦 テーマや構成の再利用について
すべてのサブサイトで minima テーマを利用可能

CSSや共通パーツは個別管理 or 外部共有可

