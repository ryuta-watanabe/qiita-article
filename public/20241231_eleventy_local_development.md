---
title: Eleventy で静的ファイルをビルドする環境を作成する（後編）
tags:
  - postcss
  - Nunjucks
  - prettier
  - eleventy
  - esbuild
private: false
updated_at: '2024-12-31T19:01:46+09:00'
id: 1d737252bd0115759a90
organization_url_name: null
slide: false
ignorePublish: false
---

この記事は、[Progate Path コミュニティ Advent Calendar 2024](https://qiita.com/advent-calendar/2024/progate-path) の 21 日目の記事です。

- [Eleventy で静的ファイルをビルドする環境を作成する（前編）](https://qiita.com/Ryuta_Watanabe/items/87f62de8a6d6c11640fd)

後編では、 `Eleventy` を使って CSS と JavaScript をビルドする環境を用意します。途中で出てくるコマンドなどは、 Mac を前提としているため、 Windows を使っている場合は適宜読み替えてください。

## Eleventy を使って CSS と JavaScript をビルドする

前編で HTML をビルドする環境の用意ができたので、次は CSS と JavaScript をビルドできるように拡張していきましょう。以下の流れで進めていきます。

1. prettier を導入して、出力後の HTML を整形する
2. PostCSS を導入して、 CSS の出力を最適化する
3. esbuild を導入して、 JavaScript をバンドルする

もしすぐに環境を利用したい場合は、 [こちら](https://github.com/ryuta-watanabe/eleventy_local_development) にリポジトリを用意したので利用してください。

### 1. prettier を導入して、出力後の HTML を整形する

前編で出力した HTML ファイルを開くと、インデントが綺麗に揃っていません。細かいことですが、これを `npm run build` 実行時に自動で整形できるようにしましょう。以下の流れで対応を進めます。

1. [prettier](https://prettier.io/) をインストールする
2. eleventy.config.js に HTML 出力後の処理を追記する

#### 1-1. prettier をインストールする

ターミナルで以下のコマンドを実行してください。

```terminal
$ npm install --save-dev prettier
```

`package.json` の `devDependencies` に、 `prettier` が追加されていれば OK です。

#### 1-2. eleventy.config.js に HTML 出力後の処理を追記する

ファイルに以下の変更を加えます。

- ファイルの冒頭で `prettier` を呼び出す
- 末尾にある出力用の `return` 文の上に、 HTML 整形用の処理を追記する

```js:eleventy.config.js
// ファイル冒頭で prettier を呼び出す
const prettier = require("prettier");

// HTMLの変換後に実行される処理を追記する
module.exports = async (eleventyConfig) => {
  eleventyConfig.addTransform("prettier", async (content, outputPath) => {
    if (outputPath?.endsWith(".html")) {
      return prettier.format(content, {
        parser: "html",
        printWidth: 120,
        tabWidth: 2,
        useTabs: false,
      });
    }
    return content;
  });
  // （中略）
};
```

出力ファイルに変更を加えるために、 [addTransform](https://www.11ty.dev/docs/transforms/) メソッドを使用します。

`addTransform` は、第一引数に処理の名前を定義するための文字列を設定し、コールバック関数に処理を記述します。 `prettier` 処理では、拡張子が `.html` だったときに、 `prettier` によるフォーマットを実行し、フォーマット後のコンテンツを返しています。

ここまで出来たら、出力後のファイルが整形されるか確認してみましょう。ターミナルで以下のコマンドを実行してください。

```terminal
$ npm run build
```

### 2. PostCSS を導入して、 CSS の出力を最適化する

CSS のトランスパイラとして、今回は [PostCSS](https://postcss.org/) を導入します。また、いくつかの機能を追加したいので、以下のパッケージを導入します。

- [autoprefixer](https://github.com/postcss/autoprefixer)
  - ベンダープレフィックスの自動付与する
- [postcss-import](https://github.com/postcss/postcss-import)
  - `@import` を使ってファイルの分割管理を実現する
- [cssnano](https://cssnano.github.io/cssnano/)
  - 出力後のファイルを minify 化する

以下の流れで対応を進めていきます。

1. 必要なパッケージをインストールする
2. eleventy.config.js に HTML 出力後の処理を追記する

#### 2-1. 必要なパッケージをインストールする

ターミナルで以下のコマンドを実行してください。

```terminal
$ npm install --save-dev postcss autoprefixer postcss-import cssnano
```

#### 2-2. eleventy.config.js に HTML 出力後の処理を追記する

ファイルに以下の変更を加えます。

- ファイルの冒頭で、 `node:path` / `autoprefixer` / `postcss-import` / `cssnano` を呼び出す
- `prettier` の処理よりも上に、 PostCSS の処理を追記する

```js:eleventy.config.js
// ファイルの冒頭で必要なパッケージを呼び出す
const path = require("node:path");
const postcss = require("postcss");
const autoprefixer = require("autoprefixer");
const postcssImport = require("postcss-import");
const cssnano = require("cssnano");

// CSSのトランスパイル処理を追記する
module.exports = async (eleventyConfig) => {
  eleventyConfig.addTemplateFormats("css");
  eleventyConfig.addExtension("css", {
    outputFileExtension: "css",
    compile: async (inputContent, inputPath) => {
      // _から始まるファイルは無視
      if (path.basename(inputPath).startsWith("_")) {
        return;
      }

      // PostCSSの処理
      const result = await postcss([
        postcssImport, // @importの解決
        autoprefixer, // ベンダープレフィックスの自動付与
        cssnano({
          // CSSの最適化（本番環境用）
          preset: "default",
        }),
      ]).process(inputContent, {
        from: inputPath,
        to: inputPath.replace(/^src/, "dist"),
      });

      return async () => {
        return result.css;
      };
    },
  });
  // （中略）
};
```

[addTemplateFormats](https://www.11ty.dev/docs/config/#template-formats) は `Eleventy` に CSS ファイルを処理対象として認識させるための設定で、 `addExtension` はその拡張子に対する処理を記述するためのメソッドです。（公式ドキュメントでこのメソッドの端的な説明が見つけられず、サンプルコードと説明からこうだろう、という推測になります）

`PostCSS` の処理を `addExtension` の中で行い、その結果を `return` することで、 `@import` やベンダープレフィックスが解決された状態で、かつ minify された CSS を出力するよう指示しています。

ここまで出来たら、動作確認のために CSS ファイルを作成し、ビルドコマンドを実行してみましょう。 以下の CSS ファイルを用意してください。

```css:src/styles/components/_header.css
header {
  background: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

  nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    max-width: 1200px;
    margin: 0 auto;
    padding: 1rem;
  }

  ul {
    display: flex;
    gap: 2rem;
    list-style: none;
    margin: 0;
    padding: 0;
  }

  a {
    color: #333;
    text-decoration: none;
    &:hover {
      color: #666;
    }
  }
}
```

```css:src/styles/main.css
/* リポジトリの main.css から一部を抜粋しています */
@import "./base/_reset.css";
@import "./components/_header.css";

:root {
  /* コンテンツ幅 */
  --content-width-sm: 640px;
  --content-width-md: 768px;
  --content-width-lg: 1024px;
  --content-width-xl: 1200px;
  --content-width-2xl: 1536px;

  /* 背景色 */
  --color-background: #ffffff;
  --color-background-alt: #f3f4f6;
  --color-background-dark: #e5e7eb;

  /* スペーシング */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
}

/* ベース */
html {
  font-family: system-ui, sans-serif;
  background-color: var(--color-background);
}

body {
  margin: 0;
  padding: 0;
}

main {
  margin: auto;
  padding: var(--spacing-md);
  width: 100%;
  max-width: var(--content-width-xl);
}
```

ファイルの作成後、ターミナルで以下のコマンドを実行してください。

```terminal
$ npm run build
```

`dist/styles/main.css` が生成されていれば OK です。

### 3. esbuild を導入して、 JavaScript をバンドルする

JavaScript のバンドルツールとして、今回は [esbuild](https://esbuild.github.io/) を導入します。以下の流れで進めていきます。

1. esbuild をインストールする
2. eleventy.config.js に JavaScript の処理を追記する

#### 3-1. esbuild をインストールする

ターミナルで以下のコマンドを実行してください。

```terminal
$ npm install --save-dev esbuild
```

`package.json` の `devDependencies` に `esbuild` が追加されていれば OK です。

#### 3-2. eleventy.config.js に JavaScript の処理を追記する

ファイルに以下の変更を加えます。

- ファイルの冒頭で `esbuild` を呼び出す
- CSS の処理の下くらいに、 JavaScript の処理を追記する

```js:eleventy.config.js
// ファイルの冒頭で esbuild を呼び出す
const esbuild = require("esbuild");

// JavaScript のバンドル処理を追記する
module.exports = async (eleventyConfig) => {
  // （中略）
  eleventyConfig.on("eleventy.before", async () => {
    const isProduction = process.env.NODE_ENV === "production"; // 環境変数で判定
    const jsFiles = [
      "src/scripts/main.js", // 他のファイルをここに追加可能
    ];

    for (const inputFile of jsFiles) {
      // modules 配下のファイルはスキップ
      if (inputFile.includes("/modules/")) {
        continue;
      }

      const outputFile = inputFile.replace(/^src\/scripts\//, "dist/scripts/");

      // esbuild バンドル
      await esbuild.build({
        entryPoints: [inputFile], // バンドル対象のファイル
        outfile: outputFile, // 出力ファイル名を同一に
        bundle: true, // バンドル有効
        minify: isProduction, // 本番環境では圧縮
        sourcemap: !isProduction, // ソースマップの出力を環境変数で制御
        format: "iife", // フォーマットは即時関数
        platform: "browser", // ブラウザ用設定
        mainFields: ["browser", "module", "main"], // モジュール解決
      });
    }
  });
  // （中略）
};
```

[eleventy.before](https://www.11ty.dev/docs/events/#eleventy.before) イベントは、 `Eleventy` によるテンプレートの処理が始まる前に、独自の処理を差し込むことができるオプションです。テンプレートの処理が始まる前に、 JavaScript のバンドル処理は完了したいので、今回はこのイベントを使用してみました。

一旦、 `eleventy.config.js` の全体像をまとめてみました。もしここまでの対応で不安があれば、参考にしてみてください。

<details>
<summary>eleventy.config.js</summary>

```js:eleventy.config.js
const path = require("node:path");
const postcss = require("postcss");
const autoprefixer = require("autoprefixer");
const postcssImport = require("postcss-import");
const cssnano = require("cssnano");
const esbuild = require("esbuild");
const prettier = require("prettier");

module.exports = async (eleventyConfig) => {
  // CSSのビルド
  eleventyConfig.addTemplateFormats("css");
  eleventyConfig.addExtension("css", {
    outputFileExtension: "css",
    compile: async (inputContent, inputPath) => {
      // _から始まるファイルは無視
      if (path.basename(inputPath).startsWith("_")) {
        return;
      }

      // PostCSSの処理
      const result = await postcss([
        postcssImport, // @importの解決
        autoprefixer, // ベンダープレフィックスの自動付与
        cssnano({
          // CSSの最適化（本番環境用）
          preset: "default",
        }),
      ]).process(inputContent, {
        from: inputPath,
        to: inputPath.replace(/^src/, "dist"),
      });

      return async () => {
        return result.css;
      };
    },
  });

  // JSファイルのバンドル
  eleventyConfig.on("eleventy.before", async () => {
    const isProduction = process.env.NODE_ENV === "production"; // 環境変数で判定
    const jsFiles = [
      "src/scripts/main.js", // 他のファイルをここに追加可能
    ];

    for (const inputFile of jsFiles) {
      // modules 配下のファイルはスキップ
      if (inputFile.includes("/modules/")) {
        continue;
      }

      const outputFile = inputFile.replace(/^src\/scripts\//, "dist/scripts/");

      // esbuild バンドル
      await esbuild.build({
        entryPoints: [inputFile], // バンドル対象のファイル
        outfile: outputFile, // 出力ファイル名を同一に
        bundle: true, // バンドル有効
        minify: isProduction, // 本番環境では圧縮
        sourcemap: !isProduction, // ソースマップの出力を環境変数で制御
        format: "iife", // フォーマットは即時関数
        platform: "browser", // ブラウザ用設定
        mainFields: ["browser", "module", "main"], // モジュール解決
      });
    }
  });

  // HTMLの変換後に実行される処理
  eleventyConfig.addTransform("prettier", async (content, outputPath) => {
    if (outputPath?.endsWith(".html")) {
      return prettier.format(content, {
        parser: "html",
        printWidth: 120,
        tabWidth: 2,
        useTabs: false,
      });
    }
    return content;
  });

  // 出力設定
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

</details>

ここまで出来たら、動作確認のために JS ファイルを作成し、ビルドコマンドを実行してみましょう。以下の JS ファイルを用意してください。

```js:src/scripts/modules/greet.js
export function greet(name) {
  console.log(`Hello, ${name}!`);
}
```

```js:src/scripts/main.js
import { greet } from "./modules/greet.js";

greet("Eleventy + esbuild");
```

ファイルの作成後、ターミナルで以下のコマンドを実行してください。

```terminal
$ npm run build
```

`dist/scripts/main.js` が生成されていれば OK です。

## まとめ

前編、後編までの対応で、 `Eleventy` を使って HTML / CSS / JS ファイルをビルドする環境を作成できました。地味ですが、複雑な機能を持たないような LP の制作だったり、バックエンドの開発は別会社で…というようなケースでは、そこそこ利用できるのかなと思っています。

余力があったら、 Github Actions を使って Xserver Static にアップロードし、認証をかけて一時的なチェック用の環境を作成する手順も書いてみたいと思います。
