# Astro 7 + Tailwind CSS v4 Starter

**Astro 7**、**Tailwind CSS v4**、**Lucide icons**、**Biome** を使った、静的サイト・Webアプリ構築のための最小限でオピニオンレイテッドなスターターです。

## スタック

| ツール | バージョン | 役割 |
|---|---|---|
| [Astro](https://astro.build) | ^7 | フレームワーク / SSG |
| [Tailwind CSS](https://tailwindcss.com) | ^4 | スタイリング(Vite プラグイン経由) |
| [@lucide/astro](https://lucide.dev) | ^1 | ツリーシェイク対応の SVG アイコン |
| [Biome](https://biomejs.dev) | 2.x(固定) | Lint + Format |
| TypeScript | ^6 | 型安全性(strict モード) |

## プロジェクト構成

```text
/
├── .github/
│   ├── copilot-instructions.md  # GitHub Copilot 向けガイドライン
│   └── prompts/                 # 再利用可能なプロンプトテンプレート(例: new-page)
├── .vscode/
│   ├── extensions.json          # 推奨拡張機能
│   └── launch.json              # デバッグ設定
├── public/
│   ├── favicon.ico
│   └── favicon.svg
├── src/
│   ├── assets/          # Vite が処理するアセット(import で参照)
│   ├── components/      # 再利用可能な .astro コンポーネント
│   ├── layouts/
│   │   └── Layout.astro # ルートの HTML シェル — グローバル CSS はここで import
│   ├── pages/           # ファイルベースのルーティング(各 .astro が URL になる)
│   └── styles/
│       └── global.css   # @import "tailwindcss" + @theme トークン
├── AGENTS.md            # AI エージェント向けガイドライン(OpenAI Codex / 汎用)
├── CLAUDE.md            # Claude Code 向けガイドライン
├── GEMINI.md            # Gemini CLI 向けガイドライン
├── astro.config.mjs
├── biome.json
└── package.json
```

## コマンド

`<pm>` は使用しているパッケージマネージャー(`npm`、`yarn`、`pnpm` など)に置き換えてください。

```sh
<pm> install           # 依存パッケージをインストール
<pm> run dev           # localhost:4321 で開発サーバーを起動
<pm> run build         # 本番用サイトを ./dist/ にビルド
<pm> run preview       # 本番ビルドをローカルでプレビュー
<pm> run astro check   # .astro ファイルの型チェック
<pm> run lint          # Biome lint --write
<pm> run format        # Biome format --write
<pm> run check         # Biome check --write(lint + format をまとめて実行)
```

## Tailwind v4 の設定

`tailwind.config.js` はありません。テーマのカスタマイズはすべて `src/styles/global.css` に集約されています。

```css
@import "tailwindcss";

@theme {
  --color-brand: #6366f1;
  --font-sans: "Inter", sans-serif;
}
```

プロジェクト固有の色は、生の Tailwind スケールユーティリティを使うのではなく `@theme` トークン(例: `--color-muted`)として定義してください。

## アイコン

[Lucide](https://lucide.dev/icons/) のアイコンは `@lucide/astro` から名前で import して使えます。

```astro
---
import { Camera } from '@lucide/astro'
---

<Camera size={24} class="text-muted" />
```

## コードスタイル

ESLint や Prettier は使用しません。JS/TS/JSON/CSS は **Biome** が管理します。`.astro` ファイルは Biome の対象外なので、型チェックには `<pm> run astro check` を使ってください。

- シングルクォート、セミコロンは `asNeeded`、末尾カンマあり。JSX の属性はダブルクォート
- 80 文字の行幅、2 スペースインデント、LF 改行(`.editorconfig`)
- コード変更を完了する前に、必ず `<pm> run astro check` と `<pm> run check` の両方を実行する

## セキュリティ

`.npmrc` で `min-release-age=1` を設定し、公開から 1 日未満のパッケージのインストールをブロックしています。
`pnpm-workspace.yaml` では `minimumReleaseAge: 1440` を設定し、公開から 24 時間未満のパッケージをブロックしています。これにより悪意のあるパッケージの誤インストールを防ぎます。

ビルドスクリプトの実行権限は `pnpm-workspace.yaml` の `allowBuilds` と `package.json` の `allowScripts` で管理されています。明示的にリストされたパッケージ(例: `esbuild`、`sharp`、`fsevents`)のみがインストールスクリプトを実行できます。

## Astro v7 の主な変更点

v6 からアップグレードする場合は、以下の破壊的変更に注意してください。

- **HTML の厳格な検証** — Rust 製コンパイラは不正な HTML を自動修正しなくなりました。閉じタグ忘れはエラーになります。
- **空白の扱い** — デフォルトが JSX スタイル(`compressHTML: 'jsx'`)に変更され、インライン要素間の空白は圧縮されます。必要な箇所には明示的にスペースを追加するか、`astro.config.mjs` で `compressHTML: true` を設定して以前の挙動に戻してください。
- **`src/fetch.ts` は予約済み** — Astro の高度なルーティングで使用されます。このパスにファイルがある場合はリネームし、設定で `fetchFile` を指定してください。
- **Vite 8** — `package.json` に Vite を `^8` に固定する `overrides` エントリが含まれています。
- **Sätteri markdown** — 新しいデフォルトの Markdown プロセッサ(Rust 製)。remark/rehype プラグインを使い続ける場合は `@astrojs/markdown-remark` を再インストールし、明示的に設定してください。
- **バックグラウンド開発サーバー** — `astro dev --background` はターミナルから切り離された状態でサーバーを起動します。`astro dev stop` / `astro dev status` / `astro dev logs` で管理してください。
