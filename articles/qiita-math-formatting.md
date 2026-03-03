---
title: "Qiitaで数式を綺麗に表示する方法まとめ"
emoji: "📐"
type: "tech"
topics: ["LaTeX", "数学", "Qiita", "KaTeX"]
published: false
scheduled_publish_at: "2026-03-05 09:00"
---

Qiitaで数学系の記事を書くとき、数式の見た目は読みやすさに直結します。せっかく内容が良くても、数式が崩れていたり読みにくかったりすると、それだけで離脱されてしまいます。

この記事では、Qiitaで数式を書くための基本から実践的なテクニックまで、コード例つきで網羅的にまとめます。

## 基本：インライン数式とブロック数式

Qiitaでは KaTeX ベースの数式レンダリングが使えます。書き方は2種類です。

### インライン数式

文中に数式を埋め込むには `$...$` で囲みます。

```
関数 $f(x) = x^2 + 1$ は全ての実数で定義される。
```

**表示結果：** 関数 $f(x) = x^2 + 1$ は全ての実数で定義される。

### ブロック数式

独立した行に数式を表示するには `$$...$$` で囲みます。前後に空行を入れるのがポイントです。

```
$$
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$
```

**表示結果：**

$$
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$

> **注意：** `$$` の前後に空行がないと、正しくレンダリングされないことがあります。必ず空行を入れましょう。

## よく使う数式パターン

### 分数

```latex
$\frac{a}{b}$          % 基本の分数
$\dfrac{a}{b}$         % 大きめの分数（インラインで使うと行間が広がるので注意）
$\cfrac{1}{1+\cfrac{1}{2}}$  % 連分数
```

**表示結果：**

$$
\frac{a}{b}, \quad \dfrac{a}{b}, \quad \cfrac{1}{1+\cfrac{1}{2}}
$$

### 総和・総乗

```latex
$\sum_{k=1}^{n} k = \frac{n(n+1)}{2}$           % インライン
```

ブロックで書くと添字が上下に配置されます：

```latex
$$
\sum_{k=1}^{n} k = \frac{n(n+1)}{2}
$$
```

$$
\sum_{k=1}^{n} k = \frac{n(n+1)}{2}
$$

総乗も同様です：

```latex
$$
\prod_{i=1}^{n} a_i = a_1 \cdot a_2 \cdots a_n
$$
```

### 積分

```latex
$$
\int_a^b f(x)\,dx
$$
```

$$
\int_a^b f(x)\,dx
$$

多重積分や線積分：

```latex
$$
\iint_D f(x,y)\,dx\,dy, \quad \oint_C \mathbf{F} \cdot d\mathbf{r}
$$
```

$$
\iint_D f(x,y)\,dx\,dy, \quad \oint_C \mathbf{F} \cdot d\mathbf{r}
$$

### 極限

```latex
$$
\lim_{n \to \infty} \left(1 + \frac{1}{n}\right)^n = e
$$
```

$$
\lim_{n \to \infty} \left(1 + \frac{1}{n}\right)^n = e
$$

### 行列

```latex
$$
A = \begin{pmatrix}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
a_{31} & a_{32} & a_{33}
\end{pmatrix}
$$
```

$$
A = \begin{pmatrix}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
a_{31} & a_{32} & a_{33}
\end{pmatrix}
$$

括弧の種類を変えたい場合：

| 環境 | 括弧の形 |
|------|---------|
| `pmatrix` | 丸括弧 $(\ )$ |
| `bmatrix` | 角括弧 $[\ ]$ |
| `Bmatrix` | 波括弧 $\{\ \}$ |
| `vmatrix` | 縦棒（行列式） |
| `Vmatrix` | 二重縦棒 |

### 場合分け（cases）

```latex
$$
|x| = \begin{cases}
x & (x \geq 0) \\
-x & (x < 0)
\end{cases}
$$
```

$$
|x| = \begin{cases}
x & (x \geq 0) \\
-x & (x < 0)
\end{cases}
$$

## 整列環境（aligned）の使い方

複数行の数式を揃えて表示したい場合、`aligned` 環境を使います。`&` で揃える位置を指定し、`\\` で改行します。

> **注意：** 標準LaTeXでは `align` 環境を使いますが、Qiita（KaTeX）では `aligned` を `$$` の中で使う必要があります。

```latex
$$
\begin{aligned}
(a+b)^2 &= (a+b)(a+b) \\
&= a^2 + ab + ba + b^2 \\
&= a^2 + 2ab + b^2
\end{aligned}
$$
```

$$
\begin{aligned}
(a+b)^2 &= (a+b)(a+b) \\
&= a^2 + ab + ba + b^2 \\
&= a^2 + 2ab + b^2
\end{aligned}
$$

連立方程式にも便利です：

```latex
$$
\begin{cases}
\begin{aligned}
2x + 3y &= 7 \\
x - y &= 1
\end{aligned}
\end{cases}
$$
```

## 数式番号について

Qiitaでは `\tag{}` を使って手動で数式番号をつけられます。

```latex
$$
E = mc^2 \tag{1}
$$
```

$$
E = mc^2 \tag{1}
$$

`\label` と `\ref` による自動参照はQiitaでは動作しません。数式番号を参照したい場合は、本文中に手動で「式(1)」のように書く必要があります。

```latex
$$
F = ma \tag{2.1}
$$

$$
W = \int_a^b F\,dx \tag{2.2}
$$
```

式(2.1)を式(2.2)に代入すると... のように手動で参照します。

## 綺麗に見せるためのTips

### 1. `\displaystyle` でインライン数式を大きく

インライン数式では、分数や総和が小さく表示されます。`\displaystyle` を使うと、ブロック数式と同じ大きさで表示できます。

```
通常: $\sum_{k=1}^n k$ → 小さい

大きく: $\displaystyle\sum_{k=1}^n k$ → 見やすい
```

ただし、行間が広がるので多用は禁物です。本当に強調したい箇所だけに使いましょう。

### 2. 適切なスペーシング

LaTeX数式内でのスペースコマンドを覚えておくと、数式が格段に読みやすくなります。

| コマンド | 幅 | 用途 |
|---------|-----|------|
| `\,` | 細いスペース | 積分の $dx$ の前、単位の前 |
| `\;` | やや広いスペース | 集合の条件区切り |
| `\quad` | 広いスペース | 数式の区切り |
| `\qquad` | とても広いスペース | 大きな区切り |

**Before：**
```latex
$$
\int_0^1 f(x)dx, \forall x \in S
$$
```

**After：**
```latex
$$
\int_0^1 f(x)\,dx, \quad \forall x \in S
$$
```

`\,dx` のスペースがあるだけで、積分がぐっと読みやすくなります。

### 3. `\left` `\right` で括弧のサイズを自動調整

中身が大きい数式に固定サイズの括弧を使うと不格好です。

```latex
% 固定サイズ（見にくい）
$$
(\frac{a}{b})^2
$$

% 自動調整（見やすい）
$$
\left(\frac{a}{b}\right)^2
$$
```

$$
\left(\frac{a}{b}\right)^2
$$

### 4. テキストを数式内に入れる

数式の途中に日本語や英語のテキストを入れたいときは `\text{}` を使います。

```latex
$$
P(\text{事象}A \mid \text{事象}B) = \frac{P(A \cap B)}{P(B)}
$$
```

$$
P(\text{事象}A \mid \text{事象}B) = \frac{P(A \cap B)}{P(B)}
$$

### 5. 色をつける

KaTeXでは `\color{}` で数式に色をつけられます。

```latex
$$
f(x) = \color{red}{a}x^2 + \color{blue}{b}x + \color{green}{c}
$$
```

$$
f(x) = \color{red}{a}x^2 + \color{blue}{b}x + \color{green}{c}
$$

強調したい部分に使うと効果的ですが、使いすぎると逆に読みにくくなります。

### 6. 太字・ボールド

ベクトルや行列を太字で書く場合：

```latex
$$
\mathbf{v} = \begin{pmatrix} v_1 \\ v_2 \\ v_3 \end{pmatrix}, \quad \boldsymbol{\alpha} + \boldsymbol{\beta}
$$
```

`\mathbf` は英字のみ、`\boldsymbol` はギリシャ文字にも使えます。

## よくあるエラーと対処法

### `&` がエスケープされてしまう

Markdownの中でそのまま `&` を書くと HTML エンティティとして解釈されることがあります。数式ブロック `$$` の中であれば基本的に問題ありませんが、うまくいかない場合は前後の空行を確認してください。

### 改行 `\\` が効かない

`aligned` や `cases` の外で `\\` を使っても改行されません。複数行の数式には必ず `aligned` 環境を使いましょう。

### インライン数式が途切れる

`$` が文中の他の `$` と誤マッチすることがあります。1つの段落に複数のインライン数式を書くときは、`$` の前後にスペースがあるか確認しましょう。

## まとめ

- インライン数式は `$...$`、ブロック数式は `$$...$$`（前後に空行を入れる）
- `aligned` 環境で数式を揃える（`align` ではなく `aligned` を使う）
- `\tag{}` で手動の数式番号を振れる
- `\,` や `\quad` のスペーシングで読みやすさが大きく変わる
- `\left` `\right` で括弧サイズを自動調整
- `\displaystyle` はインラインで大きく見せたいときに

これらを押さえておけば、Qiitaで十分に綺麗な数式記事が書けるはずです。ぜひ活用してみてください。
