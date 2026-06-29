# CSE107 Lecture 13: Digital Signatures — 심화 세미나 학습 문서

---

## 목차

1. [디지털 서명의 동기와 직관적 이해](#1-디지털-서명의-동기와-직관적-이해)
2. [디지털 서명 스킴의 정형적 정의](#2-디지털-서명-스킴의-정형적-정의)
3. [UF-CMA 보안 모델의 수학적 구조](#3-uf-cma-보안-모델의-수학적-구조)
4. [서명 위조의 분류 체계](#4-서명-위조의-분류-체계)
5. [Plain RSA 서명과 그 취약점](#5-plain-rsa-서명과-그-취약점)
6. [해시 함수의 도입: RSA PKCS#1과 한계](#6-해시-함수의-도입-rsa-pkcs1과-한계)
7. [Full Domain Hash (FDH): 안전성 증명 가능한 구조](#7-full-domain-hash-fdh-안전성-증명-가능한-구조)
8. [RSA PSS: 확률론적 서명 스킴](#8-rsa-pss-확률론적-서명-스킴)
9. [이산 로그 기반 서명 스킴](#9-이산-로그-기반-서명-스킴)
10. [서명에서의 무작위성과 해싱의 역할](#10-서명에서의-무작위성과-해싱의-역할)

---

## 1. 디지털 서명의 동기와 직관적 이해

### 1.1 손으로 쓴 서명의 문제점

물리 세계에서 서명은 문서, 수표 등에 서명자의 동의를 증명하기 위해 사용된다. 그러나 이를 디지털화하면 (예: 스캔하거나 태블릿으로 그리면) 서명 이미지가 `mysig.png`와 같은 비트스트링으로 변환된다.

**핵심 문제:** 디지털화된 손글씨 서명은 복사가 너무 쉽다. 공격자가 서명 이미지를 그대로 잘라내어 다른 문서에 붙여넣으면 위조가 완성된다.

**결론:** 안전한 디지털 서명은 서명자뿐만 아니라, **서명 대상 메시지에도 의존**해야 한다. 같은 키 쌍을 써도 메시지가 다르면 서명이 달라야 한다. (p. 1)

### 1.2 디지털 서명의 작동 원리 (고수준 뷰)

```
[Alice]
  Step 1: (vk, sk) ← K  → sk 보관, vk 공개
  Step 3: σ ← S_sk(M)   → 메시지 M에 sk로 서명

[Anyone]
  Step 4: V_vk(M, σ) = 1? → Alice의 서명인지 검증
```

- **비밀키 (sk):** Alice만 보유. 서명 생성에 사용.
- **검증키 (vk):** 누구나 접근 가능. 서명 검증에 사용.
- **authenticity 요구:** vk가 정말 Alice의 키임을 보장해야 함. 이를 위해 인증서(certificate) 체계를 사용하며, 이는 후속 강의에서 다룬다. (p. 3)

### 1.3 서명 vs. MAC: 본질적 차이

| 특성 | MAC | 디지털 서명 |
|------|-----|-----------|
| 키 구조 | 단일 대칭키 $K$ | 키 쌍 $(sk, vk)$ |
| 검증 주체 | $K$를 가진 자만 가능 | $vk$를 가진 누구나 |
| 위조 가능성 | 검증자도 위조 가능 | 검증자는 위조 불가 |
| 서버 침해 시 | 위조 가능 | 위조 불가 |

**시나리오 분석 (MACs):** Alice가 은행과 공유키 $K$로 수표에 MAC을 생성한다면, 은행 서버가 해킹되어 $K$가 유출될 경우 공격자는 Alice의 수표를 위조할 수 있다.

**서명의 우위:** 은행이 Alice의 $vk$만 보유한다면, $vk$ 유출로는 Alice의 서명을 위조할 수 없다. 서명 생성은 오직 $sk$ 보유자(Alice)만 할 수 있기 때문이다. (p. 4)

---

## 2. 디지털 서명 스킴의 정형적 정의

### 2.1 구문 (Syntax)

디지털 서명 스킴 $\mathcal{DS} = (\mathcal{K}, \mathcal{S}, \mathcal{V})$는 세 가지 알고리즘으로 구성된다.

**키 생성 알고리즘 $\mathcal{K}$:**

$$
(vk, sk) \leftarrow^{\$} \mathcal{K}
$$

무작위성을 사용하여 검증키 $vk$와 서명키 $sk$를 생성한다. $\leftarrow^{\$}$는 알고리즘이 확률적(randomized)임을 표기하는 표준 암호학 notation이다.

**서명 알고리즘 $\mathcal{S}$:**

$$
\sigma \leftarrow^{\$} \mathcal{S}_{sk}(M)
$$

서명키 $sk$와 메시지 $M \in \{0,1\}^*$를 입력받아 서명 $\sigma$를 출력한다. 알고리즘이 확률적일 수 있음에 유의.

**검증 알고리즘 $\mathcal{V}$:**

$$
d \leftarrow \mathcal{V}_{vk}(M, \sigma), \quad d \in \{0, 1\}
$$

검증키 $vk$, 메시지 $M$, 서명 후보 $\sigma$를 받아 0(거부) 또는 1(수락)을 출력한다.

### 2.2 정확성 (Correctness) 요건

모든 $\mathcal{K}$의 출력 $(vk, sk)$와 메시지 공간의 임의 $M$에 대해:

$$
\Pr\left[\mathcal{V}_{vk}(M, \mathcal{S}_{sk}(M)) = 1\right] = 1
$$

즉, 합법적으로 생성된 서명은 항상 검증을 통과해야 한다. 메시지 공간은 검증키 $vk$에 의존할 수 있다. (p. 2)

---

## 3. UF-CMA 보안 모델의 수학적 구조

### 3.1 보안의 직관적 의미

**UF-CMA (Unforgeability under Chosen Message Attack):**

공격자가 Alice에게 자신이 원하는 임의의 메시지들에 대한 서명을 얻을 수 있더라도 (Chosen Message Attack), Alice가 이전에 서명한 적 없는 **새로운** 메시지에 대한 유효 서명을 만들어낼 수 없어야 한다.

**MACs의 UF-CMA와의 차이점:** 서명 스킴의 공격자는 게임 시작 시 **검증키 $vk$를 부여받는다.** 이는 공개키 암호학의 핵심 특성 — 검증키는 공개되어 있으므로 공격자도 당연히 알 수 있다.

### 3.2 UF-CMA 게임의 수학적 정의

```
Game UF-CMA_DS

procedure Initialize:
    (vk, sk) ← K$
    S ← ∅           // 서명된 메시지 집합
    return vk        // 공격자에게 검증키 공개

procedure Sign(M):   // 공격자의 서명 요청
    σ ← S$_sk(M)
    S ← S ∪ {M}
    return σ

procedure Finalize(M, σ):  // 공격자의 위조 시도
    d ← V_vk(M, σ)
    return (d = 1 ∧ M ∉ S)  // 유효한 서명이고, 새 메시지여야 승리
```

**공격자 $A$의 UF-CMA 이점 (advantage):**

$$
\text{Adv}^{\text{uf-cma}}_{\mathcal{DS}}(A) = \Pr\left[\text{UF-CMA}^A_{\mathcal{DS}} \Rightarrow \text{true}\right]
$$

### 3.3 게임 해석: 각 프로시저의 암호학적 의미

| 프로시저 | 암호학적 의미 | 공격자에게 주는 정보 |
|--------|------------|-----------------|
| `Initialize` | 키 생성 완료, 검증키 공개 | $vk$ (공개키) |
| `Sign(M)` | 서명자 역할 시뮬레이션 | $\sigma = \mathcal{S}_{sk}(M)$ |
| `Finalize(M, σ)` | 검증자 역할 시뮬레이션 | 위조 성공 여부 |

**승리 조건 분석:**
- **Correct:** $\mathcal{V}_{vk}(M, \sigma) = 1$ — 검증자가 수락해야 함
- **New:** $M \notin S$ — 공격자가 직접 서명 요청한 메시지가 아니어야 함

**보안의 의미:** `Sign`은 서명자를, `Finalize`는 검증자를 표현한다. 보안은 공격자가 서명자가 이전에 서명하지 않은 메시지를 검증자가 수락하도록 만들 수 없음을 의미한다. (p. 5–7)

**Replay 공격에 대한 주의:** UF-CMA는 replay 공격(이미 획득한 유효 서명 재사용)에 대한 보안을 요구하지 않는다. 이는 카운터(counter)나 타임스탬프(timestamp)를 상위 레이어에서 추가하여 처리한다.

---

## 4. 서명 위조의 분류 체계

암호학에서는 공격자의 능력에 따라 위조를 세 단계로 구분한다.

| 위조 유형 | 정의 | 공격 난이도 |
|---------|-----|-----------|
| **Existential Forgery (존재적 위조)** | 공격자가 $(M, \sigma)$ 쌍을 자유롭게 선택 | 가장 쉬움 |
| **Selective Forgery (선택적 위조)** | 공격자가 오라클 호출 전에 $M$을 미리 선택 | 중간 |
| **Universal Forgery (보편적 위조)** | 공격자가 임의의 메시지에 서명 가능 | 가장 어려움 |

**UF-CMA의 철학:** 가장 쉬운 목표인 existential forgery조차 공격자가 달성하기 어려워야 한다. 즉, 서명 스킴은 **공격자에게 가장 유리한 조건** 하에서도 안전해야 한다. (p. 8)

---

## 5. Plain RSA 서명과 그 취약점

### 5.1 Plain RSA 서명 스킴 정의

RSA 생성기 $\mathcal{K}_{\text{rsa}}$를 기반으로 한 Plain RSA 서명 스킴:

$$
\text{KeyGen}: (N, p, q, e, d) \leftarrow^{\$} \mathcal{K}_{\text{rsa}}
$$

$$
vk = (N, e), \quad sk = (N, d)
$$

| 알고리즘 | 입력 | 출력 | 연산 |
|---------|-----|-----|-----|
| $\mathcal{S}_{(N,d)}(y)$ | 메시지 $y \in \mathbb{Z}^*_N$ | 서명 $x$ | $x \leftarrow y^d \bmod N$ |
| $\mathcal{V}_{(N,e)}(y, x)$ | 메시지 $y$, 서명 $x$ | $d \in \{0,1\}$ | $x^e \bmod N \stackrel{?}{=} y$ |

**정확성 증명:**

$$
x^e \bmod N = (y^d)^e \bmod N = y^{ed \bmod \phi(N)} = y^1 = y
$$

여기서 $ed \equiv 1 \pmod{\phi(N)}$은 RSA 키 생성의 기본 성질이다. (p. 9)

### 5.2 RSA의 대수학적 배경 (보충)

**오일러의 정리:** $\gcd(a, N) = 1$이면,

$$
a^{\phi(N)} \equiv 1 \pmod{N}
$$

$N = pq$ ($p, q$는 서로 다른 소수)일 때, $\phi(N) = (p-1)(q-1)$.

**RSA 키 생성의 대수적 구조:**
- $e$를 선택: $\gcd(e, \phi(N)) = 1$ (보통 $e = 65537 = 2^{16}+1$)
- $d$를 계산: $d \equiv e^{-1} \pmod{\phi(N)}$ (확장 유클리드 알고리즘으로 계산)
- 따라서: $ed = 1 + k\phi(N)$ for some integer $k$

**구체적 예시 (작은 수):**
- $p = 11, q = 13 \Rightarrow N = 143, \phi(N) = 120$
- $e = 7$ ($\gcd(7, 120) = 1$)
- $d \equiv 7^{-1} \pmod{120}$: $7 \times 103 = 721 = 6 \times 120 + 1$, 즉 $d = 103$
- 메시지 $y = 2$: 서명 $x = 2^{103} \bmod 143 = 6$
- 검증: $6^7 \bmod 143 = 279936 \bmod 143 = 2 = y$ ✓

### 5.3 Plain RSA에 대한 공격 세 가지

**공격 1: 자명한 위조 (Trivial Forgery)**

```
Adversary A((N, e)):
    return (1, 1)
```

$\text{Adv}^{\text{uf-cma}}_{\mathcal{DS}}(A) = 1$. 왜냐하면 $1^d \bmod N = 1$이므로 $(y=1, x=1)$은 항상 유효한 서명 쌍이다.

**공격 2: 곱셈 준동형성(Multiplicative Homomorphism) 이용**

```
Adversary A((N, e)):
    y1, y2 ← Z*_N \ {1}  // 서로 다른 두 원소 선택
    x1 ← Sign(y1)         // y1에 대한 서명 획득
    x2 ← Sign(y2)         // y2에 대한 서명 획득
    return (y1·y2 mod N, x1·x2 mod N)
```

**성공 이유:**

$$
(y_1 y_2)^d \bmod N = y_1^d \cdot y_2^d \bmod N = x_1 \cdot x_2 \bmod N
$$

RSA의 곱셈 준동형성 — $f^{-1}(y_1 y_2) = f^{-1}(y_1) \cdot f^{-1}(y_2)$ — 이 치명적 취약점으로 작용한다.

**공격 3: Inversion Attack (역방향 위조)**

```
Adversary A((N, e)):
    x ←$ Z*_N         // 임의의 서명값 선택
    y ← x^e mod N    // 대응하는 "메시지" 계산
    return (y, x)
```

**성공 이유:**

$$
y^d \bmod N = (x^e)^d \bmod N = x^{ed \bmod \phi(N)} = x
$$

공격자는 유효한 서명 쌍을 의미 없는(garbage) 메시지에 대해서도 만들 수 있다. 이 공격이 결정적인 이유는 공격자가 서명 오라클을 전혀 호출하지 않고도 $\text{Adv} = 1$을 달성하기 때문이다. (p. 10–12)

**근본 원인 분석:** Plain RSA의 메시지 공간 $\mathbb{Z}^*_N$은 RSA 함수 $f(x) = x^e \bmod N$에 대해 닫혀 있으며 곱셈 구조를 가진다. 서명 알고리즘이 이 대수적 구조를 그대로 노출한다.

### 5.4 1-wayness만으로는 위조 방지가 부족한 이유

**OW 게임의 목표:** 랜덤 $y$가 주어질 때 $y^d \bmod N$ 계산.

**UF-CMA 공격자의 목표:** $(N, e)$가 주어질 때, 자신이 원하는 구조의 $y$에 대해 $y^d \bmod N$ 계산.

핵심 차이: OW 게임은 $y$가 **균등 랜덤**임을 보장하지만, UF-CMA 공격자는 $y$를 자유롭게 선택할 수 있다. 1-wayness는 랜덤 입력에 대한 역함수 계산 어려움을 보장할 뿐, 특수한 구조를 가진 입력에 대해서는 아무것도 보장하지 않는다. (p. 15–17)

---

## 6. 해시 함수의 도입: RSA PKCS#1과 한계

### 6.1 PKCS#1 서명 구조

Plain RSA의 문제를 완화하기 위해 메시지를 특정 형태로 인코딩한다.

- $|N| = 1024$ bits, $n = 128$ bytes
- 해시 함수 $h: \{0,1\}^* \to \{0,1\}^{160}$ (SHA-1 계열)

**PKCS#1 해시 패딩 함수:**

$$
H_{\text{PKCS}}(M) = \underbrace{00 \| 01}_{\text{header}} \| \underbrace{FF \| \cdots \| FF}_{n-22 \text{ bytes}} \| \underbrace{h(M)}_{20 \text{ bytes}}
$$

**서명:**

$$
\mathcal{S}_{N,d}(M) = H_{\text{PKCS}}(M)^d \bmod N
$$

**아이디어:** $\mathbb{Z}^*_N$의 원소를 매우 특정한 형태로 강제하여 이전 공격들을 방지한다. 이 형태를 만족하지 않는 서명은 거부된다. (p. 14)

### 6.2 PKCS#1의 구조적 한계

그러나 $H_{\text{PKCS}}(M)$의 앞 $n - 20 = 108$ 바이트가 **고정**되어 있다. 전체 $n = 128$ 바이트 중 단 20바이트(해시값)만이 실제로 메시지에 따라 변한다.

**정리 (안전성 증명 불가능성):** $H_{\text{PKCS}}$ 구조에서는, RSA가 1-way라는 가정(과 해시 함수에 대한 어떠한 가정)만으로는 RSA PKCS#1 서명의 UF-CMA 안전성을 증명할 수 없다. 이런 증명 자체가 존재하지 않는다.

**이유:** 앞 108바이트가 고정되어 있으므로, $H_{\text{PKCS}}(M)$은 결코 $\mathbb{Z}^*_N$ 위에서 랜덤하게 분포하지 않는다. 따라서 OW 가정(랜덤 입력 전제)으로 환원하는 안전성 감소(security reduction)를 구성할 수 없다. (p. 18)

---

## 7. Full Domain Hash (FDH): 안전성 증명 가능한 구조

### 7.1 FDH의 정의 [Bellare-Rogaway 1996]

$H: \{0,1\}^* \to \mathbb{Z}^*_N$인 해시 함수를 사용하는 RSA FDH 서명 스킴:

| 알고리즘 | 연산 |
|---------|-----|
| $\mathcal{K}$ | $(N,p,q,e,d) \leftarrow^{\$} \mathcal{K}_{\text{rsa}}$, $vk=(N,e)$, $sk=(N,d)$ |
| $\mathcal{S}_{(N,d)}(M)$ | $y \leftarrow H(M)$; $x \leftarrow y^d \bmod N$; return $x$ |
| $\mathcal{V}_{(N,e)}(M, x)$ | $y \leftarrow H(M)$; return $(x^e \bmod N \stackrel{?}{=} y)$ |

메시지 공간은 $\{0,1\}^*$, 서명은 $x \in \mathbb{Z}^*_N$.

**정확성:**

$$
x^e \bmod N = (y^d)^e \bmod N = y^{ed \bmod \phi(N)} = y = H(M) \quad \checkmark
$$

(p. 20)

### 7.2 FDH에서 충돌 저항성의 필요성

**정리:** $H$에 대한 충돌을 찾는 공격자 $C$가 존재하면 RSA FDH는 UF-CMA 안전하지 않다.

**귀납적 공격 구성:**

```
Adversary A((N, e)):
    (M1, M2) ←$ C       // 충돌 쌍: M1 ≠ M2, H(M1) = H(M2)
    x1 ← Sign(M1)       // M1의 서명 획득
    return (M2, x1)     // M2에 대해 x1을 위조 서명으로 제출
```

**성공 이유:**

$$
H(M_1) = H(M_2) \implies \mathcal{S}_{(N,d)}(M_1) = H(M_1)^d \bmod N = H(M_2)^d \bmod N = \mathcal{S}_{(N,d)}(M_2)
$$

따라서 $x_1$은 $M_2$의 유효 서명이며, $M_2 \notin S$이므로 공격 성공.

**중요한 뉘앙스:** 충돌 저항성은 UF-CMA를 위해 **필요조건**이지만 **충분조건**이 아니다. (p. 21)

### 7.3 FDH에서 H의 구체적 구현

$H: \{0,1\}^* \to \mathbb{Z}^*_N$은 가변 출력 길이 함수(XOF)를 기반으로 구현한다.

**XOF (eXtendable Output Function) $G$:**

$$
G(W, \ell) = \text{SHAKE256}(W, \ell)
$$

또는:

$$
G(W, \ell) = \text{SHA256}(\langle 0 \rangle \| W) \| \text{SHA256}(\langle 1 \rangle \| W) \| \cdots \| \text{SHA256}(\langle 2^8-1 \rangle \| W)
$$

$(\ell \leq 2^8 \cdot 256$인 경우)

**$H$의 구체적 구성:** $k$가 RSA 생성기의 보안 파라미터이고 $2^{k-1} < N < 2^k$일 때:

$$
H(M) = G(M, 2k) \bmod N
$$

**$\mathbb{Z}_N$ vs $\mathbb{Z}^*_N$ 이슈:**
- $H(M) \in \mathbb{Z}_N$이지만 필요한 것은 $H(M) \in \mathbb{Z}^*_N$
- $N = pq$에서 $y \in \mathbb{Z}_N \setminus \mathbb{Z}^*_N$ (즉 $y$가 $p$ 또는 $q$의 배수)이면 $N$을 인수분해할 수 있음
- 그런 $y$의 확률은 $\frac{p+q-1}{N} \approx \frac{2}{\sqrt{N}}$으로 무시할 수 있을 만큼 작음
- 따라서 실용적으로 $H(M) \in \mathbb{Z}_N$으로도 충분하며 정확성과 안전성이 유지됨 (p. 22–23)

### 7.4 Random Oracle 모델에서의 FDH 안전성

**정리 (Bellare-Rogaway 1996):** $H$를 Random Oracle로 모델링할 때, RSA FDH의 UF-CMA 안전성은 RSA의 1-wayness로 환원된다.

$$
\text{Adv}^{\text{uf-cma}}_{\text{FDH}}(A) \leq q_H \cdot \text{Adv}^{\text{ow}}_{\text{RSA}}(B)
$$

여기서 $q_H$는 공격자 $A$의 해시 쿼리 횟수이다. 이 결합 인수(tight factor) $q_H$는 FDH의 단점이었으며, 이를 개선한 것이 PSS이다.

---

## 8. RSA PSS: 확률론적 서명 스킴

### 8.1 PSS 파라미터 설정

- RSA 생성기 $\mathcal{K}_{\text{rsa}}$, $k$-bit 공개키
- 파라미터 $\ell$: $17 \leq 2\ell + 1 < k$ (예: $k = 2048, \ell = 256$)
- 세 가지 해시 함수:
  - $H_1, H_2: \{0,1\}^* \to \{0,1\}^\ell$
  - $H_3: \{0,1\}^* \to \{0,1\}^{k-2\ell-1}$

키 생성: $vk = (N, e)$, $sk = (N, d)$ (표준 RSA 방식)

### 8.2 PSS 서명 알고리즘 상세 해설

```
Alg S_(N,d)(M):
    r ←$ {0,1}^ℓ          // 랜덤 솔트 생성
    w ← H1(M ∥ r)          // 메시지와 솔트의 압축 해시
    r* ← H2(w) ⊕ r         // 솔트를 마스킹하여 저장
    y ← 0 ∥ w ∥ r* ∥ H3(w) // PSS 인코딩 구성
    x ← y^d mod N          // RSA 역함수 적용
    return x
```

**PSS 인코딩 $y$의 비트 구조:**

```
bit k-1      k-2 ~ k-1-ℓ    k-1-ℓ ~ k-1-2ℓ    k-1-2ℓ ~ 0
[  0  ] [    w (ℓ bits)   ] [  r* (ℓ bits)   ] [ H3(w) (k-2ℓ-1 bits) ]
```

- 첫 비트 `0`: $y < 2^{k-1} < N$을 보장하여 $\bmod N$ 연산이 trivial하지 않음을 보장
- $w = H_1(M \| r)$: 메시지와 솔트를 결합한 해시
- $r^* = H_2(w) \oplus r$: 솔트를 직접 저장하지 않고 마스킹
- $H_3(w)$: 무결성 체크용 패리티 정보

### 8.3 PSS 검증 알고리즘 상세 해설

```
Alg V_(N,e)(M, x):
    y ← x^e mod N              // RSA 정방향 함수 적용
    b ∥ w ∥ r* ∥ P ← y         // 인코딩 파싱
    r ← r* ⊕ H2(w)             // 솔트 복원
    if H3(w) ≠ P: return 0     // 패리티 체크
    if b = 1: return 0         // 첫 비트 체크
    if H1(M ∥ r) ≠ w: return 0 // 메시지-솔트-해시 일관성 체크
    return 1
```

**검증의 단계별 대수적 의미:**

1. **$y \leftarrow x^e \bmod N$:** RSA 정방향 함수 — $x$가 진짜 $y^d$이면 $y$가 복원됨
2. **파싱:** PSS 인코딩 구조에 따라 $w, r^*, P$ 추출
3. **솔트 복원:** $r = r^* \oplus H_2(w) = (H_2(w) \oplus r) \oplus H_2(w) = r$ — XOR 자기 역원성 이용
4. **세 가지 검사:**
   - $H_3(w) = P$: 패리티 정보가 일치하는가
   - $b = 0$: 첫 비트가 0인가 (도메인 분리)
   - $H_1(M \| r) = w$: 이 $M$과 복원된 $r$이 $w$를 만드는가 (핵심 검증)

(p. 24–25)

### 8.4 PSS가 FDH보다 우수한 이유

**Tight Security Reduction:** PSS는 FDH의 $q_H$ 인수를 제거한 tight reduction을 가진다.

$$
\text{Adv}^{\text{uf-cma}}_{\text{PSS}}(A) \leq \text{Adv}^{\text{ow}}_{\text{RSA}}(B)
$$

**실용적 의미:** 같은 보안 수준을 달성하는 데 더 짧은 RSA 키를 사용할 수 있다.

**무작위 솔트 $r$의 역할:**
- 같은 메시지를 여러 번 서명해도 매번 다른 서명이 생성됨
- 공격자가 서명을 관찰해도 메시지에 대한 정보를 얻기 어려움
- 특정 구조의 입력을 RSA에 제출하는 것을 막음

### 8.5 PSS 표준화 현황

PSS는 광범위하게 표준화되어 있다:
- RSA PKCS#1 v2.1, v2.2 / RFC 8017
- IEEE P1363a
- ANSI X9.31
- RFC 3447
- ISO/IEC 9796-2

(p. 26)

---

## 9. 이산 로그 기반 서명 스킴

### 9.1 Schnorr 서명 스킴

**수학적 설정:**
- $\mathbb{G} = \langle g \rangle$: 소수 위수 $m$을 가진 순환 군 (예: 타원 곡선 군)
- $H: \{0,1\}^* \to \mathbb{Z}_m$: 해시 함수 (Random Oracle로 모델링)

**키 쌍:**

$$
sk = x \leftarrow^{\$} \mathbb{Z}_m, \quad vk = X = g^x
$$

이산 로그 문제(DLP)의 어려움: $X = g^x$에서 $x$를 구하는 것이 계산적으로 어렵다는 가정을 기반으로 한다.

#### Schnorr 서명 알고리즘

```
Alg S^H_x(M):
    r ←$ Z_m               // 랜덤 nonce
    R ← g^r                // 커밋먼트
    c ← H(R ∥ M)           // 챌린지
    s ← (r + cx) mod m     // 응답
    return (R, s)
```

#### Schnorr 검증 알고리즘

```
Alg V^H_X(M, (R, s)):
    if R ∉ G: return 0     // 유효한 군 원소 확인
    c ← H(R ∥ M)           // 챌린지 재계산
    if g^s = R · X^c: return 1
    else: return 0
```

**정확성 검증:**

$$
g^s = g^{r+cx} = g^r \cdot (g^x)^c = R \cdot X^c \quad \checkmark
$$

(p. 27)

#### Schnorr의 Sigma 프로토콜 구조

Schnorr 서명은 이산 로그에 대한 지식 증명(proof of knowledge) 프로토콜 — Sigma 프로토콜 — 을 비대화식(non-interactive)으로 변환한 것이다 (Fiat-Shamir 변환).

```
[Prover (Alice)]               [Verifier (Bob)]
    r ←$ Z_m
    R = g^r
                    R →
                               c ←$ Z_m
                    ← c
    s = r + cx mod m
                    s →
                               Check: g^s = R·X^c ?
```

Fiat-Shamir 변환: 챌린지 $c$를 검증자 대신 해시 함수로 생성하면 비대화식 서명이 된다. $c = H(R \| M)$.

### 9.2 EdDSA: 실용적 Schnorr 변형

EdDSA [Bernstein-Duif-Lange-Schwabe-Yang 2012]는 타원 곡선 군(Edwards curve)에서의 Schnorr 기반 서명이다.

**핵심 특징:**

1. **서명키 확장:** 랜덤 $b$-비트 문자열 $sk$를 2$b$-비트 문자열 $x_1 \| x_2$로 해시하여 확장
2. **클램핑 함수 (Clamping):** $x_1$에 비트 조작을 적용하여 Schnorr 서명키 $x \in \mathbb{Z}_m$ 생성
3. **결정론적 서명:** nonce $r$을 $x_2$와 메시지의 해시로 생성 → 무작위성 없이 결정론적 서명 달성

**타원 곡선 선택:** Curve25519 (Edwards form: Ed25519) 또는 Ed448

**표준화:** RFC 8032, FIPS 186-5 포함. OpenSSH, GnuPG, TLS 1.3 등에서 광범위하게 사용. (p. 28)

### 9.3 DSA / ECDSA

**DSA (Digital Signature Algorithm):**
- NIST 1991 제안, FIPS 186-4 표준
- $\mathbb{Z}^*_p$ ($p$: 소수)에서 El Gamal 서명의 변형
- FIPS 186-5 초안에서는 새 승인 여부가 불확실

**ECDSA (Elliptic Curve DSA):**
- 타원 곡선 군에서의 DSA 변형
- FIPS 186-4 표준
- Bitcoin, Ethereum 등 블록체인에서 핵심 서명 알고리즘으로 사용

**비교: Schnorr vs ECDSA**

| 특성 | Schnorr/EdDSA | ECDSA |
|-----|--------------|-------|
| 서명 크기 | $2 \times 32$ bytes (Ed25519) | $2 \times 32$ bytes |
| 키 집약 (key aggregation) | 가능 (MuSig 등) | 불가 |
| Tight security reduction | 존재 | 존재하지 않음 |
| 결정론적 서명 | EdDSA는 기본 제공 | RFC 6979로 별도 구현 필요 |
| 특허 문제 | 없음 | 과거 있었음 (현재 만료) |

(p. 29)

---

## 10. 서명에서의 무작위성과 해싱의 역할

### 10.1 무작위성: 필요한가?

**핵심 결론:** 암호화(encryption)와 달리, 서명에서 무작위성은 **보안을 위해 필수적이지 않다.** RSA FDH가 안전한 결정론적 서명의 예다.

그러나 무작위성 사용 시 주의점:

| 상황 | 결과 |
|-----|-----|
| nonce 재사용 (같은 $r$ 두 번 사용) | **치명적** |
| 불량 난수 생성기 | **치명적** |
| 결정론적 서명 (nonce = 해시) | 안전 |

**nonce 재사용의 치명적 결과 (Schnorr/DSA에서):**

두 메시지 $M_1, M_2$에 같은 $r$ (따라서 같은 $R = g^r$)을 사용하면:

$$
s_1 = r + c_1 x \bmod m
$$
$$
s_2 = r + c_2 x \bmod m
$$

$$
s_1 - s_2 = (c_1 - c_2) x \bmod m \implies x = \frac{s_1 - s_2}{c_1 - c_2} \bmod m
$$

서명키 $x$가 직접 복구된다! **PlayStation 3 해킹(2010)**이 이 취약점을 정확히 이용했다.

(p. 30)

### 10.2 해싱: 필수적인 도구

**주요 결론:** 무작위성은 필수가 아니지만, 해시 함수는 서명 보안에 핵심적인 역할을 한다.

**Case Study 1: 짧은 메시지 서명 (RSA 변형)**

- $sk = (N,d)$, $vk = (N,e)$, $N$이 $k$-bit
- 메시지: $0 \leq M < 2^{k-128}$ ($k-128$비트 정수)
- 서명: $M^d \bmod N$
- 검증: $\sigma^e \bmod N < 2^{k-128}$ 확인

**취약점:** Plain RSA와 동일한 공격이 작동한다.
- $(1, 1)$은 유효한 서명: $1^d \bmod N = 1 < 2^{k-128}$ ✓
- 두 짧은 메시지의 곱도 짧을 수 있으므로 같은 곱셈 공격 가능 (p. 32)

**Case Study 2: CRC를 이용한 RSA 변형**

- 메시지: $0 \leq M < 2^{k-144}$
- 서명: $(\text{CRC}_{16}(M) \times 2^{k-1-16} + M)^d \bmod N$
- 검증: 상위 16비트가 하위 $k-144$비트의 CRC인지, 중간 128비트가 모두 0인지 확인

**취약점 분석:**
1. CRC는 암호학적 해시가 아님 — 선형 함수이므로 $\text{CRC}(M_1 \oplus M_2) = \text{CRC}(M_1) \oplus \text{CRC}(M_2)$
2. 16비트는 너무 짧음 — $2^{16} = 65536$개의 CRC 값만 존재
3. CRC = 0인 짧은 메시지를 쉽게 찾을 수 있고, 그런 메시지의 곱도 CRC = 0
4. 따라서 곱셈 공격이 동작 가능 (p. 33)

**결론:** 서명에서 "검사" 역할을 하는 함수는 반드시 **암호학적 해시 함수의 성질** (특히 충돌 저항성과 역상(preimage) 저항성)을 가져야 한다.

### 10.3 결정론적 서명 만들기

무작위성이 필요한 서명 스킴(Schnorr, PSS 등)을 안전하게 결정론적으로 변환하는 방법:

$$
r = \text{PRF}(sk, M)
$$

즉, 서명키와 메시지의 의사랜덤 함수(PRF) 값으로 nonce를 결정한다. 이렇게 하면:
- 같은 $(sk, M)$ 쌍에 대해 항상 같은 $r$ 생성 → 결정론적
- $r$이 외부에서 관찰 불가 → 랜덤처럼 동작
- 안전성 손실 없음

**실제 표준화:**
- EdDSA: $r = H(x_2 \| M)$ 형태로 기본 내장
- RFC 6979: ECDSA/DSA를 결정론적으로 만드는 표준

### 10.4 서명 스킴의 전체 분류 (Universe)

다양한 특수 목적 서명 스킴들이 존재한다 (p. 34):

| 카테고리 | 예시 스킴들 | 주요 활용 |
|--------|-----------|---------|
| **집계/결합** | Aggregate, Multi-sig, Threshold | 블록체인 (BTC Taproot) |
| **프라이버시** | Ring, Group, Anonymous, Blind | 전자 투표, 익명 자격증명 |
| **특수 구조** | Homomorphic, Redactable, Sanitizable | 데이터 위임 서명 |
| **장기 보안** | Forward-secure, Leakage-resilient | 키 노출 대비 |
| **기반 수학** | Identity-based, Key-homomorphic | PKI 단순화 |

---

## 부록: 핵심 안전성 관계 정리

```
OW_RSA (1-wayness)
    │
    │  [Random Oracle 모델]
    ▼
UF-CMA_FDH (tight: q_H factor)
    │
    │  [더 tight한 reduction]
    ▼
UF-CMA_PSS (tight reduction)


DL (Discrete Log)
    │
    │  [Random Oracle 모델, Forking Lemma]
    ▼
UF-CMA_Schnorr / EdDSA
```

**핵심 메시지:**
1. Plain RSA는 UF-CMA에 대해 취약하다 — 곱셈 준동형성 때문
2. 해시(특히 RO로 모델링된 FDH)가 이를 해결한다
3. PSS는 FDH보다 tight한 reduction을 제공한다
4. 이산 로그 기반 스킴은 RSA와 독립적인 수학적 기반을 갖는다
5. 무작위성은 선택사항이지만, 올바른 구현이 매우 중요하다
