---
title: "Qiitaの数式記法と標準LaTeXの違い一覧"
emoji: "🔍"
type: "tech"
topics: ["LaTeX", "数学", "Qiita", "KaTeX"]
published: true
scheduled_publish_at: "2026-03-09 09:00"
---

Qiitaで数式を書いていると、「あれ、この記法が動かない」と困ることがあります。標準のLaTeXなら当たり前に使える記法が、Qiitaでは使えないケースが少なくありません。

これはQiitaの数式レンダリングが **KaTeX** ベースであることに起因しています。KaTeXは高速で美しいレンダリングが特長ですが、LaTeXの全機能をカバーしているわけではありません。

この記事では、Qiitaで使える記法・使えない記法を整理し、使えない場合の代替手段もまとめます。

## QiitaのKaTeX環境の基本仕様

| 項目 | 内容 |
|------|------|
| レンダリングエンジン | KaTeX |
| インライン数式 | `$...$` |
| ブロック数式 | `$$...$$` |
| マクロ定義 | 非対応（`\newcommand` 不可） |
| 外部パッケージ | 読み込み不可 |
| 自動数式番号 | 非対応（`\label` / `\ref` 不可） |

## 使える記法一覧

まず、Qiitaで問題なく使える記法を確認しましょう。

### 基本的な数式記号

| 記法 | コード | 備考 |
|------|--------|------|
| 分数 | `\frac{a}{b}` | `\dfrac`, `\cfrac` も可 |
| 上付き・下付き | `x^2`, `x_i` | ネストも可 |
| 平方根 | `\sqrt{x}`, `\sqrt[3]{x}` | n乗根も対応 |
| ドット | `\cdot`, `\cdots`, `\ldots`, `\vdots`, `\ddots` | |
| 矢印 | `\to`, `\Rightarrow`, `\iff`, `\mapsto` | 主要なものは対応 |
| 集合 | `\in`, `\subset`, `\cup`, `\cap`, `\setminus`, `\emptyset` | |
| 論理 | `\forall`, `\exists`, `\neg`, `\land`, `\lor` | |
| 関係 | `\leq`, `\geq`, `\neq`, `\sim`, `\simeq`, `\cong`, `\equiv` | |
| ギリシャ文字 | `\alpha`, `\beta`, ..., `\omega` | 大文字も可 |

### 環境

| 環境 | コード例 | 備考 |
|------|---------|------|
| 整列 | `\begin{aligned}...\end{aligned}` | `align` ではなく `aligned` |
| 場合分け | `\begin{cases}...\end{cases}` | |
| 行列 | `\begin{pmatrix}...\end{pmatrix}` | `bmatrix`, `vmatrix` 等も可 |
| 小行列 | `\begin{smallmatrix}...\end{smallmatrix}` | インライン向け |
| 配列 | `\begin{array}{cc}...\end{array}` | |
| 下括弧 | `\underbrace{x+y}_{z}` | `\overbrace` も可 |

### 装飾・フォント

| 記法 | コード | 表示例の説明 |
|------|--------|------------|
| 太字 | `\mathbf{A}` | ベクトル・行列 |
| 太字（ギリシャ文字対応） | `\boldsymbol{\alpha}` | |
| カリグラフィ | `\mathcal{F}` | フィルター、σ加法族 |
| 花文字 | `\mathbb{R}` | 数体、集合 |
| フラクトゥール | `\mathfrak{g}` | リー代数 |
| ローマン体 | `\mathrm{d}x` | 微分のd |
| テキスト | `\text{hoge}` | 数式中の文字列 |
| 色 | `\color{red}{x}` | |
| 打ち消し線 | `\cancel{x}` | |
| 箱囲み | `\boxed{E=mc^2}` | |

### アクセント

| 記法 | コード | 用途 |
|------|--------|------|
| ハット | `\hat{x}` | 推定量 |
| ワイドハット | `\widehat{ABC}` | |
| バー | `\bar{x}` | 平均、共役 |
| オーバーライン | `\overline{AB}` | 線分、共役 |
| チルダ | `\tilde{x}` | |
| ベクトル矢印 | `\vec{v}` | |
| ドット | `\dot{x}`, `\ddot{x}` | 時間微分 |

## 使えない記法一覧

ここからが本題です。標準LaTeXでは使えるのに、Qiitaでは動作しない記法をまとめます。

### 環境・パッケージ系

| 使えない機能 | 標準LaTeXでの書き方 | Qiitaでの状況 |
|-------------|-------------------|-------------|
| 定理環境 | `\begin{theorem}...\end{theorem}` | 完全に非対応。Markdownで代替するしかない |
| 証明環境 | `\begin{proof}...\end{proof}` | 非対応。QEDマーク $\square$ は `\square` で出せる |
| 可換図式 | `\begin{tikzcd}...\end{tikzcd}` | 非対応。画像で代替 |
| グラフ描画 | `\begin{tikzpicture}...\end{tikzpicture}` | 非対応 |
| アルゴリズム | `\begin{algorithm}...\end{algorithm}` | 非対応。コードブロックで代替 |
| 数式番号の自動参照 | `\label{eq:1}`, `\ref{eq:1}` | 非対応。`\tag{}` で手動番号のみ |
| マクロ定義 | `\newcommand{\R}{\mathbb{R}}` | 非対応。毎回フルで書く必要あり |
| align環境 | `\begin{align}...\end{align}` | **非対応**。`aligned` を `$$` 内で使う |
| gather環境 | `\begin{gather}...\end{gather}` | 非対応。`aligned` で `&` なしで代替 |
| subequations | `\begin{subequations}...\end{subequations}` | 非対応 |
| `\DeclareMathOperator` | `\DeclareMathOperator{\Hom}{Hom}` | 非対応。`\operatorname{Hom}` で代替 |

### 記号・コマンド系

| 使えない機能 | 標準LaTeXでの書き方 | 代替手段 |
|-------------|-------------------|---------|
| `\mathscr` | `\mathscr{L}` | `\mathcal{L}` で近似 |
| 一部の矢印 | `\rightrightarrows` 等 | KaTeXのサポート外のものがある |
| `\xrightarrow` の複雑な使用 | `\xrightarrow[\text{下}]{\text{上}}` | 基本形は使えるが複雑な内容だと崩れる場合あり |

## `align` vs `aligned` — 最大の落とし穴

標準LaTeXに慣れた人が最もハマるポイントがこれです。

**標準LaTeX（動かない）：**
```latex
\begin{align}
a &= b + c \\
  &= d
\end{align}
```

**Qiita（正しい書き方）：**
```latex
$$
\begin{aligned}
a &= b + c \\
  &= d
\end{aligned}
$$
```

違いは以下の通りです：

| | `align` | `aligned` |
|--|---------|-----------|
| 種別 | トップレベル環境 | `$$` 内で使うサブ環境 |
| 数式番号 | 自動で付く（標準LaTeX） | 付かない |
| Qiita対応 | **非対応** | 対応 |

同様に、`equation` 環境も使えません。ブロック数式は全て `$$...$$` で書く必要があります。

## `\newcommand` が使えない問題

標準LaTeXでは、長い記法をマクロにまとめるのが定石です。

```latex
% 標準LaTeXなら定義できる
\newcommand{\R}{\mathbb{R}}
\newcommand{\norm}[1]{\left\| #1 \right\|}

% 以降は $\R$ や $\norm{x}$ で書ける
```

Qiitaではこれが使えないため、毎回 `\mathbb{R}` や `\left\| x \right\|` と書く必要があります。

**対処法：**
- エディタのスニペット機能を活用する
- 執筆中はテキストエディタの検索置換で展開する
- 記事の冒頭に「本記事で使う記法」セクションを設けて、読者の理解を助ける

## 代替表記まとめ

使えない記法に対して、Qiitaで取りうる代替手段を一覧にまとめます。

| やりたいこと | 標準LaTeX | Qiitaでの代替 |
|-------------|----------|--------------|
| 定理環境 | `\begin{theorem}` | Markdown引用ブロック `>` やHTMLタグで装飾 |
| 証明環境 | `\begin{proof}` | `>` ブロック + `$\square$` |
| 可換図式 | `tikzcd` | 画像として埋め込む |
| 自動数式番号 | `\label` / `\ref` | `\tag{1}` で手動管理 |
| マクロ | `\newcommand` | 毎回フルで記述 |
| 演算子定義 | `\DeclareMathOperator` | `\operatorname{名前}` |
| align | `\begin{align}` | `\begin{aligned}` を `$$` 内で |
| gather | `\begin{gather}` | `\begin{aligned}` で `&` 省略 |
| アルゴリズム | `algorithm` / `algorithmic` | Markdownコードブロック + 擬似コード |
| グラフ描画 | `tikzpicture` | Mermaid記法（Qiita対応）または画像 |

## KaTeXのバージョンに注意

QiitaのKaTeXバージョンは必ずしも最新ではありません。KaTeX本体では対応していても、Qiita側のバージョンでは未対応という記法が存在する可能性があります。

書いた数式が正しく表示されるかは、必ずQiitaのプレビュー画面で確認しましょう。

ローカルで事前確認したい場合は、[KaTeX公式サイト](https://katex.org/)のデモページでテストできます。

## まとめ

- Qiitaの数式はKaTeXベースであり、標準LaTeXの全機能は使えない
- 最大の注意点は `align` → `aligned`、`\newcommand` 非対応、定理環境非対応の3つ
- 可換図式やtikz系の描画機能は一切使えない
- 代替手段を知っておけば、多くのケースはQiitaでもカバーできる
- ただし、数学の専門的な記事になるほど制約が厳しくなるのも事実

標準LaTeXとの違いを把握しておけば、「書いたのに表示されない」というストレスを避けられます。この記事が執筆時のリファレンスとして役立てば幸いです。
