---
title: "【RSA暗号の数学 #2】モジュラ演算 — 時計の算術が暗号になるまで"
emoji: "⏰"
type: "tech"
topics: ["RSA", "暗号", "Python", "整数論"]
published: true
scheduled_publish_at: "2026-03-23 09:00"
---

[前回の記事](#1)では、RSA暗号の安全性が「素因数分解の困難性」に依存していることを見ました。

しかし、素数だけでは暗号は作れません。RSA暗号のもう一つの柱が**モジュラ演算**（合同算術）です。暗号の計算はすべてこの「割った余りの世界」で行われます。

今回は、モジュラ演算の基礎からフェルマーの小定理、オイラーの定理まで、RSAの数学的基盤を整えます。

## モジュラ演算 — 割った余りの世界

モジュラ演算は「時計の算術」とも呼ばれます。12時間制の時計では、14時は2時。数学的に書くと：

$$
14 \equiv 2 \pmod{12}
$$

一般に、$a$ を $m$ で割った余りが $b$ と等しいとき、$a \equiv b \pmod{m}$ と書きます。

### 基本性質

モジュラ演算が暗号に使える理由は、加法・乗法・べき乗が「余りの世界で閉じている」からです。

$$
\begin{aligned}
(a + b) \bmod m &= ((a \bmod m) + (b \bmod m)) \bmod m \\
(a \times b) \bmod m &= ((a \bmod m) \times (b \bmod m)) \bmod m
\end{aligned}
$$

これにより、巨大な数のべき乗も「途中で余りを取りながら」計算できます。

```python
# 素朴な方法: 7^256 を計算してから mod 13
naive = (7 ** 256) % 13
print(f"素朴: {naive}")  # 9

# 組み込み pow: 途中で余りを取りながら計算（高速）
fast = pow(7, 256, 13)
print(f"高速: {fast}")    # 9（同じ結果）
```

### 繰り返し二乗法

`pow(base, exp, mod)` の内部では**繰り返し二乗法**が使われています。指数を2進展開し、二乗を繰り返すことで $O(\log e)$ 回の乗算で $a^e \bmod m$ を計算します。

```python
def mod_pow(base: int, exp: int, mod: int) -> int:
    """繰り返し二乗法: base^exp mod mod を O(log exp) で計算"""
    result = 1
    base = base % mod
    while exp > 0:
        if exp & 1:  # exp の最下位ビットが 1 なら
            result = (result * base) % mod
        exp >>= 1
        base = (base * base) % mod
    return result

# 例: 7^256 mod 13
print(mod_pow(7, 256, 13))  # 9
```

$e$ が数百桁（2048ビット）であっても、2048回の乗算で済みます。RSAの暗号化・復号が実用的な速度で動く理由はここにあります。

## 一方向性 — 暗号の本質

モジュラ演算が暗号に使える最大の理由は、**一方向性**を持つことです。

- **順方向**（べき乗）: $a^e \bmod n$ の計算は $O(\log e)$ で高速
- **逆方向**（離散対数）: $a^x \equiv c \pmod{n}$ を満たす $x$ を求めるのは一般に困難

この非対称性が、前回見た「掛け算 vs 素因数分解」の非対称性と組み合わさって、RSA暗号を構成します。

## ユークリッドの互除法 — 2300年前の最古のアルゴリズム

RSAの鍵生成には**最大公約数**（GCD）の計算が不可欠です。ユークリッドの互除法は紀元前300年頃に遡る、おそらく人類最古のアルゴリズムです。

$$
\gcd(a, b) = \gcd(b, a \bmod b)
$$

```python
def gcd(a: int, b: int) -> int:
    """ユークリッドの互除法: O(log min(a, b))"""
    while b:
        a, b = b, a % b
    return a

print(gcd(252, 105))  # 21
print(gcd(65537, 3120))  # 1（互いに素 → RSAで使える）
```

### 拡張ユークリッドの互除法

GCDを求めるだけでなく、$ax + by = \gcd(a, b)$ を満たす $x, y$ も同時に求められます。これが**拡張ユークリッドの互除法**です。

```python
def extended_gcd(a: int, b: int) -> tuple[int, int, int]:
    """拡張ユークリッド: gcd = a*x + b*y を満たす (gcd, x, y) を返す"""
    if a == 0:
        return b, 0, 1
    g, x, y = extended_gcd(b % a, a)
    return g, y - (b // a) * x, x

g, x, y = extended_gcd(252, 105)
print(f"gcd = {g}, x = {x}, y = {y}")
print(f"検証: 252 * {x} + 105 * {y} = {252 * x + 105 * y}")
```

これがなぜ重要なのか。RSAの秘密鍵の計算に直結するからです。

## モジュラ逆元 — RSA秘密鍵の正体

整数の世界では $5 \div 3$ は整数になりません。しかしモジュラ演算の世界では、「割り算に相当する操作」が可能です。

$a$ の $\bmod m$ での**逆元**とは、次を満たす $a^{-1}$ です：

$$
a \cdot a^{-1} \equiv 1 \pmod{m}
$$

これは $a$ と $m$ が互いに素（$\gcd(a, m) = 1$）のときに限り存在します。

拡張ユークリッドの互除法で $ax + my = 1$ を求めれば、$ax \equiv 1 \pmod{m}$ なので $x$ が逆元です。

```python
def mod_inverse(a: int, m: int) -> int:
    """モジュラ逆元: a * x ≡ 1 (mod m) を満たす x"""
    g, x, _ = extended_gcd(a % m, m)
    if g != 1:
        raise ValueError(f"{a} と {m} は互いに素ではない")
    return x % m

# 例: 3 の mod 7 での逆元
inv = mod_inverse(3, 7)
print(f"3^(-1) mod 7 = {inv}")         # 5
print(f"検証: 3 * {inv} mod 7 = {3 * inv % 7}")  # 1
```

**RSAの秘密鍵 $d$ は、公開指数 $e$ の $\bmod\ \phi(n)$ での逆元です。** つまり：

$$
e \cdot d \equiv 1 \pmod{\phi(n)}
$$

この $d$ を拡張ユークリッドで求めるのが、RSA鍵生成の核心部分です。

## フェルマーの小定理

**フェルマーの小定理**は、素数とモジュラ演算を結びつける美しい定理です。

$p$ が素数で $\gcd(a, p) = 1$ のとき：

$$
a^{p-1} \equiv 1 \pmod{p}
$$

```python
# フェルマーの小定理の検証
p = 17
for a in range(1, p):
    result = pow(a, p - 1, p)
    assert result == 1

print(f"すべての a ∈ {{1,...,{p-1}}} で a^(p-1) ≡ 1 (mod p) を確認")
```

前回のミラー・ラビン素数判定法は、この定理の対偶を利用していました。$a^{p-1} \not\equiv 1 \pmod{n}$ なら $n$ は素数ではない、という論理です。

### 直感的な理解

$a = 3, p = 7$ で考えてみましょう。$3$ のべき乗を $\bmod 7$ で計算すると：

$$
3^1 \equiv 3, \quad 3^2 \equiv 2, \quad 3^3 \equiv 6, \quad 3^4 \equiv 4, \quad 3^5 \equiv 5, \quad 3^6 \equiv 1
$$

$\{1, 2, 3, 4, 5, 6\}$ が全て現れて、$3^6$ で $1$ に戻ります。$p$ が素数のとき、$\{1, 2, \ldots, p-1\}$ は乗法について**巡回群**をなし、必ず $p-1$ ステップ以内に $1$ に戻るのです。

## オイラーの定理 — フェルマーの一般化

フェルマーの小定理は「$p$ が素数」の場合に限られますが、**オイラーの定理**はこれを一般の整数に拡張します。

$\gcd(a, n) = 1$ のとき：

$$
a^{\phi(n)} \equiv 1 \pmod{n}
$$

ここで $\phi(n)$ は**オイラーのトーシェント関数**——$1$ から $n$ までの整数のうち $n$ と互いに素なものの個数です。

```python
def euler_totient(n: int) -> int:
    """オイラーのトーシェント関数 φ(n)"""
    result = n
    p = 2
    temp = n
    while p * p <= temp:
        if temp % p == 0:
            while temp % p == 0:
                temp //= p
            result -= result // p
        p += 1
    if temp > 1:
        result -= result // temp
    return result

# φ(12) = |{1, 5, 7, 11}| = 4
print(f"φ(12) = {euler_totient(12)}")  # 4

# φ(p) = p - 1（p が素数のとき）
print(f"φ(17) = {euler_totient(17)}")  # 16
```

### RSAにとって重要な公式

$n = pq$（$p, q$ は異なる素数）のとき：

$$
\phi(n) = \phi(pq) = (p - 1)(q - 1)
$$

RSAでは $n$ の素因数 $p, q$ を知っている人だけが $\phi(n)$ を計算でき、したがって秘密鍵 $d$ を計算できます。$n$ だけ知っている人は $\phi(n)$ を求められません（それには素因数分解が必要）。

```python
p, q = 61, 53
n = p * q           # 3233（公開）
phi_n = (p-1)*(q-1) # 3120（秘密 — p,qを知らないと計算できない）

print(f"n = {n}")
print(f"φ(n) = {phi_n}")

# オイラーの定理の検証: a^φ(n) ≡ 1 (mod n)
a = 17
print(f"{a}^{phi_n} mod {n} = {pow(a, phi_n, n)}")  # 1
```

## RSAの数学的構造（予告）

ここまでの道具を使って、RSAの暗号化・復号がなぜ機能するかを一行で書くと：

$$
(m^e)^d = m^{ed} = m^{1 + k\phi(n)} = m \cdot (m^{\phi(n)})^k \equiv m \cdot 1^k = m \pmod{n}
$$

暗号化 $m^e$ して復号 $(m^e)^d$ すると元のメッセージ $m$ に戻る。オイラーの定理がこの「魔法」の正体です。

詳細な実装は次回。

## まとめ

今回の内容を整理します。

| 概念 | 内容 | RSAでの役割 |
|:--|:--|:--|
| モジュラ演算 | 余りの世界の算術 | 全ての暗号計算の基盤 |
| 繰り返し二乗法 | $a^e \bmod n$ を $O(\log e)$ で計算 | 暗号化・復号の実行 |
| ユークリッドの互除法 | GCDを $O(\log n)$ で計算 | $e$ と $\phi(n)$ の互いに素の確認 |
| モジュラ逆元 | $a \cdot a^{-1} \equiv 1 \pmod{m}$ | **秘密鍵 $d$ の計算** |
| フェルマーの小定理 | $a^{p-1} \equiv 1 \pmod{p}$ | 素数判定の基礎 |
| オイラーの定理 | $a^{\phi(n)} \equiv 1 \pmod{n}$ | **RSAの正当性の数学的根拠** |

次回はいよいよ、これらの道具を使って**RSA暗号をPythonでゼロから実装**します。鍵生成、暗号化、復号を一つずつ組み立てていきましょう。
