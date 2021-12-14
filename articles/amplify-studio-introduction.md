---
title: "AWS Amplify Studio(プレビュー版)を触ってみた 【デザインからReactコンポーネントを自動生成】"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Amplify", "Figma", "AWS", "React"]
published: false
---

# 執筆の経緯

2021年12月3日、社内Slackにて以下の記事が話題になる。

@[card](https://www.publickey1.jp/blog/21/awswebaws_amplify_studioaws_reinvent_2021.html)

めちゃくちゃ便利そう！しかし、発表されたばかりということもありあまり情報が転がっていない...

「なら自分で触って社内に共有しよう！」

ということで使い方の大まかな手順と、触ってみて感じたに良いところ、悪いところをまとめてみました。

# Amplify Studioとは

Amplify Studioは、Figmaで作成されたデザインをReactのコンポーネントコードに自動的に変換してくれる開発環境のこと。

要は、**Figmaでデザイン組めば自動でコードが生成されますよ**というもの。

# Amplify Studioの導入手順

## 1. Amplify Studioのプロジェクトを作成

初めてAmplifyプロジェクトを作成する場合は以下の画面になるので**使用を開始する**を選択。

![](https://storage.googleapis.com/zenn-user-upload/7bcbbe0e6ed1-20211214.png)

プロジェクトの名前を決めたら**Confirm deployment**を選択。(後から変更可能)

![](https://storage.googleapis.com/zenn-user-upload/137088984c42-20211215.png)

数分後セットアップが完了すると以下の画面になるので**Studioを起動する**を選択。

![](https://storage.googleapis.com/zenn-user-upload/b7559b66fc9d-20211215.png)

## 2. FigmaとAmplify Studioを連携

今回はAmplifyがデモ用に用意してくれたデザインを用いて進めます。

:::message
実際に自分のデザインを使う場合FigmaのAuto Layoutという機能を用いてうまくパーツを組み立てないと思った通りにコードが生成されません。（難易度高め）
:::

Amplify Studioを起動後、**UI Library** → **Get started**を選択。

![](https://storage.googleapis.com/zenn-user-upload/e3597fb079d6-20211215.png)

モーダルが出現するので、①の[Use our Figma template](https://www.figma.com/community/file/1047600760128127424)を選択。

![](https://storage.googleapis.com/zenn-user-upload/1846c24ba148-20211215.png)

Figmaに遷移するので右上の**Duplicate**を選択。

![](https://storage.googleapis.com/zenn-user-upload/5e8984968f17-20211215.png)

するとFigmaのプロジェクトが開かれるのでURLをコピー。

![](https://storage.googleapis.com/zenn-user-upload/405cb88115e9-20211215.png)

コピーしてきたFigmaのURLをペースト＆**Continue**を選択。

![](https://storage.googleapis.com/zenn-user-upload/fef3216f8dbe-20211215.png)

Figmaとの連携が終わるとFigmaでコンポーネントとして定義されているパーツが表示され、それぞれAmplify Studioに取り込むかどうかが聞かれる。
今回はひとまず右上の**Accept all**を選択する。

![](https://storage.googleapis.com/zenn-user-upload/0301197451d9-20211215.png)

## 3. データの流し込み

データモデルを作成することでコンポーネントに流し込みたいpropsを定義できます。
今回はサンプルとして画像のpropsを受け渡します。

UI Libraryから好きなコンポーネントを選択 →右上の**Configure**を選択。

![](https://storage.googleapis.com/zenn-user-upload/4dd303734f90-20211215.png)

画像を選択(左カラム) → Propsを設定(右カラム)
設定が完了したら右上の**Create collection**を選択。

![](https://storage.googleapis.com/zenn-user-upload/2687a03a49cf-20211215.png)

ひとまずこのまま保存。
![](https://storage.googleapis.com/zenn-user-upload/bf8c05631c78-20211215.png)

## 4. UI LibraryをReactアプリケーションに取り込む(amplify pull)

まずは`create-react-app`にてReactアプリを用意。

```zsh
npx create-react-app studio-demo
```

:::message
2021年12月15日時点、create-react-app 5.0.0でインストールされるreact-scripts 5.0.0にてnpm startできない問題を確認。
create-react-appのバージョンを下げることで対応できることは確認できたが詳しくは調査中。
:::

Reactアプリが用意できたらAmplify Studio画面右上より**Local setup instructions**を選択。
Amplify CLIをまだインストールしていない場合は**View Amplify CLI installation instructions**に沿ってCLIをインストール。

![](https://storage.googleapis.com/zenn-user-upload/4d68ccfaf18f-20211215.png)

無事amplify pullが成功すると`/src/ui-components`にFigmaから自動生成されたコンポーネントが流れこんでくる。

![](https://storage.googleapis.com/zenn-user-upload/eea28126e37c-20211215.png)

## 5. 自動生成されたコンポーネントを画面に表示する

必要なライブラリのインストール。

```zsh
npm install aws-amplify @aws-amplify/ui-react
```

以下を`/src/index.js`にインポート。

```jsx
import Amplify from 'aws-amplify';
import "@aws-amplify/ui-react/styles.css";
import {AmplifyProvider} from "@aws-amplify/ui-react";
import awsconfig from './aws-exports';
Amplify.configure(awsconfig);
```

STEP3で定義したimageUrl propがHeroLayout1コンポーネントへ渡せるようになっているので画像URLを流し込みます。
好きなコンポーネントを設置すると...

```jsx:src/index.jsx
ReactDOM.render(
  <React.StrictMode>
    <AmplifyProvider>
      <NavBar />
      <HeroLayout1 imageUrl="https://picsum.photos/1440/1128" />
      <FeaturesText2x2 />
      <MarketingPricing />
      <Features2x2 />
      <MarketingFooter />
    </AmplifyProvider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

それっぽいページの完成！
デモは[こちら](https://morookakotaro.github.io/amplify-studio-demo/)

![](https://storage.googleapis.com/zenn-user-upload/735c20072925-20211215.png)

# 触ってみた感想

## 良かった点

## 悪かった点

# まとめ

# 参考

@[card](https://aws.amazon.com/jp/blogs/news/aws-amplify-studio-figma-to-fullstack-react-app-with-minimal-programming/)
