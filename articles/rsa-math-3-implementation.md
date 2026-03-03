---
title: "【RSA暗号の数学 #3】RSA暗号をPythonでゼロから実装する"
emoji: "🔧"
type: "tech"
topics: ["RSA", "暗号", "Python", "セキュリティ"]
published: false
scheduled_publish_at: "2026-03-26 09:00"
---

[#1](素数)で「掛け算は簡単、素因数分解は困難」という非対称性を、[#2](モジュラ演算)で「オイラーの定理」と「モジュラ逆元」を学びました。

道具は揃いました。今回はいよいよ、RSA暗号をPythonでゼロから実装します。ライブラリに頼らず、全てのステップを自分の手で組み立てることで、RSAの仕組みを完全に理解することが目標です。

## RSA暗号の全体像

まず、RSAの全体像を3ステップで整理します。

### Step 1: 鍵生成（秘密の操作）

1. 大きな素数 $p, q$ を生成する
2. $n = pq$ を計算する
3. $\phi(n) = (p-1)(q-1)$ を計算する
4. $\gcd(e, \phi(n)) = 1$ を満たす $e$ を選ぶ（通常 $e = 65537$）
5. $ed \equiv 1 \pmod{\phi(n)}$ を満たす $d$ を計算する

**公開鍵**: $(n, e)$ — 誰でも見られる
**秘密鍵**: $(n, d)$ — 本人だけが持つ

### Step 2: 暗号化（公開鍵で誰でもできる）

$$
c = m^e \bmod n
$$

### Step 3: 復号（秘密鍵がないとできない）

$$
m = c^d \bmod n
$$

## 前回までの道具を準備

#1 と #2 で実装した関数を再掲します。

```python
import random

def miller_rabin(n: int, k: int = 20) -> bool:
    """ミラー・ラビン素数判定法"""
    if n < 2:
        return False
    if n == 2 or n == 3:
        return True
    if n % 2 == 0:
        return False

    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2

    for _ in range(k):
        a = random.randrange(2, n - 1)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True

def generate_prime(bits: int) -> int:
    """指定ビット数の素数を生成"""
    while True:
        n = random.getrandbits(bits) | (1 << (bits - 1)) | 1
        if miller_rabin(n):
            return n

def extended_gcd(a: int, b: int) -> tuple[int, int, int]:
    """拡張ユークリッド: gcd = a*x + b*y"""
    if a == 0:
        return b, 0, 1
    g, x, y = extended_gcd(b % a, a)
    return g, y - (b // a) * x, x

def mod_inverse(a: int, m: int) -> int:
    """モジュラ逆元: a * x ≡ 1 (mod m)"""
    g, x, _ = extended_gcd(a % m, m)
    if g != 1:
        raise ValueError(f"{a} と {m} は互いに素ではない")
    return x % m

def gcd(a: int, b: int) -> int:
    """ユークリッドの互除法"""
    while b:
        a, b = b, a % b
    return a
```

## 鍵生成の実装

```python
def generate_rsa_keypair(bits: int = 512):
    """RSA鍵ペアの生成

    Args:
        bits: 各素数のビット数。nのビット数は約2倍になる。
              実用では1024以上（n は2048ビット以上）

    Returns:
        公開鍵(n, e), 秘密鍵(n, d), デバッグ用(p, q)
    """
    # Step 1: 2つの大きな素数を生成
    p = generate_prime(bits)
    q = generate_prime(bits)
    while p == q:  # 同じ素数は避ける
        q = generate_prime(bits)

    # Step 2: n = p * q
    n = p * q

    # Step 3: φ(n) = (p-1)(q-1)
    phi_n = (p - 1) * (q - 1)

    # Step 4: 公開指数 e を選ぶ
    # 65537 = 2^16 + 1 が標準。二進表現が 10000000000000001 で
    # 繰り返し二乗法での乗算回数が少ない（高速）
    e = 65537
    if gcd(e, phi_n) != 1:
        # 極めてまれだが、万一の場合は別の e を探す
        e = 3
        while gcd(e, phi_n) != 1:
            e += 2

    # Step 5: 秘密指数 d = e^(-1) mod φ(n)
    d = mod_inverse(e, phi_n)

    return {
        "public_key": (n, e),
        "private_key": (n, d),
        "p": p,
        "q": q,
    }
```

### なぜ e = 65537 なのか

$e = 65537 = 2^{16} + 1$ は、ほぼ全てのRSA実装で使われる標準的な公開指数です。

- **高速性**: 二進表現が `10000000000000001`（1が2つだけ）なので、繰り返し二乗法で17回の二乗と1回の乗算だけで $m^e$ を計算できる
- **安全性**: $e = 3$ でも数学的には機能するが、パディングなしの場合に攻撃に弱い。$65537$ は十分に大きく、既知の攻撃が適用されない

## 暗号化と復号の実装

```python
def rsa_encrypt(message: int, public_key: tuple[int, int]) -> int:
    """RSA暗号化: c = m^e mod n"""
    n, e = public_key
    if message < 0 or message >= n:
        raise ValueError(f"メッセージは 0 以上 {n} 未満でなければならない")
    return pow(message, e, n)

def rsa_decrypt(ciphertext: int, private_key: tuple[int, int]) -> int:
    """RSA復号: m = c^d mod n"""
    n, d = private_key
    return pow(ciphertext, d, n)
```

たったこれだけです。暗号化も復号も、やっていることは**モジュラべき乗**の一発。シンプルですが、このシンプルさの背後にフェルマー、オイラー、ユークリッドの数学が詰まっています。

### 動作確認

```python
# 鍵生成（512ビット素数 → 1024ビットRSA）
keys = generate_rsa_keypair(bits=512)
pub = keys["public_key"]
priv = keys["private_key"]

print(f"n ({keys['public_key'][0].bit_length()} bits)")
print(f"e = {pub[1]}")
print(f"d = {priv[1]}")

# 暗号化・復号
message = 42
ciphertext = rsa_encrypt(message, pub)
decrypted = rsa_decrypt(ciphertext, priv)

print(f"\n平文:   {message}")
print(f"暗号文: {ciphertext}")
print(f"復号:   {decrypted}")
print(f"一致:   {message == decrypted}")  # True
```

## なぜ復号できるのか — 数学的証明

RSAの正しさは、オイラーの定理から直接導かれます。

**主張**: $\gcd(m, n) = 1$ のとき、$m^{ed} \equiv m \pmod{n}$

**証明**:

$ed \equiv 1 \pmod{\phi(n)}$ より、ある整数 $k$ が存在して $ed = 1 + k\phi(n)$。

$$
\begin{aligned}
c^d &= (m^e)^d = m^{ed} \\
&= m^{1 + k\phi(n)} \\
&= m \cdot (m^{\phi(n)})^k \\
&\equiv m \cdot 1^k \quad (\text{オイラーの定理: } m^{\phi(n)} \equiv 1) \\
&= m \pmod{n}
\end{aligned}
$$

$\square$

暗号化して復号すると元に戻る。これがRSAの数学的保証です。

> **補足**: $\gcd(m, n) \neq 1$ の場合（$m$ が $p$ または $q$ の倍数）はオイラーの定理が直接は使えませんが、中国剰余定理を使えばこの場合も $m^{ed} \equiv m \pmod{n}$ が成り立つことを示せます。

## CRT最適化 — 復号を4倍速くする

実用のRSA実装では、復号に**中国剰余定理**（CRT）を使って高速化しています。

### 中国剰余定理とは

$\gcd(m_1, m_2) = 1$ のとき、連立合同式

$$
\begin{cases}
x \equiv a_1 \pmod{m_1} \\
x \equiv a_2 \pmod{m_2}
\end{cases}
$$

は $\bmod\ m_1 m_2$ で一意な解を持ちます。

### RSAへの適用

$c^d \bmod n$ を直接計算する代わりに：

1. $m_p = c^{d \bmod (p-1)} \bmod p$ を計算
2. $m_q = c^{d \bmod (q-1)} \bmod q$ を計算
3. CRTで $m_p$ と $m_q$ から $m \bmod n$ を復元

$n$ の半分のサイズで2回べき乗するので、計算量は約 $1/4$ になります。

```python
def rsa_decrypt_crt(ciphertext: int, p: int, q: int, d: int) -> int:
    """CRTによる高速RSA復号"""
    # 秘密指数を分割
    dp = d % (p - 1)
    dq = d % (q - 1)

    # 半分のサイズでべき乗（ここが速い）
    mp = pow(ciphertext, dp, p)
    mq = pow(ciphertext, dq, q)

    # CRTで結合
    q_inv = mod_inverse(q, p)
    h = (q_inv * (mp - mq)) % p
    m = mq + h * q

    return m

# CRT復号の検証
crt_decrypted = rsa_decrypt_crt(ciphertext, keys["p"], keys["q"], priv[1])
print(f"CRT復号: {crt_decrypted}")
print(f"通常復号と一致: {crt_decrypted == decrypted}")  # True
```

OpenSSL などの実装では、このCRT最適化が標準で使われています。

## 文字列の暗号化

数値だけでなく、文字列（テキスト）も暗号化してみましょう。

```python
def encrypt_message(text: str, public_key: tuple[int, int]) -> list[int]:
    """文字列をRSA暗号化（文字ごとに暗号化する簡易版）"""
    return [rsa_encrypt(ord(ch), public_key) for ch in text]

def decrypt_message(ciphertexts: list[int], private_key: tuple[int, int]) -> str:
    """RSA暗号文を復号して文字列に戻す"""
    return "".join(chr(rsa_decrypt(c, private_key)) for c in ciphertexts)

# テスト
plaintext = "Hello, RSA!"
encrypted = encrypt_message(plaintext, pub)
print(f"平文:   {plaintext}")
print(f"暗号文: {encrypted[:3]}...（{len(encrypted)} 個の巨大整数）")

restored = decrypt_message(encrypted, priv)
print(f"復号:   {restored}")
print(f"一致:   {plaintext == restored}")  # True
```

> **注意**: 文字ごとの暗号化は教育用です。実用では、メッセージ全体を1つの大きな整数として扱い、OAEP パディングを施してから暗号化します。文字ごとに暗号化すると、頻度分析で解読される可能性があります。

## 実用RSAとの差

この実装は「RSAの数学的本質」を完全に含んでいますが、実用のRSA暗号ではさらに以下の対策が必要です。

| 項目 | この実装 | 実用RSA |
|:--|:--|:--|
| 鍵長 | 1024ビット | 2048ビット以上 |
| パディング | なし | OAEP（PKCS#1 v2.2） |
| 乱数生成 | `random` | `secrets` / `/dev/urandom` |
| 素数検証 | ミラー・ラビンのみ | ミラー・ラビン + 追加検証 |
| 鍵の保管 | 変数 | PKCS#8 / HSM |

パディングの欠如は特に危険です。パディングなしのRSA（textbook RSA）は、同じメッセージに対して常に同じ暗号文を生成するため、選択暗号文攻撃に脆弱です。

## 全コードまとめ

ここまでの実装を1つにまとめたものを掲載します。

```python
"""RSA暗号のフルスクラッチ実装（教育用）"""
import random
import time

# --- 素数生成 (#1) ---

def miller_rabin(n: int, k: int = 20) -> bool:
    if n < 2: return False
    if n == 2 or n == 3: return True
    if n % 2 == 0: return False
    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1; d //= 2
    for _ in range(k):
        a = random.randrange(2, n - 1)
        x = pow(a, d, n)
        if x == 1 or x == n - 1: continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1: break
        else: return False
    return True

def generate_prime(bits: int) -> int:
    while True:
        n = random.getrandbits(bits) | (1 << (bits - 1)) | 1
        if miller_rabin(n): return n

# --- モジュラ演算 (#2) ---

def gcd(a, b):
    while b: a, b = b, a % b
    return a

def extended_gcd(a, b):
    if a == 0: return b, 0, 1
    g, x, y = extended_gcd(b % a, a)
    return g, y - (b // a) * x, x

def mod_inverse(a, m):
    g, x, _ = extended_gcd(a % m, m)
    if g != 1: raise ValueError("逆元なし")
    return x % m

# --- RSA (#3) ---

def generate_keypair(bits=512):
    p, q = generate_prime(bits), generate_prime(bits)
    while p == q: q = generate_prime(bits)
    n, phi = p * q, (p-1) * (q-1)
    e = 65537
    d = mod_inverse(e, phi)
    return (n, e), (n, d), p, q

def encrypt(m, pub): return pow(m, pub[1], pub[0])
def decrypt(c, priv): return pow(c, priv[1], priv[0])

# --- デモ ---

if __name__ == "__main__":
    start = time.time()
    pub, priv, p, q = generate_keypair(512)
    print(f"鍵生成: {time.time() - start:.3f}秒")
    print(f"鍵長: {pub[0].bit_length()} bits\n")

    msg = 12345678901234567890
    c = encrypt(msg, pub)
    m = decrypt(c, priv)
    print(f"平文:   {msg}")
    print(f"暗号文: {c}")
    print(f"復号:   {m}")
    print(f"成功:   {msg == m}")
```

## まとめ

今回実装した内容：

1. **鍵生成**: 素数生成 → $n, \phi(n)$ の計算 → $e$ の選択 → $d$ の計算（モジュラ逆元）
2. **暗号化**: $c = m^e \bmod n$（モジュラべき乗の一発）
3. **復号**: $m = c^d \bmod n$（同じくモジュラべき乗）
4. **正しさの証明**: オイラーの定理 $m^{\phi(n)} \equiv 1$ から $m^{ed} \equiv m$ が従う
5. **CRT最適化**: 復号を約4倍高速化

3回かけて、素数→モジュラ演算→実装と積み上げてきました。最終回の次回は、**RSAの弱点と、それを補う楕円曲線暗号**を扱います。量子コンピュータの脅威にも触れます。
