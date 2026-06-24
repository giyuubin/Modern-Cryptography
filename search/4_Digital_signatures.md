# CSE107: 현대암호학 - 전자서명 (Digital Signatures) 세미나 학습 문서

## 1. 전자서명의 동기와 보안 모델 (Motivation and Definitions)

### 1.1 전자서명의 필요성과 특징
전통적인 수기 서명(Handwritten signatures)은 문서에 물리적으로 종속되지만, 이를 단순히 스캔하여 디지털화(예: `mysig.png`)할 경우 쉽게 복사되어 위조될 수 있습니다. 따라서 안전한 전자서명은 **서명자의 신원(Signer's identity)**뿐만 아니라 **서명되는 메시지(Message)** 자체에도 수학적으로 종속되어야 합니다.
전자서명은 메시지 인증 코드(MAC, Message Authentication Code)와 유사한 목적을 가지지만, 비대칭키(공개키) 암호학을 기반으로 한다는 점에서 결정적인 차이가 있습니다. MAC은 송수신자가 동일한 대칭키를 공유하므로 검증자가 서명을 위조할 수도 있고, 제3자에게 서명의 유효성을 증명할 수 없습니다(부인 방지 불가). 반면 전자서명은 누구나 공개키로 검증할 수 있으나, 생성은 서명자의 개인키로만 가능하여 **부인 방지(Non-repudiation)** 및 **공개 검증성(Public Verifiability)**을 제공합니다.

### 1.2 전자서명 스킴의 구문(Syntax) 및 정확성(Correctness)
전자서명 스킴 $\mathcal{DSS} = (\mathcal{K}, \mathcal{S}, \mathcal{V})$는 다음 세 가지 다항시간 알고리즘으로 구성됩니다.
1. **키 생성 (Key Generation, $\mathcal{K}$)**: $(vk, sk) \leftarrow \mathcal{K}$
   - 검증키(Verification key, $vk$)와 서명키(Signing key, $sk$) 쌍을 생성합니다.
2. **서명 생성 (Signing, $\mathcal{S}$)**: $\sigma \leftarrow \mathcal{S}_{sk}(M)$
   - 서명키 $sk$와 메시지 $M$을 입력받아 서명 $\sigma$를 출력합니다. (확률적 알고리즘일 수 있음)
3. **서명 검증 (Verification, $\mathcal{V}$)**: $d \leftarrow \mathcal{V}_{vk}(M, \sigma)$
   - 검증키 $vk$, 메시지 $M$, 서명 $\sigma$를 입력받아 유효하면 $1$(Accept), 아니면 $0$(Reject)을 출력합니다.

**정확성 요구사항(Correctness Requirement):**
정상적으로 생성된 키 쌍과 서명에 대해 검증 알고리즘은 항상 $1$을 출력해야 합니다.
$$\Pr[\mathcal{V}_{vk}(M, \mathcal{S}_{sk}(M)) = 1] = 1$$

### 1.3 UF-CMA 보안 모델 (Existential Unforgeability under Chosen Message Attack)
전자서명의 표준 보안 모델은 **선택 평문 공격에 대한 실존적 위조 불가성(UF-CMA)**입니다. 공격자는 검증키 $vk$를 알고 있으며, 서명 오라클(Signing Oracle)에 접근하여 임의의 메시지 $M_i$에 대한 유효한 서명 $\sigma_i$를 얻을 수 있습니다.
- **승리 조건**: 공격자가 서명 오라클에 질의하지 않은 새로운 메시지 $M^*$에 대해 유효한 서명 $\sigma^*$를 생성하여 $\mathcal{V}_{vk}(M^*, \sigma^*) = 1$이 되면 위조에 성공한 것입니다. (Existential Forgery)
- 이 외에도 공격 목표에 따라 특정 메시지를 위조하는 선택적 위조(Selective forgery), 모든 메시지를 서명할 수 있게 되는 범용 위조(Universal forgery) 등으로 분류되나, 가장 방어하기 어려운 강력한 모델인 UF-CMA를 기준으로 안전성을 평가합니다.

*(해당 내용은 교안의 1~13 페이지에 대응)*

---

## 2. Plain RSA 서명과 대수학적 취약점

### 2.1 Plain RSA 서명 스킴 구조
RSA 암호의 수학적 기반을 그대로 서명에 적용한 방식입니다. 서명은 복호화(Decryption) 연산과 유사하게, 검증은 암호화(Encryption) 연산과 유사하게 동작합니다.
- **키 생성**: $N=pq$, $ed \equiv 1 \pmod{\varphi(N)}$을 만족하는 $(N, e, d)$를 생성하여 $vk=(N,e)$, $sk=(N,d)$로 설정.
- **서명**: 메시지 $y \in \mathbb{Z}_N^*$에 대해 $x \leftarrow y^d \pmod N$
- **검증**: $x^e \pmod N \stackrel{?}{=} y$

**정확성 증명**:
$$x^e \pmod N = (y^d)^e \pmod N = y^{ed \pmod{\varphi(N)}} = y^1 = y$$

### 2.2 Plain RSA의 치명적인 공격 벡터 (Attacks)
RSA의 일방향성(One-Wayness)에도 불구하고, Plain RSA는 수학적 구조(Homomorphic property)로 인해 UF-CMA 모델에서 완전히 붕괴됩니다.

1. **Trivial Message Attack (자명한 위조)**
   - 공격자 $\mathcal{A}$는 서명키 없이도 $y=1$에 대한 서명이 $x=1$임을 압니다. ($1^d \pmod N = 1$). 검증 $\mathcal{V}_{(N,e)}(1, 1)$은 통과됩니다.
2. **Multiplicative Attack (곱셈 성질을 이용한 위조)**
   - 서명 오라클에 $y_1, y_2$를 질의하여 각각 서명 $x_1, x_2$를 얻습니다.
   - 새로운 메시지 $y_3 = (y_1 \cdot y_2) \pmod N$의 서명은 $x_3 = (x_1 \cdot x_2) \pmod N$가 됩니다.
   - 증명: $(y_1 y_2)^d \equiv y_1^d y_2^d \equiv x_1 x_2 \pmod N$
3. **No-Message Attack (역방향 위조)**
   - 임의의 서명 값 $x \in \mathbb{Z}_N^*$를 먼저 선택한 후, $y = x^e \pmod N$을 계산합니다. 의미 없는 가비지(Garbage) 메시지일 확률이 높지만, $(y, x)$ 쌍은 수학적으로 완벽히 유효한 서명입니다.

추가로, Plain RSA는 메시지가 $\mathbb{Z}_N^*$ 공간에 속해야 하므로 임의의 길이(Arbitrary length)의 문자열을 서명할 수 없다는 한계가 있습니다.

*(해당 내용은 교안의 14~21 페이지에 대응)*

---

## 3. 해시 함수의 도입과 RSA FDH (Full Domain Hash)

### 3.1 Hash-and-Sign 패러다임과 PKCS#1 v1.5의 한계
메시지 길이 제한과 대수학적 공격을 방어하기 위해 메시지를 해시한 후 서명하는 방식이 도입되었습니다. 대표적인 것이 RSA PKCS#1 방식입니다.
- $H_{PKCS}(M) = 00 || 01 || FF || \dots || FF || h(M)$
- $\mathcal{S}_{(N,d)}(M) = H_{PKCS}(M)^d \pmod N$

상위 바이트에 패딩을 강제하여 앞선 No-Message Attack 등을 방어하지만, 이 방식은 구조적 한계가 있습니다. 해시 출력값의 상위 바이트가 고정되어 있으므로 치환된 값 공간이 $\mathbb{Z}_N^*$ 상에서 균일하게 분포(Uniformly random)하지 않습니다. 따라서 RSA의 일방향성(One-wayness) 가정만으로는 안전성을 수학적으로 완벽히 증명할 수 없습니다.

### 3.2 RSA Full Domain Hash (FDH)
RSA-FDH는 해시 함수 $H: \{0,1\}^* \rightarrow \mathbb{Z}_N^*$ 를 사용하여 메시지를 $\mathbb{Z}_N^*$ 전체 도메인으로 매핑합니다.
- **서명**: $y \leftarrow H(M)$, $x \leftarrow y^d \pmod N$
- **검증**: $x^e \pmod N \stackrel{?}{=} H(M)$

**보안성 분석 및 랜덤 오라클 모델(ROM)**:
해시 함수 $H$가 랜덤 오라클(이상적인 해시 함수)이며 충돌 회피성(Collision-Resistant)을 가진다면, RSA-FDH는 RSA 역함수 계산의 어려움에 기반하여 UF-CMA 측면에서 증명 가능하게 안전(Provably Secure)합니다. 만약 공격자가 $H(M_1) = H(M_2)$인 충돌을 찾는다면 동일한 서명 $x_1$이 두 메시지에 모두 유효하게 되어 서명 체계가 무너지므로, 충돌 회피성은 필수 조건입니다.

**구현 요소 (XOF 활용)**:
$\mathbb{Z}_N^*$ 도메인 전체로 매핑하기 위해 SHAKE256과 같은 XOF(eXtendable Output Function)를 사용하거나, 일반 해시(SHA-256)에 카운터를 붙여 결합하는 방식을 사용합니다. $H(M) = G(M, 2k) \pmod N$ 처럼 모듈러 연산을 통해 적절한 길이로 조정합니다.

*(해당 내용은 교안의 22~33 페이지에 대응)*

---

## 4. RSA PSS (Probabilistic Signature Scheme)

### 4.1 결정론적 서명에서 확률적 서명으로
RSA-FDH는 결정론적(Deterministic) 서명입니다. 동일한 메시지에는 항상 동일한 서명이 생성됩니다. 보안 증명을 더욱 견고하게(Tight reduction) 만들고 이론적 안전성을 극대화하기 위해, 서명 과정에 난수(Randomness)를 주입하는 확률적 서명 방식인 **RSA-PSS**가 등장하여 표준(RFC 8017)으로 자리 잡았습니다.

### 4.2 RSA-PSS 알고리즘 데이터 흐름 분석
서명 생성 시 무작위 솔트(Salt) $r \leftarrow \{0,1\}^l$을 생성하여 메시지 해시에 포함합니다.

1. **메시지 다이제스트 생성**: $w \leftarrow H_1(M || r)$
2. **마스킹 연산**: 난수 $r$을 $H_2(w)$와 XOR 연산하여 $r^* \leftarrow H_2(w) \oplus r$ 생성. 이를 통해 $w$와 결합 시 난수를 안전하게 숨깁니다.
3. **패딩 구조화**: $y \leftarrow 0 || w || r^* || H_3(w)$ (여기서 $H_3$는 추가적인 패딩 용도)
4. **RSA 복호화 연산**: 패딩된 $y$ 값을 $\mathbb{Z}_N^*$의 원소로 취급하여 $x \leftarrow y^d \pmod N$ 수행.

검증 시에는 서명 $x$를 $x^e \pmod N$으로 복원하여 패딩 블록 $y$를 뜯어내고, 복원된 $r^*$과 $w$를 이용해 $r$을 역산($r = r^* \oplus H_2(w)$)한 뒤 $H_1(M || r)$이 $w$와 일치하는지 확인합니다.

*(해당 내용은 교안의 34~37 페이지에 대응)*

---

## 5. 이산 대수(DLog) 기반 서명 스킴 (Schnorr, DSA, EdDSA)

이산 대수 문제(Discrete Logarithm Problem, DLP)를 기반으로 하는 서명 체계입니다. 교안의 핵심인 Schnorr 서명에 대한 깊이 있는 수학적 전개와 취약점 분석을 추가합니다.

### 5.1 Schnorr 서명 스킴의 대수학적 구조 및 원리
Schnorr 서명은 영지식 증명(Zero-Knowledge Proof)의 일종인 $\Sigma$-Protocol에 **Fiat-Shamir 변환(Heuristic)**을 적용하여 대화형 증명을 비대화형 전자서명으로 바꾼 매우 우아한 구조를 가집니다.

**파라미터 세팅**:
- 위수(Order)가 소수 $m$인 순환군(Cyclic Group) $G = \langle g \rangle$를 정의. (예: 타원곡선 군이나 $\mathbb{Z}_p^*$의 부분군)
- 해시 함수 $H: \{0,1\}^* \rightarrow \mathbb{Z}_m$

**1) 키 생성 (Key Generation)**
- 개인키(서명키): $x \leftarrow \mathbb{Z}_m$ (무작위 선택)
- 공개키(검증키): $X \leftarrow g^x$

**2) 서명 생성 (Sign)**
- 메시지 $M$을 서명하기 위해, 매번 **일회성 난수(Nonce) $r$**을 무작위로 선택: $r \leftarrow \mathbb{Z}_m$
- 난수에 대한 군 연산(Commitment): $R \leftarrow g^r$
- 챌린지 생성(해시): $c \leftarrow H(R || M)$ (Fiat-Shamir 변환 적용 지점)
- 서명 응답 연산: $s \leftarrow (r + cx) \pmod m$
- 출력: 서명 쌍 $(R, s)$

**3) 검증 (Verify)**
- 서명 $(R,s)$를 받아 챌린지 $c \leftarrow H(R || M)$을 재계산.
- 검증식: $g^s \stackrel{?}{=} R \cdot X^c$ 확인.

**수학적 정확성 증명**:
$$g^s = g^{r + cx} = g^r \cdot g^{cx} = g^r \cdot (g^x)^c = R \cdot X^c$$

### 5.2 구체적인 소수 연산 예시 (Concrete Example)
직관적인 이해를 돕기 위해 아주 작은 소수군을 가정한 예시입니다.
- 소수 $p = 23$, 하위 순환군 위수 $m = 11$, 생성자 $g = 2$ ($2^{11} \equiv 1 \pmod{23}$)
- **키 생성**: 개인키 $x = 4 \in \mathbb{Z}_{11}$ 선택. 공개키 $X = 2^4 \pmod{23} = 16$.
- **서명**: 난수 $r = 3$ 선택. $R = 2^3 \pmod{23} = 8$.
  - 해시를 통해 $c = H(8 || M) = 5$가 나왔다고 가정.
  - $s = (3 + 5 \times 4) \pmod{11} = 23 \pmod{11} = 1$.
  - 서명: $(R=8, s=1)$
- **검증**: $g^s = 2^1 = 2 \pmod{23}$.
  - $R \cdot X^c = 8 \cdot 16^5 \pmod{23}$.
  - $16 \equiv -7$, $(-7)^5 = -16807 \equiv 6 \pmod{23}$.
  - $8 \cdot 6 = 48 \equiv 2 \pmod{23}$.
  - 양변이 $2$로 일치하여 서명 검증 성공.

### 5.3 치명적인 구현 취약점: Nonce 재사용 공격 (Nonce Reuse Attack)
Schnorr, DSA, ECDSA 등 일회성 난수 $r$을 사용하는 알고리즘에서 **가장 치명적인 보안 취약점은 동일한 $r$을 두 번의 서명에 재사용하는 것**입니다. 소니 플레이스테이션 3 (PS3) 해킹 사건이 이 취약점으로 인해 발생했습니다.

만약 동일한 개인키 $x$와 동일한 난수 $r$로 두 개의 서로 다른 메시지 $M_1, M_2$에 서명했다면:
- 서명 1: $s_1 = r + c_1 x \pmod m$
- 서명 2: $s_2 = r + c_2 x \pmod m$

공격자는 이 두 수식을 빼서 $r$을 소거할 수 있습니다.
$$s_1 - s_2 \equiv x(c_1 - c_2) \pmod m$$
$$x \equiv (s_1 - s_2)(c_1 - c_2)^{-1} \pmod m$$
공격자는 공개된 정보($s_1, s_2, c_1, c_2$)만으로 서명자의 **마스터 개인키 $x$를 즉시 복원**할 수 있으며, 시스템은 완전히 붕괴됩니다.

### 5.4 최신 동향 (EdDSA, DSA/ECDSA)
- **EdDSA (Edwards-curve Digital Signature Algorithm)**: 난수 $r$ 의존성 취약점을 극복하기 위해, EdDSA (RFC 8032)는 난수 $r$을 시스템 난수 생성기에 의존하지 않고 **메시지와 개인키의 일부를 해시($H(sk_{part} || M)$)하여 결정론적으로(Deterministic) 생성**합니다. 이는 Nonce 재사용 공격을 원천 차단하며, OpenSSH 및 GnuPG 등 최신 인프라에서 널리 쓰입니다.
- **DSA/ECDSA**: ElGamal 서명에서 발전된 형태로 NIST FIPS 186-4에 표준화되어 있으나, 복잡한 파라미터 구조와 난수 의존성 문제로 인해 점차 EdDSA나 다른 현대 암호로 대체되는 추세(FIPS 186-5 초안)입니다.

*(해당 내용은 교안의 38~41 페이지에 대응)*

---

## 6. 서명 구현 시의 해싱과 난수 생성 오류 분석

### 6.1 불량한 난수 생성기의 위험성
암호학에서 안전한 난수 생성(CSPRNG)은 생명과도 같습니다. 교안의 코드 조각(`return 4; // chosen by fair dice roll`)은 시스템 내장 난수 생성기가 예측 가능한 값을 반환할 때의 위험성을 풍자합니다. 앞서 분석한 바와 같이, 확률적 서명에서 고정되거나 예측 가능한 난수가 사용되면 개인키가 유출될 수 있습니다. (다만 RSA-PSS의 경우는 난수가 유출되어도 개인키가 즉각 털리지는 않으나 보안 증명이 무효화됩니다.)

### 6.2 안전하지 않은 해시 구조 (CRC의 오남용)
전자서명에서 해싱이 없는 스킴은 Plain RSA처럼 자명한 공격에 노출됩니다. 그러나 해싱을 하더라도 **암호학적 해시 함수가 아닌 오류 검출 코드(예: CRC16)**를 사용하면 심각한 취약점이 발생합니다.
- 교안 예시: 서명을 $(CRC_{16}(M) \times 2^{k-1-16} + M)^d \pmod N$ 형태로 구성.
- **취약점 분석**: CRC(Cyclic Redundancy Check)는 단순한 다항식 나눗셈 기반이므로 선형성(Linearity)을 띄고 있으며, 충돌(Collision)을 의도적으로 만들어내기 매우 쉽습니다. 또한 16비트 출력 공간은 $2^{16} = 65,536$에 불과하여 브루트포스 공격에 즉각 무너집니다. 공격자는 $CRC_{16}(M) = 0$이 되는 메시지들을 쉽게 찾아내어 서명의 대수적 곱셈 속성을 이용해 위조 서명을 생성할 수 있습니다.

따라서 전자서명에는 반드시 SHA-256, SHA-3(Keccak), BLAKE2 등 충돌 회피성과 역상 저항성이 수학적으로 보장된 해시 함수를 전체 메시지 도메인(FDH)에 걸쳐 적용해야 합니다.

*(해당 내용은 교안의 42~48 페이지에 대응)*
