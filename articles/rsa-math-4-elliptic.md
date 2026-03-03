---
title: "【RSA暗号の数学 #4】RSAの限界と楕円曲線暗号 — 次世代の暗号へ"
emoji: "🔐"
type: "tech"
topics: ["RSA", "暗号", "楕円曲線", "Python"]
published: false
scheduled_publish_at: "2026-03-30 09:00"
---

シリーズ最終回です。[#1](素数)、[#2](モジュラ演算)、[#3](実装)を通じて、RSA暗号を数学と実装の両面から理解してきました。

RSAは1977年の発表以来、50年近くにわたってインターネットの安全性を支えてきた偉大な暗号方式です。しかし、万能ではありません。

最終回では、RSAに対する攻撃手法、鍵長の肥大化問題、そしてそれを解決する**楕円曲線暗号**（ECC）を扱います。さらに、量子コンピュータがもたらす脅威にも触れます。

## RSAの弱点

### 攻撃1: 素因数分解アルゴリズムの進化

RSAの安全性は「$n = pq$ の素因数分解が困難」に依存しています。裏を返せば、素因数分解のアルゴリズムが進歩すれば、RSAは弱くなります。

| アルゴリズム | 計算量 | 特徴 |
|:--|:--|:--|
| 試し割り | $O(\sqrt{n})$ | 最も素朴 |
| Pollardの$\rho$法 | $O(n^{1/4})$ | 乱択アルゴリズム |
| 二次篩法（QS） | $L_n[1/2]$ | 中規模の数に有効 |
| 一般数体篩法（GNFS） | $L_n[1/3]$ | 現在最速 |

ここで $L_n[\alpha] = \exp(c \cdot (\ln n)^\alpha (\ln \ln n)^{1-\alpha})$ です。

GNFSの登場により、RSA-512（155桁）は1999年に、RSA-768（232桁）は2009年に分解されました。安全な鍵長は年々大きくなり、現在は2048ビット以上が推奨されています。

### 攻撃2: サイドチャネル攻撃

数学的ではない攻撃も存在します。

- **タイミング攻撃**: 復号の処理時間の差から秘密鍵の情報を推測
- **電力解析攻撃**: 暗号装置の消費電力パターンから鍵を復元
- **フォールト攻撃**: 計算中にエラーを意図的に発生させ、その結果から鍵を導出

CRT最適化された復号にフォールト攻撃を仕掛けると、1回のエラーから $p$ や $q$ を復元できることが知られています。

### 攻撃3: パディングの不備

#3で触れた通り、パディングなしの「教科書的RSA」は脆弱です。

- **同一メッセージ攻撃**: 同じ $m$ を $e = 3$ の異なる公開鍵で暗号化すると、中国剰余定理で $m^3$ を復元でき、3乗根を取れば $m$ が分かる
- **選択暗号文攻撃**: 任意の暗号文の復号結果を観察できれば、標的の暗号文を解読可能

これらは OAEP パディングで防御されますが、実装のバグが脆弱性に直結します。

## 鍵長の肥大化問題

安全性を維持するために鍵を長くすると、計算コストが増大します。

| 安全性レベル | RSA鍵長 | 鍵サイズ |
|:--|:--|:--|
| 80ビット | 1024 | 128バイト |
| 112ビット | 2048 | 256バイト |
| 128ビット | 3072 | 384バイト |
| 256ビット | 15360 | 1920バイト |

RSAの暗号化・復号の計算量は鍵長の3乗に比例するため、鍵を2倍にすると処理は8倍遅くなります。IoT機器やモバイルデバイスでは、この計算コストが無視できません。

## 楕円曲線暗号（ECC）の登場

楕円曲線暗号は、同じ安全性をはるかに短い鍵で実現します。

| 安全性レベル | RSA | ECC | 比率 |
|:--|:--|:--|:--|
| 80ビット | 1024 | 160 | 6.4倍 |
| 112ビット | 2048 | 224 | 9.1倍 |
| 128ビット | 3072 | 256 | 12倍 |
| 256ビット | 15360 | 512 | 30倍 |

なぜこれほどの差が生まれるのか。それは、楕円曲線離散対数問題（ECDLP）に対して、GNFSのような準指数時間アルゴリズムが知られていないからです。最良の攻撃でもほぼ指数時間かかるため、鍵を短くしても安全性を保てます。

## 楕円曲線の群構造

楕円曲線とは、有限体 $\mathbb{F}_p$ 上の方程式

$$
y^2 \equiv x^3 + ax + b \pmod{p}
$$

を満たす点 $(x, y)$ の集合に、**無限遠点** $\mathcal{O}$（単位元）を加えたものです。

この点の集合に「加法」を定義できます。

### 楕円曲線上の加法

2点 $P, Q$ の「和」$P + Q$ は、幾何学的に定義されます：

1. $P$ と $Q$ を通る直線を引く
2. その直線と楕円曲線の第3の交点を求める
3. その交点を $x$ 軸について反転する

$P = Q$ の場合は、$P$ での接線を使います。

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int | None
    y: int | None

class EllipticCurve:
    """有限体 F_p 上の楕円曲線 y^2 = x^3 + ax + b"""

    def __init__(self, a: int, b: int, p: int):
        self.a = a
        self.b = b
        self.p = p
        # 判別式 ≠ 0 を確認（特異点がないこと）
        disc = (4 * a**3 + 27 * b**2) % p
        assert disc != 0, "特異曲線は使えない"

    def is_on_curve(self, P: Point) -> bool:
        """点が曲線上にあるか"""
        if P.x is None:
            return True  # 無限遠点
        lhs = (P.y * P.y) % self.p
        rhs = (P.x**3 + self.a * P.x + self.b) % self.p
        return lhs == rhs

    def add(self, P: Point, Q: Point) -> Point:
        """楕円曲線上の点の加法"""
        # 単位元（無限遠点）の処理
        if P.x is None:
            return Q
        if Q.x is None:
            return P

        # P + (-P) = O（逆元との和）
        if P.x == Q.x and (P.y + Q.y) % self.p == 0:
            return Point(None, None)

        if P == Q:
            # 接線の傾き: λ = (3x² + a) / (2y)
            lam = (3 * P.x * P.x + self.a) * pow(2 * P.y, -1, self.p) % self.p
        else:
            # 2点を結ぶ直線の傾き: λ = (y₂ - y₁) / (x₂ - x₁)
            lam = (Q.y - P.y) * pow(Q.x - P.x, -1, self.p) % self.p

        # 新しい点の座標
        x_r = (lam * lam - P.x - Q.x) % self.p
        y_r = (lam * (P.x - x_r) - P.y) % self.p
        return Point(x_r, y_r)

    def scalar_mult(self, k: int, P: Point) -> Point:
        """スカラー倍: kP = P + P + ... + P（k回）
        繰り返し二乗法で O(log k) に高速化
        """
        result = Point(None, None)  # 単位元
        current = P
        while k > 0:
            if k & 1:
                result = self.add(result, current)
            current = self.add(current, current)
            k >>= 1
        return result
```

### 群の公理の確認

楕円曲線上の点は、加法について群をなします：

- **閉性**: 2点の和は曲線上の点
- **結合律**: $(P + Q) + R = P + (Q + R)$
- **単位元**: 無限遠点 $\mathcal{O}$（$P + \mathcal{O} = P$）
- **逆元**: 各点 $P = (x, y)$ に対して $-P = (x, -y)$

```python
# 群の公理を実際に確認
curve = EllipticCurve(a=2, b=3, p=97)
P = Point(3, 6)
Q = Point(80, 10)

# 曲線上の点であることを確認
assert curve.is_on_curve(P)
assert curve.is_on_curve(Q)

# 加法の結果も曲線上
R = curve.add(P, Q)
assert curve.is_on_curve(R)
print(f"P + Q = ({R.x}, {R.y})")

# 単位元
O = Point(None, None)
assert curve.add(P, O) == P

# 逆元
neg_P = Point(P.x, (-P.y) % curve.p)
assert curve.add(P, neg_P) == O
print("群の公理を確認")
```

## ECDH鍵交換

楕円曲線暗号の最も基本的な応用が、**ECDH鍵交換**です。TLS 1.3ではデフォルトでECDHが使われています。

### 仕組み

1. Alice と Bob は、曲線とベースポイント $G$ を共有（公開情報）
2. Alice: 秘密鍵 $a$ を選び、公開鍵 $A = aG$ を公開
3. Bob: 秘密鍵 $b$ を選び、公開鍵 $B = bG$ を公開
4. Alice: $S = aB = a(bG) = abG$ を計算
5. Bob: $S = bA = b(aG) = abG$ を計算

両者が同じ点 $S = abG$ を得ます。盗聴者は $A = aG$ と $B = bG$ を見ますが、$a$ や $b$ を知らないので $abG$ を計算できません（ECDLP）。

```python
# ECDH鍵交換のデモ
import random

curve = EllipticCurve(a=2, b=3, p=97)
G = Point(3, 6)

# --- Alice ---
alice_secret = random.randrange(1, 97)
alice_public = curve.scalar_mult(alice_secret, G)
print(f"Alice 秘密鍵: {alice_secret}")
print(f"Alice 公開鍵: ({alice_public.x}, {alice_public.y})")

# --- Bob ---
bob_secret = random.randrange(1, 97)
bob_public = curve.scalar_mult(bob_secret, G)
print(f"\nBob 秘密鍵: {bob_secret}")
print(f"Bob 公開鍵: ({bob_public.x}, {bob_public.y})")

# --- 共有秘密の計算 ---
alice_shared = curve.scalar_mult(alice_secret, bob_public)
bob_shared = curve.scalar_mult(bob_secret, alice_public)

print(f"\nAlice の共有秘密: ({alice_shared.x}, {alice_shared.y})")
print(f"Bob の共有秘密:   ({bob_shared.x}, {bob_shared.y})")
print(f"一致: {alice_shared == bob_shared}")  # True!
```

### なぜECDLPは難しいのか

$P$ と $Q = kP$ が与えられたとき、$k$ を求める問題がECDLPです。

RSAの素因数分解には GNFS という準指数時間アルゴリズムが存在しますが、ECDLPに対して同等のアルゴリズムは知られていません。最良の攻撃は Pollard の $\rho$ 法で、計算量は $O(\sqrt{n})$（$n$ は群の位数）。これはほぼ総当たりと同じです。

この差が、ECCがRSAよりはるかに短い鍵で同等の安全性を実現できる理由です。

## 量子コンピュータの脅威

1994年、ピーター・ショアは量子コンピュータ上で動作する**ショアのアルゴリズム**を発表しました。このアルゴリズムは：

- **素因数分解**を多項式時間 $O((\log n)^3)$ で解く → RSAを破る
- **離散対数問題**を多項式時間で解く → ECCも破る

つまり、十分な規模の量子コンピュータが実現すれば、RSAもECCも安全ではなくなります。

### 現状

2024年時点で最大の量子コンピュータは約1000量子ビット。RSA-2048を破るには、エラー訂正込みで数百万量子ビットが必要と推定されています。すぐに危険というわけではありませんが、「今暗号化されたデータを保存しておき、量子コンピュータが実現したら復号する」（Harvest Now, Decrypt Later）という脅威は現実的です。

### ポスト量子暗号

この脅威に備えて、量子コンピュータでも破れない暗号方式の研究が進んでいます。NISTは2024年に以下の標準を選定しました：

| 方式 | ベースとなる数学 |
|:--|:--|
| ML-KEM（CRYSTALS-Kyber） | 格子問題 |
| ML-DSA（CRYSTALS-Dilithium） | 格子問題 |
| SLH-DSA（SPHINCS+） | ハッシュ関数 |

格子ベースの暗号は、量子コンピュータでも効率的に解けないとされる「最短ベクトル問題」に基づいています。

## RSAとECCの使い分け

現時点での実用的な選択基準をまとめます。

| 観点 | RSA | ECC |
|:--|:--|:--|
| 成熟度 | 50年の実績 | 30年の実績 |
| 鍵長 | 長い（2048+ bits） | 短い（256 bits） |
| 暗号化速度 | 速い（$e$ が小さい） | 不可（直接暗号化はしない） |
| 復号速度 | 遅い | — |
| 鍵交換 | DH（大きな鍵） | ECDH（小さな鍵、高速） |
| TLS 1.3 | 鍵交換には使えない | デフォルト |
| 量子耐性 | なし | なし |

TLS 1.3では鍵交換にECDH（またはX25519）が必須で、RSAは署名にのみ使用されます。新規のシステムでは ECC を選ぶのが標準です。

## シリーズまとめ

4回にわたって、RSA暗号の数学を基礎から積み上げてきました。

| 回 | テーマ | 核心 |
|:--|:--|:--|
| #1 | 素数と素因数分解 | 掛け算は簡単、分解は困難 |
| #2 | モジュラ演算 | オイラーの定理が暗号の正当性を保証 |
| #3 | 実装 | 数学の道具を組み合わせてRSAを構築 |
| #4 | 限界と次世代 | ECCは短い鍵で同等の安全性、量子が次の転換点 |

暗号技術は、純粋数学の美しい定理が日常のインフラを支えている稀有な分野です。素数定理、フェルマーの小定理、オイラーの定理、楕円曲線の群構造——これらの数学が、毎日何十億回もの通信を守っています。

---

**さらに体系的に学びたい方へ**

このシリーズでは暗号に関連する整数論と楕円曲線を扱いましたが、整数論にはさらに豊かな世界が広がっています。二次剰余の相互法則、原始根、p進数、ディリクレのL関数など、証明付きで段階的に学びたい方は [Folioの整数論シリーズ](https://interconnectd.app/ja/series) で無料公開しています。
