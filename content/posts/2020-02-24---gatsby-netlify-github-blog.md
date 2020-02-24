---
title: Gatsby.js + Netlify + GitHubを使ってブログを公開する
date: "2020-02-24T19:30:00.000+09:00"
template: "post"
draft: false
slag: "gatsby-netlify-github-blog"
category: "JS"
tags:
    - "Tech"
    - "Gatsby.js"
    - "JS"
description: "Gatsbyを使って簡単に個人ブログを公開しようという記事です。Gatsbyには多くのブログテンプレートが用意されているので、初めての人はそこから始められるのが嬉しいですね！"
---

Gatsbyを使って簡単に個人ブログを公開しようという記事です。Gatsbyには多くのブログテンプレートが用意されているので、初めての人はそこから始められるのが嬉しいですね！

## あっという間にブログが公開できちゃった
[こちらのQiita記事](https://qiita.com/Zabit/items/fdff7b036ce6231ff8db)を参考にさせて頂きました！記事はmarkdown形式で書けるので私にとってはとても馴染みやすいものです！これから頑張って記事をアップしていこうと思います！

## 手順
大まかな手順は以下のようになります。
1. NetlifyにGatsbyのプロジェクトをデプロイします。
    - NetlifyではGitHubとの連携でプロジェクト管理とデプロイができるので、GitHubアカウント連携とリポジトリの作成を行います。
2. GitHubよりプロジェクトをクローンしてローカル環境を構築
    - デプロイが完了するとGitHub上にリポジトリが作成されますので、ローカル環境にクローンしてブログ開発環境を準備します。
3. プロフィール変更や記事の投稿手順
    - ローカル環境でソースコードを修正し、自分用のブログの雛形ができあがったらコミットして、GitHubにpushすれば後は勝手にNetlifyに変更内容がデプロイされます！簡単だね！

と言っても手順1と2についてはQiita記事の内容をそのまま実行しただけなので割愛します。ここでは手順3でQiitaに書いていない修正を行うことで、どのようにして[こちらのブログ](https://jackkitte-blog.netlify.com/)のようになるかを説明しようかと思います！

### レイアウトサイズの変更
ページの横幅を広くしたかったので、*src/assets/scss/_variables.scss*を以下のように修正することで画像のように広くすることができます。
```
// Layout
// $layout-post-single-width: 945px;
$layout-post-single-width: 1150px; // 修正部分
$layout-post-width: $layout-post-single-width - 305px;

// $layout-width: 1070px;
$layout-width: 1270px; // 修正部分
$layout-breakpoint-sm: 685px;
$layout-breakpoint-md: 960px;
$layout-breakpoint-lg: 1100px;
```
**トップページ修正前**
![修正前のトップページ](/media/origin_top.png)
**トップページ修正後**
![修正後のトップページ](/media/update_top.png)
**投稿ページ修正前**
![修正前のトップページ](/media/origin_post.png)
**投稿ページ修正後**
![修正後のトップページ](/media/update_post.png)

### プロフィール/キャリアページの変更

About meのページは*content/pages/about.md*を修正すれば直ぐに反映されます。
```
---
title: "About me"
template: "page"
socialImage: "/media/image-2.jpg"
---
![image-2.jpg](/media/image-2.jpg)

- 出身：沖縄県
- 英語は全然喋れません
- 最終学歴：琉球大学大学院　理工学研究科　情報工学専攻
    - 大学で情報工学を学び、研究では當間研究室にてニューラルネットワークをやっておりました。ディープラーニングにおけるパラメータ調整を自動で最適な値に設定できないかどうかの検証を行いました。
~~ 省略 ~~
```
**プロフィールページ**
![修正後のabout me](/media/about.png)

キャリアページについては、まず*config.js*を以下のように変更します。
```
  menu: [
    {
      label: 'Articles',
      path: '/'
    },
    {
      label: 'About me',
      path: '/pages/about'
    },
    {
      label: 'Career',          // 'Contact me'から名前を変更
      path: '/pages/career'     // '/pages/contacts'から名前を変更
    }
  ],
```
次に、*content/pages/career.md*ファイルを新規に作成し以下のようにします。すると、画像のようにキャリアページが作れます。
```
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
~~ 省略 ~~
```
**キャリアページ**
![修正後のcareer](/media/career.png)
