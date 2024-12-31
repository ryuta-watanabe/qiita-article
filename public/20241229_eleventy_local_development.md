---
title: Eleventy で静的ファイルをビルドする環境を作成する（前編）
tags:
  - Nunjucks
  - eleventy
private: false
updated_at: '2024-12-31T13:09:49+09:00'
id: 87f62de8a6d6c11640fd
organization_url_name: null
slide: false
ignorePublish: false
---

この記事は、[Progate Path コミュニティ Advent Calendar 2024](https://qiita.com/advent-calendar/2024/progate-path) の 13 日目の記事です。

前編では、 `Eleventy` を使って HTML をビルドする環境をハンズオン形式で作成します。途中で出てくるコマンドなどは、 Mac を前提としているため、 Windows を使っている場合は適宜読み替えてください。

## Eleventy を使ってローカル開発環境を作成する

まずは、 `Eleventy` を使って静的ファイルを生成する環境を用意します。ここでは以下の流れで進めていきます。

1. `Eleventy` とは
2. `Eleventy` のインストール方法
3. `Eleventy` の設定

もしすぐに環境を利用したい場合は、 [こちら](https://github.com/ryuta-watanabe/eleventy_local_development) にリポジトリを用意したので利用してください。

### Eleventy とは

`Eleventy` は、シンプルで非常に柔軟な静的サイトジェネレーターです。 [公式サイト](https://www.11ty.dev/) によると、ビルドが他のジェネレーターツールよりも高速であることが示されており、また特定の JavaScript フレームワークに依存することなく使用できるため、単純な HTML / CSS / JavaScript で構成されたリソースを生成するのに向いています。

一つだけ依存しているものを挙げると、 `Node.js` のバージョンが `18` 以上である必要があります。もしターミナルで以下のコマンドを実行して、バージョン番号が `18` 未満である場合、先に `Node.js` のバージョンアップをしてください。

```terminal
$ node -v
v20.11.0 # 使用しているバージョンにより出力が変わります
```

### Eleventy のインストール方法

`Eleventy` のインストールは、公式サイトの [ドキュメント](https://www.11ty.dev/docs/) ページに記載のある手順で進めることが出来ますが、以下に簡単な説明を添えて対応の流れを示します。

#### 1. 作業用のフォルダを作成する

まずは、 `Eleventy` を導入して作業をするためのフォルダを作成しましょう。ターミナルで以下のコマンドを実行してください。 `path/to/project-name` については、環境や好みに応じて自由に変更してください。

```terminal
$ mkdir /path/to/project-name && cd /path/to/project-name
# 例: mkdir ~/project/eleventy-sample && cd ~/project/eleventy-sample
```

また、作業用のフォルダが作成できたら、 [VSCode](https://code.visualstudio.com/) で該当のフォルダを開いておきましょう。ターミナルで以下のコマンドを実行してください。

```terminal
$ code .
# . は、カレントディレクトリを参照することを意味しています。
```

#### 2. Eleventy をインストールする

次に、 フォルダを `Node.js` のプロジェクトとして初期化した後、 `Eleventy` をインストールします。ターミナルで以下のコマンドを実行してください。

```terminal
$ npm init -y
$ npm install @11ty/eleventy
$ npm install --save-dev rimraf
```

フォルダ内に、 `package.json` ファイルと、 `node_modules` フォルダが生成されていれば成功しています。

また、 npm scripts についても `Eleventy` を実行するための内容に変更しておきましょう。 `package.json` の `scripts` プロパティを以下のように変更してください。

```json
{
  "scripts": {
    "clean": "rimraf dist",
    "build": "npm run clean && NODE_ENV=production eleventy",
    "build:dev": "npm run clean && NODE_ENV=development eleventy",
    "watch": "eleventy --watch",
    "start": "NODE_ENV=development eleventy --serve"
  }
}
```

### 3. Eleventy の設定

`Eleventy` の設定は、 `eleventy.config.js` ファイルで管理します。インストールを実行してもこのファイルは生成されないため、ターミナルで以下のコマンドを実行して作成してください。

```terminal
$ touch eleventy.config.js
```

最初に設定するのは、入力や出力などの必要最低限の設定です。 `eleventy.config.js` に以下の内容を記述してください。

```js
module.exports = async (eleventyConfig) => {
  return {
    dir: {
      input: "src",
      output: "dist",
      includes: "_includes",
      layouts: "_layouts",
      data: "_data",
    },
  };
};
```

`dir` オブジェクト内にある各プロパティについて、以下に簡単な説明を記載します。

- `input`
  - 入力（バンドル元）フォルダを指定できます。 `src` は、単一のフォルダ名を指定していますが、 glob を使うことも可能です
- `output`
  - 出力（バンドル先）フォルダを指定できます。こちらも `input` プロパティと同様に glob を使うことが可能です
- `includes`
  - テンプレートファイルに読み込むためのファイルが格納されるフォルダを指定できます。たとえば、ヘッダーやフッターのファイルを分離して `includes` に指定したフォルダに格納することで、他のページで共通パーツとして呼び出せます
- `layouts`
  - インクルードと似ていますが、レイアウトを指定するためのファイルが格納されるフォルダを指定できます。
- `data`
  - テンプレートで使用するデータ用ファイルが格納されるフォルダを指定できます。

## Eleventy を使って静的ファイルを出力する

ここまでで必要最低限の設定が出来たので、 `/src` フォルダに作成したファイルをバンドルして、 `/dist` フォルダに出力できるようにしてみましょう。

### 1. テンプレートファイルを用意する

この記事では、最小限の設定でファイルの出力ができるようにしていきましょう。ターミナルで以下のコマンドを実行してください。

```terminal
$ mkdir -p src/_data && touch src/_data/site.json
$ mkdir src/_includes && touch src/_includes/footer.njk src/_includes/header.njk
$ mkdir src/_layouts && touch src/_layouts/base.njk
$ mkdir src/pages && touch src/pages/index.njk
```

`.njk` という見慣れない拡張子が登場しましたが、これは [Nunjucks](https://mozilla.github.io/nunjucks/) というテンプレートエンジンを使うことを示しています。詳細な説明は公式サイトに委ねますが、 HTML を拡張して、変数やフィルタ、ループなど JavaScript の処理を利用できるようにしたものです。

`Evelenty` では、 `Nunjucks` 以外にも多数のテンプレートエンジンに対応しています。

#### 1-1. site.json にメタデータを定義する

`site.json` には、サイトのメタデータを記述します。今回は以下のようにしてみました。

```json
{
  "name": "サイト名",
  "description": "サイトの説明",
  "url": "https://example.com",
  "keywords": "キーワード1, キーワード2, キーワード3",
  "ogpImage": "https://example.com/ogp.jpg",
  "ogpUrl": "https://example.com"
}
```

#### 1-2. header.njk と footer.njk にヘッダーとフッターの HTML を記述する

`header.njk` には、共通で使い回すためのヘッダーを記述します。

```html
<header>
  <nav>
    <div>
      <a href="/">Your Site</a>
    </div>
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</header>
```

`footer.njk` には、共通で使い回すためのフッターを記述します。

```html
<footer>
  <p>© 2024 {{ site.name }}</p>
</footer>
```

#### 1-3. base.njk にレイアウト用 HTML を記述する

`base.njk` には、ページの骨格となるレイアウト用の HTML を記述します。

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{ title }} | {{ site.name }}</title>
    <meta name="description" content="{{ description }}" />
    <meta name="keywords" content="{{ keywords }}" />
    <meta name="format-detection" content="telephone=no,email=no,address=no" />
    <meta property="og:title" content="{{ site.name }}" />
    <meta property="og:description" content="{{ description }}" />
    <meta property="og:image" content="{{ site.ogpImage }}" />
    <meta property="og:url" content="{{ ogpUrl }}" />
    <meta property="og:type" content="website" />
    <link rel="icon" href="/assets/common/favicon.ico" type="image/x-icon" />
    <link rel="stylesheet" href="/styles/main.css" />
    <script src="/scripts/main.js" />
  </head>
  <body>
    {% include "header.njk" %}
    <main>{{ content | safe }}</main>
    {% include "footer.njk" %}
  </body>
</html>
```

#### 1-4. index.njk にトップページ用の HTML を記述する

`index.njk` には、ページのメタデータとなる [Front Matter](https://frontmatter.codes/docs/markdown) と、HTML を記述します。

```html
---
permalink: /index.html
layout: base.njk
title: ホームページ
description: トップページの説明
keywords: キーワード1, キーワード2, キーワード3
ogpUrl: https://example.com
---

<div class="hero">
  <h1>Welcome to {{ site.name }}</h1>
  <p>{{ description }}</p>
</div>
```

冒頭の `Front Matter` に記述されている `permalink` と `layout` について、以下に簡単な説明を記載します。

- `permalink`
  - 出力ディレクトリ（今回は `dist`）内のどこにファイルを出力するかをカスタマイズできるプロパティです。今回は入力が `pages/index.njk` であるため、ここを記述しないと `dist/pages/index.html` として出力されるため、記述しておきましょう
- `layout`
  - 使用するレイアウトファイルを指定するためのプロパティです

ここまで出来たら、ファイルが出力されるか確認しましょう。ターミナルで以下のコマンドを実行してください。

```terminal
$ npm run build
```

`dist/index.html` ファイルが生成されていれば OK です。

以上で `Eleventy` を使って HTML を出力できる環境が作成できました。 CSS や JavaScript のビルドについては、後編でハンズオンをします。
