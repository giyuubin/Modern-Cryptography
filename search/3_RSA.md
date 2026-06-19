# RSA and Public-Key Encryption Security

본 문서는 현대암호학의 핵심인 RSA 암호 시스템의 수학적 기반부터 공개키 암호(PKE)의 엄밀한 안전성 모델(IND-CPA, IND-CCA)까지의 내용을 세미나 발표 및 심층 학습을 위해 구조화한 문서입니다. 대수학적 원리, 실제 구현 알고리즘, 그리고 양자내성암호(PQC)로의 전환 배경 등을 상세히 다룹니다.

---

## 1. RSA 암호 시스템의 대수학적 기반 (RSA Math & Generator)

RSA 암호 체계는 정수론의 핵심 개념인 오일러 파이 함수(Euler's Totient Function)와 소인수분해의 어려움에 기반합니다.

### 1.1. 오일러 파이 함수 (Euler's Totient Function)
오일러 파이 함수 $\varphi(N)$은 $N$과 서로소인 $N$ 이하의 양의 정수의 개수, 즉 크기가 $N$인 곱셈군 $\mathbb{Z}_{N}^{*}$의 크기($|\mathbb{Z}_{N}^{*}|$)를 나타냅니다.
어떤 정수 $N \ge 1$이 다음과 같이 소인수분해 된다고 가정해 보겠습니다.

$$
N = p_{1}^{\alpha_{1}} \cdot p_{2}^{\alpha_{2}} \cdot \dots \cdot p_{n}^{\alpha_{n}}
$$

이때, $\varphi(N)$은 다음과 같이 계산됩니다.

$$
\varphi(N) = N \times \left(1 - \frac{1}{p_1}\right) \times \left(1 - \frac{1}{p_2}\right) \times \dots \times \left(1 - \frac{1}{p_n}\right)
$$

RSA에서는 서로 다른 두 소수 $p$와 $q$를 곱하여 모듈러스(Modulus) $N = pq$를 생성하므로, 공식은 다음과 같이 극도로 단순화됩니다.

$$
\varphi(N) = (p-1)(q-1)
$$

예를 들어 $N=15 = 3 \cdot 5$인 경우, $\varphi(15) = (3-1)(5-1) = 8$이 되며, 실제로 $\mathbb{Z}_{15}^{*} = \{1, 2, 4, 7, 8, 11, 13, 14\}$로 그 원소의 개수가 8개임을 확인할 수 있습니다.

### 1.2. RSA 함수와 트랩도어 일방향 순열 (Trapdoor One-Way Permutation)
RSA 함수는 $\mathbb{Z}_{N}^{*}$에서 $\mathbb{Z}_{N}^{*}$로 매핑되는 순열(Permutation)입니다. 
- **암호화 지수(Encryption Exponent) $e$**: $e \in \mathbb{Z}_{\varphi(N)}^{*}$
- **복호화 지수(Decryption Exponent) $d$**: $d \in \mathbb{Z}_{\varphi(N)}^{*}$이며, $ed \pmod{\varphi(N)} = 1$을 만족하도록 설정됩니다.

함수의 정의는 다음과 같습니다.
- **순방향 (암호화)**: $f(x) = x^e \pmod N$
- **역방향 (복호화)**: $f^{-1}(y) = y^d \pmod N$

**[정확성 증명 (Proof of Correctness)]**
임의의 $x \in \mathbb{Z}_{N}^{*}$에 대하여 복호화가 원본 평문을 복원함을 증명할 수 있습니다. 이는 페르마의 소정리를 일반화한 오일러의 정리($x^{\varphi(N)} \equiv 1 \pmod N$)를 바탕으로 합니다.

$$
f^{-1}(f(x)) = (x^e)^d \pmod N = x^{ed} \pmod N
$$

이때 $ed \equiv 1 \pmod{\varphi(N)}$이므로, $ed = k\cdot\varphi(N) + 1$로 표현할 수 있습니다.

$$
x^{k\cdot\varphi(N) + 1} = (x^{\varphi(N)})^k \cdot x^1 \equiv 1^k \cdot x \equiv x \pmod N
$$

### 1.3. RSA 동작 예시 분해
$N=15$, $e=3$, $d=3$ (즉, $ed = 9 \equiv 1 \pmod 8$)으로 설정했을 때의 매핑 과정을 살펴보면 RSA가 어떻게 $\mathbb{Z}_{N}^{*}$ 상에서 일대일 대응(Bijective)을 이루는지 확인할 수 있습니다.
함수는 $f(x) = x^3 \pmod{15}$ 이고, 역함수는 $g(y) = y^3 \pmod{15}$ 입니다.

| $x$ | $f(x)$ (암호화 결과) | $g(f(x))$ (복호화 결과) |
| :---: | :---: | :---: |
| 1 | 1 | 1 |
| 2 | 8 | 2 |
| 4 | 4 | 4 |
| 7 | 13 | 7 |
| 8 | 2 | 8 |
| 11 | 11 | 11 |
| 13 | 7 | 13 |
| 14 | 14 | 14 |

위 표에서 보듯, 입력값 $x$가 $f(x)$를 거쳐 완전히 다른 값으로 치환되지만, 여기에 다시 지수 $d$를 취하는 $g(y)$를 적용하면 정확히 원래의 $x$로 되돌아옵니다. 트랩도어(비밀키 $d$)를 모른다면 $f(x)$에서 $x$를 역산하는 것은 매우 어렵습니다.

### 1.4. RSA 키 생성 알고리즘 (RSA Generator $\mathcal{K}_{rsa}$)
보안 매개변수(Security Parameter) $k$가 주어졌을 때, 안전한 RSA 파라미터를 생성하는 알고리즘은 다음과 같이 동작합니다. 암호화 지수 $e$가 고정된 상태(예: $e=3$)에서의 생성 절차입니다.

```text
Alg K_rsa^e(k)
  repeat
    p <- {2^{k/2-1}, ..., 2^{k/2}-1}  // k/2 비트 크기의 임의의 정수 선택
    q <- {2^{k/2-1}, ..., 2^{k/2}-1}
    N <- p * q
    M <- (p-1)(q-1)  // 파이 함수 값
  until (N >= 2^{k-1} and p, q are prime and gcd(e, M) = 1)
  
  d <- MOD-INV(e, M)  // 확장 유클리드 알고리즘 등으로 역원 계산
  return N, p, q, e, d
```
여기서 핵심은 $p$와 $q$가 소수인지 판별하는 효율적인 알고리즘이 존재한다는 점과, $\gcd(e, \varphi(N)) = 1$ 조건을 만족해야만 모듈로 역원 $d$가 존재한다는 점입니다.

*(참고: 교안 1p ~ 13p)*

---

## 2. RSA의 일방향성(One-wayness)과 인수분해 문제

### 2.1. 일방향성 정식화 (Game $OW_{\mathcal{K}_{rsa}}$)
RSA의 일방향성(OW, One-Wayness)은 공격자 $\mathcal{A}$가 공개키 $(N, e)$와 암호문 $y$가 주어졌을 때 평문 $x$를 찾아낼 확률을 의미합니다. 이를 엄밀한 게임 모델로 정의하면 다음과 같습니다.

```text
Game OW_{K_rsa}
  procedure Initialize
    (N, p, q, e, d) <- K_rsa
    x <- Z_N^*
    y <- x^e mod N
    return N, e, y

  procedure Finalize(x')
    return (x == x')
```
공격자 $\mathcal{A}$의 OW-Advantage는 이 게임에서 `true`를 반환할 확률입니다.

$$
\mathbf{Adv}_{\mathcal{K}_{rsa}}^{ow}(\mathcal{A}) = \Pr[OW_{\mathcal{K}_{rsa}}^{\mathcal{A}} \Rightarrow \text{true}]
$$

### 2.2. RSA 역연산의 조건적 용이성
RSA를 깨는 것(Inverting)은 다음 중 하나라도 알고 있다면 수학적으로 매우 쉬워집니다.
1. **$d$를 아는 경우**: 단순히 $x = y^d \pmod N$을 계산하면 됩니다.
2. **$\varphi(N)$을 아는 경우**: $e$와 $\varphi(N)$을 통해 $d = \text{MOD-INV}(e, \varphi(N))$을 쉽게 계산할 수 있습니다.
3. **$p, q$를 아는 경우**: $\varphi(N) = (p-1)(q-1)$을 계산할 수 있으므로, 2번의 과정으로 이어집니다.

결국 공격자에게 주어지는 유일한 정보는 모듈러스 $N$뿐이므로, RSA의 근본적인 안전성은 **"주어진 $N$으로부터 $p$와 $q$를 알아내는 것(소인수분해)"**이 얼마나 어려운가에 직결됩니다.

### 2.3. 소인수분해 알고리즘 (Factoring Algorithms)
가장 단순한 소인수분해 알고리즘은 $2$부터 $\sqrt{N}$까지 모든 수로 $N$을 나누어보는 것입니다.
```text
Alg FACTOR(N) // N = p*q
  for i = 2, ..., ceil(sqrt(N)) do
    if N mod i == 0 then
      p <- i; q <- N/i
      return p, q
```
하지만 이 나이브(Naive) 방식의 시간 복잡도는 $\mathcal{O}(\sqrt{N}) = \mathcal{O}(e^{0.5 \ln N})$로, $N$이 커질수록 천문학적인 시간이 소요됩니다. 암호 해독을 위해 고안된 발전된 소인수분해 알고리즘들의 복잡도는 다음과 같습니다.

| 알고리즘 (Algorithm) | 소요 시간 복잡도 (Time taken to factor $N$) |
| :--- | :--- |
| Naive | $\mathcal{O}(e^{0.5 \ln N})$ |
| Quadratic Sieve (QS) | $\mathcal{O}(e^{c(\ln N)^{1/2}(\ln \ln N)^{1/2}})$ |
| Number Field Sieve (NFS) | $\mathcal{O}(e^{1.92(\ln N)^{1/3}(\ln \ln N)^{2/3}})$ |

**[소인수분해 최고 기록 (Factoring Records)]**
발전된 NFS 알고리즘과 컴퓨팅 파워의 향상으로 소인수분해 한계는 점차 깨지고 있습니다.

| 비트 길이 (Bit-length) | 돌파 연도 | 사용된 알고리즘 |
| :---: | :---: | :---: |
| 400 | 1993 | QS |
| 512 | 1999 | NFS |
| 768 | 2009 | NFS |
| 829 | 2020 | NFS |

현재 1024비트 RSA 모듈러스는 대략 80비트의 보안 강도($2^{80}$ 연산 필요)를 제공하는 것으로 추산되며, 대규모 기관에 의해 해독될 위험이 있어 2010년경부터는 최소 2048비트 이상의 모듈러스를 사용할 것이 권장되고 있습니다.

*(참고: 교안 14p ~ 28p)*

---

## 3. 구현 효율성, 취약점, 그리고 양자 위협 (PQC의 대두)

### 3.1. 암호화 지수($e$)의 선택과 효율성
모듈러 거듭제곱 연산 $x^e \pmod N$을 수행할 때, 지수 $e$의 이진수 표현(Binary Expansion, $\text{bin}(e)$)에 포함된 1과 0의 개수에 따라 연산 비용이 결정됩니다. 비트 $b \in \{0,1\}$에 대해, $b=0$이면 1번의 모듈러 곱셈($c(0)=1$), $b=1$이면 2번의 모듈러 곱셈($c(1)=2$)이 필요합니다. 
따라서 이진수 표현에서 1의 개수가 적을수록 암호화 연산이 고속으로 처리됩니다.

| $e$ | $\text{bin}(e)$ (이진수 표현) | 특징 |
| :---: | :---: | :--- |
| 3 | 11 | 1이 2개 (가장 빠름, 하지만 패딩 없이 사용 시 극도로 취약) |
| 17 | 10001 | 1이 2개 |
| 65,537 | 10000000000000001 | 1이 2개 (현대 RSA 표준 권장값) |

### 3.2. 저지수 공격 (Low-exponent Attacks)
$e$를 작게 설정했을 때, RSA를 부적절하게 사용하면 일방향성(OW-security) 자체는 깨지지 않더라도 특정 조건에서 평문이 유출될 수 있습니다.
- Coppersmith's attack
- Franklin-Reiter attack
- Håstad attack (동일한 평문을 서로 다른 $e=3$ 공개키로 보낼 때 중국인의 나머지 정리를 이용해 복원)

따라서 RSA를 실무에 적용할 때는 반드시 적절한 패딩(Padding) 기법을 함께 설계하여 엄밀한 보안 증명을 거쳐야 합니다.

### 3.3. 쇼어 알고리즘과 양자 위협 (The Quantum Threat & PQC)
최근 학계(특히 Smart Security 및 암호학계)의 가장 큰 화두는 양자 컴퓨터의 발전입니다. **쇼어의 알고리즘(Shor's Algorithm)**이 양자 컴퓨터에서 실행될 경우, 소인수분해(Factoring) 문제와 이산 대수 문제(DLog)를 '다항 시간(Polynomial Time)' 내에 풀 수 있습니다.
- 이는 현재 널리 쓰이는 RSA, Diffie-Hellman(DH), 그리고 타원곡선암호(ECC) 체계를 모두 무력화시킵니다.
- 이에 대응하기 위해, 양자 알고리즘으로도 효율적으로 풀리지 않는다고 알려진 수학적 난제(예: 격자의 최단 벡터 문제, Lattices)를 기반으로 하는 **양자내성암호(PQC, Post-Quantum Cryptography)**에 대한 표준화 및 연구가 전 세계적으로 활발히 진행 중입니다 (예: CRYSTALS-Kyber, Dilithium 등).

*(참고: 교안 29p ~ 35p)*

---

## 4. 공개키 암호 체계(PKE)의 구조와 안전성 개념

### 4.1. 비대칭 암호와 키 분배 문제 해결
대칭키 암호 체계에서는 앨리스와 밥이 통신하기 전 반드시 사전에 비밀키 $K_{AB}$를 안전한 채널을 통해 공유해야 하는 심각한 제약이 있었습니다.
반면, 비대칭키(공개키) 암호 체계에서는 누구나 접근할 수 있는 **공개 암호화 키(ek)**와 수신자만 비밀로 간주하는 **비밀 복호화 키(dk)**를 분리함으로써 키 분배 문제를 우아하게 해결합니다.

### 4.2. PKE의 문법적 정의 (Syntax of a PKE scheme)
공개키 암호 체계 $\mathcal{AE} = (\mathcal{K}, \mathcal{E}, \mathcal{D})$는 세 가지 알고리즘으로 구성됩니다.
1. **키 생성 (Key Generation)**: $(ek, dk) \leftarrow^{\$} \mathcal{K}$ (난수를 기반으로 암호화/복호화 키 쌍 생성)
2. **암호화 (Encryption)**: $C \leftarrow \mathcal{E}_{ek}(M)$ (암호화 키 $ek$를 사용하여 메시지 $M$을 암호문 $C$로 변환. 이때 알고리즘 $\mathcal{E}$는 반드시 **확률적(Randomized)**이어야 함)
3. **복호화 (Decryption)**: $M' \leftarrow \mathcal{D}_{dk}(C)$ (복호화 키 $dk$를 사용하여 암호문 $C$를 원래 메시지로 복원)

**[정확성 요구사항 (Correct Decryption Requirement)]**
합법적으로 생성된 키 쌍과 유효한 메시지 $M$에 대하여 다음 확률이 반드시 1이어야 합니다.

$$
\Pr[\mathcal{D}_{dk}(\mathcal{E}_{ek}(M)) = M] = 1
$$

*(참고: 교안 36p ~ 42p)*

---

## 5. PKE의 엄밀한 안전성 증명: IND-CPA와 IND-CCA

현대 암호학에서 PKE 스킴이 '안전하다'고 평가받기 위해서는, 단순한 평문 복원 불가(One-Wayness)를 넘어 두 가지 강력한 구별 불가능성(Indistinguishability) 모델을 통과해야 합니다.

### 5.1. 선택 평문 공격에 대한 안전성 (IND-CPA)
IND-CPA(Indistinguishability under Chosen-Plaintext Attack)는 공격자가 공개키 $ek$를 가지고 있으면서, 두 개의 메시지 중 어느 것이 암호화되었는지 구별할 수 없어야 한다는 개념입니다.

**[IND-CPA Game 모델]**
공격자 $\mathcal{A}$는 환경(Game)과 상호작용합니다. 게임은 내부적으로 비밀 비트 $b \in \{0,1\}$를 선택합니다 ($b=0$이면 Left 게임, $b=1$이면 Right 게임).

```text
Game Left/Right_AE
  procedure Initialize
    (ek, dk) <- K
    return ek  // 공격자에게 공개키 제공

  procedure LR(M_0, M_1)
    return E_ek(M_b)  // 공격자가 길이 같은 M_0, M_1을 주면 M_b를 암호화하여 반환
```
공격자 $\mathcal{A}$의 IND-CPA 이점(Advantage)은 다음과 같습니다.

$$
\mathbf{Adv}_{\mathcal{AE}}^{\text{ind-cpa}}(\mathcal{A}) = \Pr[\text{Right}_{\mathcal{AE}}^{\mathcal{A}} \Rightarrow 1] - \Pr[\text{Left}_{\mathcal{AE}}^{\mathcal{A}} \Rightarrow 1]
$$

**[핵심 통찰: 암호화는 왜 결정론적(Deterministic)이면 안 되는가?]**
만약 $\mathcal{E}$가 난수를 사용하지 않는 결정론적 알고리즘이라면, 공격자는 공개키 $ek$를 가지고 스스로 $C_0 = \mathcal{E}_{ek}(M_0)$와 $C_1 = \mathcal{E}_{ek}(M_1)$을 계산해 볼 수 있습니다. 이후 `LR` 오라클이 반환한 암호문과 직접 계산한 값을 비교하면 $b$를 100% 확률로 맞출 수 있습니다. 따라서 **IND-CPA를 만족하기 위해 암호화 알고리즘 $\mathcal{E}$는 반드시 난수를 포함(Randomized)해야 합니다.**

### 5.2. 선택 암호문 공격에 대한 안전성 (IND-CCA)
현실 세계에서 공격자는 단순히 암호화만 해보는 것이 아니라, 시스템의 오류 메시지나 반응을 통해 암호문을 복호화해 보는 오라클(Padding Oracle Attack 등)에 접근할 수 있습니다. 이를 모델링한 것이 IND-CCA(Chosen-Ciphertext Attack)입니다.

IND-CCA 게임은 IND-CPA 게임과 동일하되, **복호화 오라클 (Decryption Oracle, `Dec`)**이 추가로 제공됩니다.

```text
Game Left/Right_AE
  procedure Initialize
    (ek, dk) <- K
    S <- empty_set
    return ek

  procedure LR(M_0, M_1)
    C <- E_ek(M_b)
    S <- S union {C}  // 챌린지 암호문 기록
    return C

  procedure Dec(C)
    if C in S then return bot  // 챌린지 암호문을 그대로 복호화하는 꼼수 방지
    else
      M <- D_dk(C)
      return M
```
공격자의 IND-CCA 이점 공식은 다음과 같습니다.

$$
\mathbf{Adv}_{\mathcal{AE}}^{\text{ind-cca}}(\mathcal{A}) = \Pr[\text{Right}_{\mathcal{AE}}^{\mathcal{A}} \Rightarrow 1] - \Pr[\text{Left}_{\mathcal{AE}}^{\mathcal{A}} \Rightarrow 1]
$$

**[IND-CCA의 중요성]**
- 어떤 스킴이 IND-CCA를 만족하면(공격자가 Dec 오라클 쿼리를 전혀 하지 않는 특수한 경우가 IND-CPA이므로) 논리적으로 IND-CPA도 만족합니다. 반대는 성립하지 않습니다.
- 현대의 모든 PKE 구현 및 사용 사례는 이 IND-CCA 안전성을 표준 검증 모델로 요구하고 있습니다.

*(참고: 교안 43p ~ 52p)*
