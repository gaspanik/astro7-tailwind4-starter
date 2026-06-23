---
name: figma-workflow
description: >-
  Figma URL を受け取り、実装 → レビュー → 最適化 → Tailwindレビュー の4フェーズをサブエージェントで逐次実行するワークフロースキル（Astro向け）
argument-hint: "<figma-url>"
allowed-tools: Agent
---

# Figma → Code ワークフロー（Astro）

Figma URL を受き取り、**実装 → レビュー → 最適化 → Tailwindレビュー** の4フェーズをサブエージェントで逐次実行するワークフロースキルです。

## 引数

`$ARGUMENTS` に Figma URL が渡されます。

## 注意：大きなページの場合

縦に長いランディングページなど大きなフレームを1度に渡すとコンテキストが肥大化します。
その場合は **セクション単位のフレーム**（Hero, Features, CTA など）に分けて、このワークフローを複数回実行してください。

## 実行ルール

- 各フェーズは **Agent ツール**でサブエージェントとして起動する
- 必ず **Phase 1 → Phase 2 → Phase 3 → Phase 4** の順に実行する（並列不可）
- 各エージェントの完了を確認してから次を起動する
- 全フェーズ完了後、各エージェントの報告内容をまとめてユーザーに提示する

---

## Phase 1: 実装

Agent ツールで以下のプロンプトを渡してサブエージェントを起動する。

```
以下のFigma URLのデザインを実装してください：$ARGUMENTS

## 手順

### 0. 実装先ファイルの確認

`$ARGUMENTS` にファイル名（例: `about.astro`）が含まれていない場合、実装を開始する前にユーザーに確認する：

> 実装先のファイル名を教えてください（例: `src/pages/index.astro`, `src/components/Hero.astro`）

- `src/pages/` への実装（ページ）か `src/components/` への実装（コンポーネント）かも確認する
- 既存ファイルへの追加か、新規ファイルの作成かも確認する

### 1. デザイン情報の取得
- `get_design_context` でデザインコンテキストとリファレンスコードを取得する
- `get_variable_defs` でデザイントークン（色、スペーシング等）を取得する
- ファイル中のアノテーションの内容を確認し、実装に必要な情報を把握する

### 2. 既存コードの確認
- `src/components/` 内の既存コンポーネントを確認し、再利用できるものがないか調べる
- `src/styles/global.css` の `@theme` ブロックで定義済みのトークンを確認する

### 3. 実装
CLAUDE.md のガイドラインに厳密に従って実装すること：

**Astro コンポーネント構造**
- 新規コンポーネントは `src/components/` に配置（ファイル名はPascalCase、例: `HeroSection.astro`）
- 新規ページは `src/pages/` に配置
- `.astro` ファイルはフロントマター（`---`）でインポートとロジックを記述し、テンプレートにHTMLを書く

**スタイリング**
- Tailwind CSS v4 構文を使用（`space-x/y-*` は使わず `gap-*` + flex/grid を使う）
- クラスは `class` 属性に直接記述する（JSXの `className` は使わない）
- グローバルスタイルの追加は `src/styles/global.css` に行う

**アクセシビリティ**
- セマンティックなHTMLタグを使用（`header`, `main`, `section`, `nav` 等）
- `aria-label`, `aria-expanded` 等の適切なARIA属性を付与
- 画像には意味のある `alt` テキストを設定

**デザイントークン**
- Figmaから取得したトークンが `@theme` に未定義の場合、`src/styles/global.css` に追加する
- 任意値（`w-[42px]`）は最終手段として使用し、複数箇所で使う場合は `@theme` に追加する

**アイコン**
- `@lucide/astro` のアイコンをAstroコンポーネントとしてインポートして使用する
  ```astro
  ---
  import { Camera } from '@lucide/astro';
  ---
  <Camera size={16} stroke-width={1} class="text-muted" />
  ```
- Props: `size`（デフォルト `24`）、`color`（デフォルト `currentColor`）、`stroke-width`（デフォルト `2`）

**画像アセット**
- Figma MCP の画像URL（`https://www.figma.com/api/mcp/asset/...`）は **7日間で失効** するため、必ずローカルに保存する
- `src/assets/images/` に `webp` 形式でダウンロードし、内容を表す kebab-case で命名する
  ```bash
  curl -s -o src/assets/images/<name>.webp "<figma-asset-url>"
  ```
- コード内ではViteの `import` で参照する（直接URLは埋め込まない）：
  ```astro
  ---
  import heroImage from '../assets/images/hero.webp';
  ---
  <img src={heroImage.src} alt="..." />
  ```

### 4. 完了確認
実装完了後、以下を確認してメッセージに含める：
- 作成・変更したファイルの一覧
- 新たに `@theme` に追加したトークン（あれば）
- 既存コンポーネントの再利用状況
- ダウンロードした画像ファイルの一覧（`src/assets/images/` に保存したもの）
- アノテーションで指定された要件をすべて満たしているか
```

---

## Phase 2: レビューと修正

Phase 1 完了後、Agent ツールで以下のプロンプトを渡してサブエージェントを起動する。

```
以下のFigma URLのデザインと実装を比較してレビューしてください：$ARGUMENTS

## 手順

### 1. Figmaのスクリーンショット取得
- `get_screenshot` で対象ノードのスクリーンショットを取得する
- スクリーンショットを参照しながら実装との視覚的差異を特定する

### 2. 視覚的差異の確認と修正
スクリーンショットと実装を比較して以下を確認・修正する：

**レイアウト**
- 要素の配置・整列（flex/grid の方向、justify/align の設定）
- スペーシング（padding, margin, gap の値）
- サイズ（幅・高さ）

**スタイル**
- 色（`@theme` トークンまたはTailwindのカラーパレット）
- タイポグラフィ（フォントサイズ、ウェイト、行間）
- 角丸（border-radius）、シャドウ（box-shadow）
- ボーダー（色・太さ・スタイル）

**レスポンシブ**
- モバイル（`sm:`）・タブレット（`md:`）・デスクトップ（`lg:`）での見た目

### 3. 型チェックとエラーの解消
```bash
<pm> run astro check
```
を実行して `.astro` ファイルの型エラーをすべて解消する。

### 4. コードスタイルの確認
```bash
<pm> run check
```
を実行してBiomeのリント・フォーマットエラーを自動修正する（`.astro` ファイルはBiome対象外）。

### 5. アクセシビリティの確認
- キーボードナビゲーションが機能するか
- `aria-label` 等のARIA属性が適切に設定されているか
- 色コントラストがWCAG AA基準（4.5:1）を満たしているか
- 画像に意味のある `alt` テキストがあるか

### 6. レビュー結果の報告
修正完了後、以下をまとめて報告する：
- 視覚的差異として修正した箇所
- 型エラー・コードスタイルの修正内容（あれば）
- アクセシビリティの改善点（あれば）
- 残課題があれば明記する
```

---

## Phase 3: 最適化

Phase 2 完了後、Agent ツールで以下のプロンプトを渡してサブエージェントを起動する。

```
`src/components/` 内の実装済みコンポーネントをリファクタリングして、再利用性と保守性を高めてください。

## 手順

### 1. 現状の把握
`src/components/` および `src/pages/` 内のファイルをすべて確認して以下を特定する：
- 重複しているマークアップやスタイル
- ハードコードされた値（色、サイズ等）
- コンポーネントとして切り出せるまとまり

### 2. コンポーネントの整理
- 同じマークアップが2箇所以上で使われていたらコンポーネント化を検討する
- ページから適切な粒度のコンポーネントを切り出し `src/components/` に配置する
- Astro の `<slot />` を活用して柔軟なコンポーネントを設計する
- ただし過度な抽象化は避ける（DRYより明快さを優先）

### 3. デザイントークンの整理
- ハードコードされた色・スペーシング値を `src/styles/global.css` の `@theme` ブロックに移動する
- 任意値（`w-[42px]`）を複数箇所で使っている場合はトークン化する
- トークン名はセマンティックに命名する（`--color-brand-primary` 等）

### 4. コードスタイルの最終確認
```bash
<pm> run astro check && <pm> run check
```
を実行してエラーがないことを確認する。

### 5. 最適化結果の報告
以下をまとめて報告する：
- リファクタリングしたファイルと変更内容
- `@theme` に追加・整理したトークン
- 新たに作成したコンポーネント（あれば）
```

---

## Phase 4: Tailwind CSS レビュー

Phase 3 完了後、Agent ツールで以下のプロンプトを渡してサブエージェントを起動する。

```
プロジェクトのTailwind CSSコードをレビューし、最適化してください。

`.claude/skills/tailwind-review/SKILL.md` を Read ツールで直接読み込み、記載の手順に従って実行してください。

修正はすべて確認なしで自動適用してください（パイプライン内での実行のため）。

> **Note:** Skill ツールで `tailwind-review` を呼び出せない場合（CWD不一致など）は、`.claude/skills/tailwind-review/SKILL.md` を直接読んで手順に従うこと。
```

---

## 最終報告

4フェーズすべて完了後、以下の形式でまとめて報告する：

```
## Figma ワークフロー完了

### Phase 1: 実装
[Agent 1 の報告内容を要約]

### Phase 2: レビュー
[Agent 2 の報告内容を要約]

### Phase 3: 最適化
[Agent 3 の報告内容を要約]

### Phase 4: Tailwind CSS レビュー
[Agent 4 の報告内容を要約]
```
