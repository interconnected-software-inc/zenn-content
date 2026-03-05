---
title: "Qiitaで定理・証明・補題を見やすく書くテクニック"
emoji: "📝"
type: "tech"
topics: ["LaTeX", "数学", "定理", "証明"]
published: true
scheduled_publish_at: "2026-03-12 09:00"
---

数学の記事を書くとき、定理（Theorem）、補題（Lemma）、証明（Proof）を明確に区別して表示したいという需要は大きいです。標準LaTeXなら `\begin{theorem}` と書けば済む話ですが、Qiitaにはそういった環境がありません。

だからといって、のっぺりとした文章の中に定理や証明を埋もれさせてしまうのはもったいないです。Markdownの機能とちょっとした工夫で、十分に読みやすい構造を作れます。

この記事では、実際に使えるテクニックを具体例つきで紹介します。

## 方法1：引用ブロックを使う（最もシンプル）

Markdownの引用ブロック `>` は、定理の強調にそのまま使えます。

```markdown
> **定理 1（ラグランジュの定理）**
> 有限群 $G$ とその部分群 $H$ について、$H$ の位数は $G$ の位数の約数である。
> $$
> |G| = [G : H] \cdot |H|
> $$
```

**表示結果：**

> **定理 1（ラグランジュの定理）**
> 有限群 $G$ とその部分群 $H$ について、$H$ の位数は $G$ の位数の約数である。
> $$
> |G| = [G : H] \cdot |H|
> $$

シンプルですが、背景色が変わることで本文と明確に区別できます。Qiitaの引用ブロックは薄いグレー背景になるため、定理の「枠」として十分に機能します。

## 方法2：水平線で区切る

引用ブロックの代わりに、水平線（`---`）で定理を囲む方法です。

```markdown
---

**定理 2（オイラーの公式）**

任意の実数 $\theta$ に対して、

$$
e^{i\theta} = \cos\theta + i\sin\theta
$$

---
```

**表示結果：**

---

**定理 2（オイラーの公式）**

任意の実数 $\theta$ に対して、

$$
e^{i\theta} = \cos\theta + i\sin\theta
$$

---

テキストの前後に水平線が入るだけですが、セクションの区切りとして機能し、定理が独立したブロックであることが視覚的に伝わります。

## 方法3：HTMLタグで定理ボックスを作る

Qiitaでは一部のHTMLタグとインラインスタイルが使えます。これを活用すると、より本格的な定理ボックスが作れます。

```html
<div style="border-left: 4px solid #1a73e8; padding: 12px 16px; margin: 16px 0; background: #f8f9fa;">

**定理 3（中間値の定理）**

$f$ が閉区間 $[a, b]$ 上の連続関数で $f(a) \neq f(b)$ のとき、$f(a)$ と $f(b)$ の間の任意の値 $c$ に対して、$f(\xi) = c$ を満たす $\xi \in (a, b)$ が存在する。

</div>
```

> **注意：** QiitaではHTMLの `style` 属性が制限される場合があります。投稿前に必ずプレビューで確認してください。CSSが無効化されている場合は、方法1の引用ブロックが最も確実です。

### 種類ごとに色を変える

定理・補題・系をそれぞれ別の色で区別するとさらに読みやすくなります。

```html
<!-- 定理：青 -->
<div style="border-left: 4px solid #1a73e8; padding: 12px 16px; margin: 16px 0; background: #f8f9fa;">

**定理（Theorem）**
内容...

</div>

<!-- 補題：緑 -->
<div style="border-left: 4px solid #34a853; padding: 12px 16px; margin: 16px 0; background: #f8f9fa;">

**補題（Lemma）**
内容...

</div>

<!-- 系：オレンジ -->
<div style="border-left: 4px solid #fa7b17; padding: 12px 16px; margin: 16px 0; background: #f8f9fa;">

**系（Corollary）**
内容...

</div>
```

色の使い分けは記事の冒頭で軽く説明しておくと親切です。

## 証明の書き方

### 基本パターン

証明は定理とは異なる見た目にしたいので、引用ブロックは使わず、折りたたみ（`<details>`）を使う方法が効果的です。

```html
<details>
<summary><b>証明</b></summary>

$G$ の $H$ による左剰余類の集合を考える。各剰余類 $aH$ は $|H|$ 個の元を含み、異なる剰余類は互いに素である。$G$ は有限個の剰余類の非交和に分解されるから、

$$
|G| = (\text{剰余類の個数}) \times |H| = [G:H] \cdot |H|
$$

$\square$
</details>
```

**表示結果：**

<details>
<summary><b>証明</b></summary>

$G$ の $H$ による左剰余類の集合を考える。各剰余類 $aH$ は $|H|$ 個の元を含み、異なる剰余類は互いに素である。$G$ は有限個の剰余類の非交和に分解されるから、

$$
|G| = (\text{剰余類の個数}) \times |H| = [G:H] \cdot |H|
$$

$\square$
</details>

折りたたみを使うことで、まず定理の主張だけを読んで、証明は必要なときに展開するという読み方が可能になります。長い証明がある記事では特に効果的です。

### QEDマークの表現

標準LaTeXの `\begin{proof}` 環境では、証明の末尾に自動でQEDマーク（$\square$）が表示されます。Qiitaでは自分で書く必要があります。

いくつかの選択肢があります：

| 表記 | コード | 見た目 |
|------|--------|--------|
| 白四角 | `$\square$` | $\square$ |
| 黒四角 | `$\blacksquare$` | $\blacksquare$ |
| テキスト | `■` | ■ |
| 右寄せ | `<div style="text-align: right;">$\square$</div>` | 右端に配置 |

右寄せがLaTeXの慣習に近いですが、Qiitaでスタイルが効かない場合は末尾にそのまま `$\square$` を書くのが確実です。

### 証明を展開状態で表示したい場合

`<details open>` とすると、最初から展開された状態になります。

```html
<details open>
<summary><b>証明</b></summary>

証明本文...

$\square$
</details>
```

## 定理・補題・系・命題の書き分け

数学の記事では、以下の概念を区別して書きます。

| 種別 | 意味 | 使いどころ |
|------|------|-----------|
| **定理（Theorem）** | 主要な結果 | 記事の中心となる結果 |
| **補題（Lemma）** | 定理の証明に使う補助的な命題 | 定理の証明準備 |
| **命題（Proposition）** | 定理ほど重要でないが、自明でもない結果 | 中規模の結果 |
| **系（Corollary）** | 定理から直ちに従う結果 | 定理の直後に配置 |
| **定義（Definition）** | 用語・概念の定義 | 使用前に配置 |
| **例（Example）** | 具体例 | 定義や定理の直後に配置 |
| **注意（Remark）** | 補足事項 | 適宜 |

### 番号の付け方

番号の付け方にはいくつかの流派があります。

**方式1：種別ごとに独立した番号**

> **定理 1**, **定理 2**, **補題 1**, **補題 2** ...

**方式2：通し番号（推奨）**

> **定理 1**, **補題 2**, **定理 3**, **系 4** ...

通し番号の方が「補題2は定理3の前にある」ということが番号だけで分かるので、参照しやすくなります。論文やテキストでも通し番号が主流です。

**方式3：セクション番号つき**

> **定理 2.1**, **補題 2.2**, **定理 3.1** ...

長い記事ではセクション番号をつけると追いやすくなります。

## 実践例：群論の準同型定理

ここまでのテクニックを組み合わせて、群論の第一同型定理を書いてみましょう。

---

### 第一同型定理

> **定義 1（群準同型）**
> 群 $(G, \cdot)$ と $(H, \ast)$ に対して、写像 $\varphi: G \to H$ が**群準同型**であるとは、任意の $a, b \in G$ に対して
> $$
> \varphi(a \cdot b) = \varphi(a) \ast \varphi(b)
> $$
> が成り立つことをいう。

> **定義 2（核と像）**
> 群準同型 $\varphi: G \to H$ に対して、
> $$
> \begin{aligned}
> \ker\varphi &= \{ g \in G \mid \varphi(g) = e_H \} \\
> \operatorname{Im}\varphi &= \{ \varphi(g) \mid g \in G \}
> \end{aligned}
> $$
> をそれぞれ $\varphi$ の**核**と**像**という。

> **補題 3**
> 群準同型 $\varphi: G \to H$ の核 $\ker\varphi$ は $G$ の正規部分群である。

<details>
<summary><b>証明</b></summary>

$\ker\varphi$ が部分群であることは容易に確かめられる。正規性を示す。任意の $g \in G$ と $n \in \ker\varphi$ に対して、

$$
\begin{aligned}
\varphi(gng^{-1}) &= \varphi(g)\varphi(n)\varphi(g^{-1}) \\
&= \varphi(g) \cdot e_H \cdot \varphi(g)^{-1} \\
&= e_H
\end{aligned}
$$

よって $gng^{-1} \in \ker\varphi$ であり、$\ker\varphi \trianglelefteq G$ が示された。 $\square$
</details>

> **定理 4（第一同型定理）**
> 群準同型 $\varphi: G \to H$ に対して、
> $$
> G / \ker\varphi \cong \operatorname{Im}\varphi
> $$

<details>
<summary><b>証明</b></summary>

$N = \ker\varphi$ とおく。写像 $\bar{\varphi}: G/N \to \operatorname{Im}\varphi$ を

$$
\bar{\varphi}(gN) = \varphi(g)
$$

で定義する。

**Well-defined性：** $gN = g'N$ とすると $g^{-1}g' \in N$ なので $\varphi(g^{-1}g') = e_H$。よって $\varphi(g) = \varphi(g')$ となり、$\bar{\varphi}$ は代表元の取り方によらない。

**準同型性：**

$$
\bar{\varphi}(gN \cdot g'N) = \bar{\varphi}(gg'N) = \varphi(gg') = \varphi(g)\varphi(g') = \bar{\varphi}(gN)\bar{\varphi}(g'N)
$$

**単射性：** $\bar{\varphi}(gN) = e_H$ とすると $\varphi(g) = e_H$ なので $g \in N$、すなわち $gN = N$。

**全射性：** $\operatorname{Im}\varphi$ の定義より明らか。

以上より $\bar{\varphi}$ は同型写像であり、$G/N \cong \operatorname{Im}\varphi$ が示された。 $\square$
</details>

> **系 5**
> 群準同型 $\varphi: G \to H$ が全射ならば $G / \ker\varphi \cong H$ である。

---

このように、引用ブロック、折りたたみ、通し番号を組み合わせることで、LaTeXの定理環境に近い構造を実現できます。

## テンプレート

最後に、コピペして使えるテンプレートをまとめます。

### 定理テンプレート

```markdown
> **定理 N（定理名）**
> 主張をここに書く。
> $$
> 数式
> $$
```

### 証明テンプレート（折りたたみ）

```markdown
<details>
<summary><b>証明</b></summary>

証明本文をここに書く。

$\square$
</details>
```

### 定義テンプレート

```markdown
> **定義 N（用語名）**
> 定義の内容をここに書く。
```

## まとめ

- 定理の強調には引用ブロック `>` + 太字が最も確実で簡単
- 証明は `<details>` で折りたたむと読みやすい
- QEDマークは `$\square$` または `$\blacksquare$` を手動で配置
- 定理・補題・系には通し番号を振ると参照しやすい
- HTMLの `style` 属性で装飾できる場合もあるが、Qiitaの仕様変更で効かなくなるリスクがある
- 引用ブロック方式なら、Qiitaの仕様が変わっても壊れにくい

標準LaTeXの定理環境には及びませんが、これらのテクニックを使えば、Qiitaでも数学的に構造化された記事を書くことは十分に可能です。
