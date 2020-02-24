---
title: "Career"
template: "page"
socialImage: "/media/image-1.jpg"
---
![image-1.jpg](/media/image-1.jpg)

### 来歴
- 1社目：人工知能系の会社
    - 上級研究員の支持のもと、最新論文や技術を調査し、それらの技術が人工知能エンジンに適用可能かどうかの簡易なテストも含めた調査報告を行っていた。
- 2社目：SI会社
    - 社内の業務効率化のための様々な技術検証を行っていました。テキストマイニング、Elasticsearchを用いた検索システムの検証、GoogleAnalyticsを使ったWeb解析、Node.js製のチャットボットフレームワークの導入検討など。
- 3社目：気象会社
    - 防災チャットボットプロジェクト：アプリケーションエンジニア
        - チャットボットの主要部分はAPI Gateway/SQS/Lambda/DynamoDBの構成で開発
        - チャットボットからユーザーへのPush通知機能
            - バックエンドはAPI Gateway/VPC/Lambda/Aurora PostgreSQL(serverless Auroraへの移行を検討)の構成で開発。
            - フロントエンド部分はNuxt.jsで開発、現在はReactへの移行を検討中
        - ソースコードはGitHubで管理し、CodePipeline/ECR/GitHub/CodeBuildの構成でデプロイを行います。
        - 開発チームではSlackを使ったコミュニケーション、Jira/confluenceを使用したプロジェクト管理で動いています。