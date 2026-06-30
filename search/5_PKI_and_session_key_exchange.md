# CSE107 Lecture 14 심화 세미나 노트: PKI와 세션 키 교환

## Part 1. 공개키 설정의 근본 문제: 신뢰의 전달

공개키 암호 시스템에서 Bob은 비밀키 $sk[B]$와 이에 대응하는 공개키 $pk[B]$를 갖는다. Alice가 $pk[B]$를 이미 보유하고 있다는 가정 하에, 다음 두 가지 연산이 가능해진다.

- 암호화: $C \leftarrow \mathcal{E}_{pk[B]}(M)$, 복호화: $M \leftarrow \mathcal{D}_{sk[B]}(C)$
- 서명: $\sigma \leftarrow \mathcal{S}_{sk[B]}(M)$, 검증: $\mathcal{V}_{pk[B]}(M,\sigma)$

여기서 핵심 가정은 "Alice가 $pk[B]$를 갖고 있다"는 것 자체이며, 이 가정이 어떻게 충족되는가가 PKI 전체 논의의 출발점이다 (p. 4).

실제 TLS와 같은 프로토콜에서 Bob은 보통 서버(예: `google.com`)이고 Alice는 클라이언트로, 식별자는 도메인 이름이나 IP 주소가 된다 (p. 5).

### 1.1 순진한 해법과 그 실패: 신원-중간자 공격(Entity-in-the-Middle Attack)

가장 단순한 발상은 Bob이 키 생성 알고리즘 $\mathcal{K}$를 실행해 $(pk[B], sk[B]) \xleftarrow{\$} \mathcal{K}$를 얻고, $(B, pk[B])$를 그대로 Alice에게 전송하는 것이다 (p. 3).

문제는 이 채널이 인증되지 않았다는 점이다. 공격자 $E$가 통신 경로에 개입하여 자신의 키쌍 $(pk[E], sk[E]) \xleftarrow{\$} \mathcal{K}$를 생성한 뒤 $(B, pk[E])$를 Alice에게 전달하면:

$$C \xleftarrow{\$} \mathcal{E}_{pk[E]}(M) \quad \xrightarrow{C} \quad M \leftarrow \mathcal{D}_{sk[E]}(C)$$

$$\mathcal{V}_{pk[E]}(M,\sigma) \quad \xleftarrow{M,\sigma} \quad \sigma \xleftarrow{\$} \mathcal{S}_{sk[E]}(M)$$

Alice는 자신이 Bob과 통신한다고 믿지만 실제로는 $E$가 모든 암호문을 복호화하고 모든 서명을 위조할 수 있다. 즉 $E$가 Bob의 신원을 완전히 찬탈한다 (p. 4).

**심화: 왜 이것이 본질적으로 불가피한 문제인가.** 공개키 자체는 단순한 비트열(예: RSA의 경우 modulus $N$과 지수 $e$)에 불과하며, 어떤 수학적 구조도 "이 키가 특정 신원에 속한다"는 의미를 자체적으로 담고 있지 않다. 따라서 공개키와 신원의 바인딩(binding)은 반드시 외부의 신뢰 메커니즘으로 보강되어야 한다. 이것이 PKI의 존재 이유다.

## Part 2. PKI의 구조: 인증기관(CA)과 인증서(Certificate)

목표는 다음과 같다: Alice가 Bob의 공개키 $pk$를 진짜로 Bob의 것이라는 **증명**과 함께 얻는 것. 이를 위한 대중적 해법이 PKI이며, 두 핵심 요소는:

- **인증기관(CA)**: 신원-공개키 바인딩을 증명해주는 신뢰된 제3자
- **인증서(Certificate)**: 그 증명 자체 (p. 9)

다른 대안적 방법들 — Facebook이나 개인/회사 웹페이지에 공개키 게시, 이메일 첨부, OpenPGP SKS와 같은 키서버 등록, 직접 대면 전달 — 도 언급되지만, 이들은 확장성이나 신뢰성 면에서 CA 기반 모델만큼 체계적이지 않다 (p. 9, p. 23).

### 2.1 인증서 발급 절차 (Certificate Process)

전체 흐름은 다음 7단계로 구성된다 (p. 8):

1. Bob이 $(pk, sk) \xleftarrow{\$} \mathcal{K}$로 키쌍을 생성
2. Bob이 신원 $B$와 $pk$를 CA에 전송
3. CA가 신원 확인(identity check)을 수행하여 $pk$가 실제로 $B$의 것임을 검증
4. Bob이 $sk$를 보유하고 있음을 CA에 증명 (지식의 증명, proof of knowledge)
5. CA가 인증서를 발급
6. Bob이 인증서를 Alice에게 전송
7. Alice가 인증서를 검증하고 $pk$를 추출

**신원 확인의 구체적 방식**: 도메인 이름의 경우 CA가 챌린지를 보내 해당 도메인 웹페이지에 게시하도록 요구(ACME 프로토콜의 HTTP-01/DNS-01 검증과 유사); 이메일 주소의 경우 검증 링크를 발송; 여권/운전면허는 오프라인 물리적 검증이 가능하다. 비밀키 지식 증명은 CA가 도전 메시지에 서명하게 하거나 특정 평문을 복호화하게 함으로써, Bob이 단순히 타인의 공개키를 복사해 제출하는 것을 방지한다 (p. 11).

### 2.2 인증서의 수학적 구조

CA가 $pk$가 $B$의 것임을 확신하면, 다음 형태의 인증서를 구성한다:

$$\mathrm{CERT}[B] = (\mathrm{CERTDATA}, \sigma), \quad \sigma \leftarrow \mathcal{S}_{sk[CA]}(\mathrm{CERTDATA})$$

여기서 $\mathrm{CERTDATA}$는 $B$의 공개키 $pk$와 키 타입(RSA, EC 등), 신원 $B$, CA 이름, 만료일 등을 포함한다 (p. 14).

**검증 절차** (Alice 측에서 $pk[CA]$를 이미 보유했다고 가정):

1. $(\mathrm{CERTDATA}, \sigma) \leftarrow \mathrm{CERT}[B]$로 파싱
2. $\mathcal{V}_{pk[CA]}(\mathrm{CERTDATA}, \sigma) \stackrel{?}{=} 1$ 검증
3. $(pk, B, \text{expiry}, \ldots) \leftarrow \mathrm{CERTDATA}$ 추출
4. 만료 여부 확인
5. 신원이 의도한 대상과 일치하는지 확인

여기서 자연스러운 재귀적 질문이 등장한다: **Alice는 $pk[CA]$를 어떻게 얻는가?** 답은 CA의 공개키가 브라우저나 운영체제(macOS의 키체인 등)에 사전 내장되어 배포된다는 것이다 (p. 15, p. 18).

### 2.3 인증서 체인과 계층 구조

단일 루트 CA가 모든 신원을 직접 검증하는 것은 비효율적이므로, 계층적 위임 구조가 사용된다:

$$\mathrm{CA(USA)} \to \mathrm{CA(Calif)} \to \mathrm{CA(SD)} \to \mathrm{CA(UCSD)} \to \mathrm{Alice}$$

일반화된 표기로, 상위 기관 $X$가 하위 기관 또는 사용자 $Y$에게 발급하는 인증서는:

$$\mathrm{CERT}[X:Y] = \big( (pk[Y], Y, \ldots),\ \mathcal{S}_{sk[X]}((pk[Y], Y, \ldots)) \big)$$

Alice의 신원을 검증하려면 체인 전체

$$\mathrm{CERT}[\mathrm{CA(USA)}:\mathrm{CA(Calif)}],\ \mathrm{CERT}[\mathrm{CA(Calif)}:\mathrm{CA(SD)}],\ \mathrm{CERT}[\mathrm{CA(SD)}:\mathrm{CA(UCSD)}],\ \mathrm{CERT}[\mathrm{CA(UCSD)}:\mathrm{Alice}]$$

를 순차적으로 검증해 나가되, 신뢰의 출발점(trust anchor)은 오직 $pk[\mathrm{CA(USA)}]$ 하나로 충분하다 (p. 16).

이 구조의 실용적 이점은 세 가지다: (1) 하위 기관이 자신의 소속 구성원에 대해 더 풍부한 신원 정보를 갖고 있어 검증이 용이하고, (2) 검증 부담이 분산되며, (3) 클라이언트 소프트웨어는 루트 CA 공개키만 내장하면 되므로 신뢰 저장소(trust store) 크기가 작아진다 (p. 17).

**실무 보안 고찰 — 인증서 체인의 취약점**: 체인의 어느 한 중간 CA라도 비밀키가 탈취되면, 공격자는 해당 CA 산하의 임의 신원에 대한 유효한 인증서를 위조할 수 있다(2011년 DigiNotar 사건이 대표적 실례). 이는 PKI가 "가장 약한 고리(weakest link)" 문제를 갖는다는 것을 의미하며, 이를 완화하기 위한 메커니즘이 후술할 Certificate Transparency다.

### 2.4 OpenSSL을 이용한 키 생성과 인증서 요청 실습

**RSA 키 생성**:
```bash
$ openssl genrsa -out key.pem 3072
$ openssl rsa -in key.pem -outform PEM -pubout -out public.pem
```

**타원곡선(EC) 키 생성** (곡선 `prime256v1` = NIST P-256):
```bash
$ openssl ecparam -name prime256v1 -genkey -noout -out key.pem
$ openssl ec -in key.pem -pubout -out public.pem
```

(p. 13, p. 14)

**왜 EC 키와 인증서 요청이 RSA보다 짧은가**: RSA의 안전성은 큰 합성수의 소인수분해 문제(Integer Factorization Problem)에 기반하며, 일반수체 체(General Number Field Sieve, GNFS)와 같은 준지수 시간(subexponential) 알고리즘이 존재하기 때문에 충분한 안전성을 위해 2048~3072비트의 큰 모듈러스가 필요하다. 반면 타원곡선 이산로그 문제(ECDLP)는 알려진 최선의 공격이 지수 시간에 가까운 Pollard's rho 알고리즘($O(\sqrt{n})$)이므로, 256비트 키만으로 RSA 3072비트와 동등한 안전성(약 128비트 보안 수준)을 달성할 수 있다. 이는 슬라이드에서 언급된 "DLog 문제가 타원곡선에서 매우 어렵기 때문(very hard)"이라는 설명의 대수적 근거다 (p. 13 참조).

### 2.5 인증서 폐기(Revocation)와 그 한계

비밀키가 유출되는 등의 사유로 인증서를 폐기해야 할 경우:

1. Bob이 $\mathrm{CERT}[B]$와 폐기 요청을, $sk$로 서명하여 CA에 전송
2. CA가 $pk$로 서명을 검증
3. CA가 $(\mathrm{CERT}[B], \text{RevocationDate})$를 인증서 폐기 목록(CRL)에 등재
4. CRL이 배포됨

Alice는 Bob의 인증서를 수락하기 전에 CRL에 포함되어 있지 않은지 확인해야 하며, 이를 실시간으로 처리하는 프로토콜이 OCSP(Online Certificate Status Protocol)다 (p. 20).

**시간 지연 취약점(Revocation Gap)**: 슬라이드의 구체적 시나리오를 보면 —

- 5월 13일: Bob의 비밀키 유출
- 5월 16일: $\mathrm{CERT}[B]$가 폐기됨
- 5월 17일: Alice가 CRL을 확인함

이 3~4일의 간극 동안 공격자는 탈취한 $sk[B]$로 여전히 유효한 인증서를 악용할 수 있다. 실무에서 CRL은 방대해지고 실시간 동기화가 어려워, OCSP Stapling(서버가 자신의 OCSP 응답을 미리 받아 클라이언트에 제공하는 방식)과 같은 보완책이 등장했다 (p. 21).

### 2.6 Certificate Transparency: "감시자를 감시하는" 메커니즘

CA가 신원 확인 절차를 건너뛰거나 악의적으로 부정확한 인증서를 발급할 가능성(오발급, mis-issuance)에 대한 대응책이 Certificate Transparency(CT)다. CT는 모든 발급된 인증서를 공개적으로 검증 가능하고(publicly verifiable), 추가만 가능하며(append-only), 변조 불가능한(tamper-proof) 로그에 기록하도록 요구한다. 이 로그는 머클 트리(Merkle Tree) 구조를 기반으로 구축된다 (p. 22).

**심화: 머클 트리의 수학적 원리**. 리프 노드 $h_i = H(\text{cert}_i)$로부터 시작해, 인접한 두 해시값을 결합하여 상위 노드를 재귀적으로 구성한다:

$$h_{\text{parent}} = H(h_{\text{left}} \,\|\, h_{\text{right}})$$

루트 해시 하나만으로 $n$개의 리프에 대한 포함 증명(inclusion proof)을 $O(\log n)$ 크기의 증명 경로로 검증할 수 있다. 이는 누군가 로그에 몰래 인증서를 삽입하거나 제거하면 루트 해시가 즉시 변경되어 탐지된다는 의미이며, 따라서 CA의 부정 발급을 사후에라도 공개적으로 적발할 수 있게 한다.

## Part 3. 세션 키 교환: 비대칭에서 대칭으로

### 3.1 왜 세션 키가 필요한가 (하이브리드 암호화의 동기)

TLS와 같은 실제 프로토콜은 공개키 암호로 데이터를 직접 암호화하지 않는다. 대신 공개키 암호는 **세션 키 교환(session-key exchange)**에만 사용되어, 클라이언트 $A$와 서버 $B$가 대칭 세션 키 $K$를 공유하도록 한다. 이후 실제 데이터는 인증 암호화(Authenticated Encryption, $\mathcal{AE} = (\mathcal{K}, \mathcal{E}, \mathcal{D})$) 스킴 하에서 $K$로 보호된다:

$$M \leftarrow \mathcal{D}_K(C) \quad \xleftarrow{C} \quad C \xleftarrow{\$} \mathcal{E}_K(M)$$

(p. 24)

이유는 두 가지다 (p. 25):

1. **성능**: 대칭 암호 연산(AES 등)은 비대칭 연산(RSA, DH의 모듈러 거듭제곱)보다 수 자릿수 빠르다. RSA 암호화/복호화는 큰 정수에 대한 모듈러 지수 연산이 필요해 연산량이 $O(\log N)$번의 모듈러 곱셈을 요구하는 반면, AES는 비트 단위 치환-순열 네트워크로 훨씬 가볍다.
2. **아키텍처적 이유**: 인터넷 환경에서는 동일한 두 당사자가 여러 개의 동시다발적 세션을 가질 수 있으며, 세션마다 새로운(fresh) 키를 사용함으로써 한 세션의 보안 침해가 다른 세션에 영향을 주지 않도록 격리(session independence)할 수 있다.

### 3.2 Diffie-Hellman 키 교환 복습

순환군 $G = \langle g \rangle$, 위수(order) $m$, 그 위에서 CDH(Computational Diffie-Hellman) 문제가 어렵다고 가정한다. 해시함수 $\mathbf{H}: \{0,1\}^* \to \{0,1\}^k$를 사용한다.

$$A: x \xleftarrow{\$} \mathbb{Z}_m,\ X \leftarrow g^x \quad \xrightarrow{A, X} \quad B: y \xleftarrow{\$} \mathbb{Z}_m,\ Y \leftarrow g^y$$
$$A: L \leftarrow Y^x \quad \xleftarrow{B, Y}$$
$$B: L \leftarrow X^y$$

핵심 등식은 군의 결합법칙과 지수 법칙에 의해 성립한다:

$$Y^x = (g^y)^x = g^{xy} = (g^x)^y = X^y =: L$$

공유 세션 키는 $K = \mathbf{H}(L) = \mathbf{H}(g^{xy})$로 정의된다 (p. 26).

**소규모 수치 예시**: $G = \mathbb{Z}_{23}^*$ (위수 $m = 22$인 순환군이라 가정), 생성원 $g = 5$. Alice가 $x = 6$을 선택하면 $X = 5^6 \bmod 23 = 15625 \bmod 23 = 8$. Bob이 $y = 15$를 선택하면 $Y = 5^{15} \bmod 23 = 19$. 공유 비밀은 $L = Y^x \bmod 23 = 19^6 \bmod 23 = 2$이며, 동시에 $L = X^y \bmod 23 = 8^{15} \bmod 23 = 2$로 일치함을 확인할 수 있다(실제 RFC 표준에서는 $m$이 소수이거나 큰 소인수를 갖는 안전소수가 사용되어 소부분군 공격을 방지한다).

### 3.3 수동적 공격에 대한 안전성과 능동적 공격에 대한 취약성

**수동적 공격자(Passive/Eavesdropping Adversary)**는 통신을 도청하여 $X = g^x$와 $Y = g^y$를 획득하지만, $K = \mathbf{H}(g^{xy})$를 계산하려면 CDH 문제를 풀어야 하므로 가정상 불가능하다 (p. 27).

그러나 **인증성(authenticity) 문제는 여전히 남는다**. 이는 KEM(Key Encapsulation Mechanism)에서 공개키 $ek$의 진위성이 항상 가정되어야 했던 것과 동일한 구조적 문제다 (p. 27).

**능동적 공격자(Active Adversary)**에게 순수 DH는 무력하다. 신원-중간자 공격의 변형:

$$A: x \xleftarrow{\$} \mathbb{Z}_m,\ X \leftarrow g^x \quad \xrightarrow{A,X} \quad E: y \xleftarrow{\$} \mathbb{Z}_m,\ Y \leftarrow g^y$$
$$A: L \leftarrow Y^x \quad \xleftarrow{B,Y}$$
$$E: L \leftarrow X^y$$

$E$가 $B$를 사칭하면, Alice는 자신이 $K = \mathbf{H}(L)$을 Bob과 공유한다고 믿지만 실제로는 $E$와 공유하고 있다. Alice가 이후 $K$로 메시지를 암호화하면 $E$가 그대로 복호화할 수 있다. 즉 **순수 DH 자체는 세션 키 교환 문제를 해결하지 못한다** — 인증이 결여되어 있기 때문이다 (p. 28).

## Part 4. 세션 키 교환의 정형 요구사항

단방향(unilateral) 공개키 설정을 고려한다: $B$는 인증서 $\mathrm{CERT}[B]$와 키쌍 $(pk[B], sk[B])$를 갖지만, $A$는 인증서가 없다(TLS의 전형적 클라이언트-서버 비대칭 상황) (p. 29).

세션 키 교환은 다음을 만족해야 한다:

- **인증성(Authenticity)**: $A$가 실제로 $B$와 키를 공유하는 것이며, 다른 누구와도 아니다.
- **비밀성(Secrecy)**: 공격자가 $K$를 알 수 없다.

이는 공격자가 다른 세션들의 키를 이미 알고 있고(known-key security), 통신을 완전히 통제하는 능동적 공격자(active adversary)인 경우에도 성립해야 한다. 추가 요구사항으로 순방향 비밀성(forward secrecy), 익명성(anonymity) 등이 있다 (p. 29).

### 4.1 비밀성의 정형적 정의: 구별 불가능성 게임

비밀성은 IND(Indistinguishability) 스타일의 게임으로 정의된다. 프로토콜이 종료되고 한 당사자 $X$가 세션 키 $K$를 출력했다고 하자. 게임은 다음을 정의한다:

$$K_1 \leftarrow K, \quad K_0 \xleftarrow{\$} \{0,1\}^{|K|}$$

도전자가 $K_0$ 또는 $K_1$ 중 하나를 무작위로 반환하고, 공격자는 어느 쪽인지 구별해야 한다. 공격자의 우위(advantage)가 작아야 안전하다고 정의되며, 이는 다른 모든 세션의 키를 알고 있고 통신을 능동적으로 제어하는 상황에서도 성립해야 한다. 이 정의는 KEM의 IND-CCA 보안 개념과 본질적으로 유사한 구조를 갖는다 (p. 30).

## Part 5. 프로토콜 KE1, KE2, KE3: 점진적 보강을 통한 설계

### 5.1 KE1: 기초 인증 키 교환과 그 결함

$$A \xrightarrow{A, R_A} B$$
$$B \xrightarrow{B, R_B, \mathrm{CERT}[B], \mathrm{Sig}_B(R_A \| R_B)} A$$
$$A: C \xleftarrow{\$} \mathrm{Enc}_B(L) \xrightarrow{C, \mathrm{MAC}_M(R_A \| R_B \| C)} B$$

여기서 $R_A, R_B$는 각 당사자가 무작위로 선택하는 논스(nonce)이고, $\mathrm{Sig}_B(X)$는 $sk[B]$로 계산되어 $\mathrm{CERT}[B]$에 담긴 $pk[B]$로 검증 가능한 서명이다. $L$은 Alice가 무작위로 선택하며, 세션 키는 $K = \mathbf{H}_1(L)$, MAC 키는 $M = \mathbf{H}_2(L)$로 파생된다. $\mathrm{Enc}_B(L)$은 $pk[B]$ 하의 공개키 암호화다 (p. 32).

**데이터 흐름 단계별 분석**:
1. Alice가 무작위 논스 $R_A$를 생성해 신원과 함께 전송 — 이는 재전송 공격(replay attack) 방지용이다.
2. Bob이 자신의 논스 $R_B$, 인증서, 그리고 두 논스의 연결에 대한 서명을 반환 — 서명은 "내가 이 특정 세션에 참여하고 있다"는 신선성(freshness)이 보장된 증명이다.
3. Alice가 비밀값 $L$을 생성하고 $pk[B]$로 암호화하여 전송하며, $L$로부터 파생된 MAC 키로 무결성 인증을 첨부한다.

**신원-중간자 공격(Identity Mis-binding Attack)**: 공격자 $E$가 Alice와 Bob 사이에 끼어들어 단순 전달자(relay) 역할만 해도 공격이 성립한다.

$$A \xrightarrow{A,R_A} E \xrightarrow{E,R_A} B$$
$$A \xleftarrow{B,R_B,\mathrm{CERT}[B],\mathrm{Sig}_B(R_A\|R_B)} E \xleftarrow{B,R_B,\mathrm{CERT}[B],\mathrm{Sig}_B(R_A\|R_B)} B$$
$$A \xrightarrow{C,\mathrm{MAC}_M(\cdots)} E \xrightarrow{C,\mathrm{MAC}_M(\cdots)} B$$

여기서 $E$는 메시지 내용을 전혀 수정하지 않고 자신의 신원 라벨만 바꿔 끼운다. 결과적으로 **Alice는 자신이 Bob과 키를 공유한다고 믿지만, Bob은 자신이 $E$와 키를 공유한다고 믿는다.** $E$는 실제 키 $K$를 알지 못함에도 불구하고, 이는 신원 바인딩의 실패로 간주되는 심각한 공격이다. 좋은 정의는 이 상황을 공격 성공으로 판정해야 하며, 좋은 프로토콜은 "$A$가 $B$를 $K$로 수락했다면, $B$ 역시 $A$를 동일한(또는 관련된) $K$로 수락하거나 아무도 수락하지 않아야 한다"는 성질을 보장해야 한다 (p. 33).

**왜 이것이 실질적 위협인가**: 만약 Bob의 응용 계층이 "이 세션은 신원 $E$의 것"이라고 (잘못) 기록하고 그에 따라 권한(authorization)을 부여한다면, $E$는 자신이 키를 모르더라도 세션의 신원 귀속을 조작하여 권한 상승이나 로그 위조와 같은 부차적 공격을 시도할 수 있다.

### 5.2 KE2: 신원 바인딩 보강

해결책은 서명과 MAC에 양측의 신원 $A, B$를 명시적으로 포함시키고, 서버로부터의 확인 MAC을 추가하는 것이다:

$$A \xrightarrow{A, R_A} B$$
$$B \xrightarrow{B, R_B, \mathrm{CERT}[B], \mathrm{Sig}_B(A\|B\|R_A\|R_B)} A$$
$$A: C \xleftarrow{\$} \mathrm{Enc}_B(L) \xrightarrow{C, \mathrm{MAC}_M(0\|A\|B\|R_A\|R_B\|C)} B$$
$$B \xrightarrow{\mathrm{MAC}_M(1\|A\|B\|R_A\|R_B)} A$$

세션 키는 $K = \mathbf{H}_1(A\|B\|R_A\|R_B\|L)$, MAC 키는 $M = \mathbf{H}_2(A\|B\|R_A\|R_B\|L)$로 모든 컨텍스트(신원, 논스)를 키 유도 함수(KDF) 입력에 바인딩한다 (p. 34).

**도메인 분리(domain separation)의 역할**: 마지막 두 메시지에서 사용된 프리픽스 비트 $0$과 $1$은 클라이언트→서버 MAC과 서버→클라이언트 MAC을 같은 키 $M$ 아래에서도 서로 다른 입력 공간으로 분리하기 위함이다. 이것이 없다면, 한 메시지의 MAC을 다른 방향의 메시지에 재사용하는 반사 공격(reflection attack)이 이론적으로 가능해질 수 있다.

### 5.3 KE2의 결정적 약점: 순방향 비밀성의 부재

KE2 이후 Alice가 세션 키 $K$로 추가 데이터 $X$를 암호화해 전송한다고 하자: $C_B \xleftarrow{\$} \mathrm{Enc}_K(X)$.

**공격 시나리오** (p. 35):
- 4월 17일: 공격자 $E$가 전체 핸드셰이크 흐름($C$와 $C_B$ 포함)을 기록.
- 5월 13일: $E$가 Bob의 시스템을 침해해 $sk[B]$를 획득.
- 5월 17일: Bob이 $\mathrm{CERT}[B]$를 폐기.

문제는 폐기 시점과 무관하게, 5월 13일 이후 언제든 $E$가 다음을 수행할 수 있다는 점이다:

$$L \leftarrow \mathrm{Dec}_{sk[B]}(C), \quad K \leftarrow \mathbf{H}_1(A\|B\|R_A\|R_B\|L), \quad X \leftarrow \mathrm{Dec}_K(C_B)$$

즉 **과거에 기록해둔 암호문이 미래의 키 유출로 인해 소급하여 복호화 가능**해진다. 이것이 순방향 비밀성(Forward Secrecy) 위반이다 — KE2는 $L$ 자체를 $pk[B]$로 암호화해 전송하기 때문에, $sk[B]$ 단 하나의 유출이 모든 과거 세션 키를 무너뜨린다.

### 5.4 순방향 비밀성의 정의와 DH의 재등장

> **정의 (순방향 비밀성)**: $sk[B]$의 노출이 그 노출 시점 이전에 교환된 세션 키 $K$들의 복구를 허용하지 않아야 한다.

순방향 비밀성은 세션 키 교환 프로토콜 내부에 DH 키 교환을 삽입함으로써 달성되며, 현대 TLS 1.3에서 필수 요구사항으로 자리잡았다. 이러한 프로토콜들은 흔히 **인증된 DH 키 교환(Authenticated DH Key Exchange)**으로 불린다 (p. 36).

**대수학적 근거**: 핵심은 $sk[B]$가 더 이상 비밀값 $L$ 자체를 암호화하는 데 쓰이지 않고, 오직 일시적(ephemeral)으로 생성된 DH 값 $g^a, g^b$에 대한 **서명**에만 사용된다는 점이다. 서명 위조 능력($sk[B]$ 보유)과 CDH 문제를 푸는 능력은 독립적인 계산 문제이므로, $sk[B]$가 유출되어도 과거에 일시적으로 사용되고 폐기된 지수 $a, b$를 복구할 수 없는 한 $L = g^{ab}$도 복구할 수 없다.

### 5.5 KE3: 서명 기반 인증 DH 키 교환 (TLS 1.3의 핵심 원형)

순환군 $G = \langle g \rangle$, 위수 $m$, CDH 문제가 어렵다고 가정.

$$A \xrightarrow{A, g^a} B$$
$$B \xrightarrow{B, g^b, \mathrm{CERT}[B], \mathrm{Sig}_B(A\|B\|g^a\|g^b), \mathrm{MAC}_M(1\|A\|B\|g^a\|g^b)} A$$
$$A \xrightarrow{\mathrm{MAC}_M(0\|A\|B\|g^a\|g^b)} B$$

여기서 $a, b \xleftarrow{\$} \mathbb{Z}_m$는 각각 Alice, Bob이 매 세션마다 새로 생성하는 **일시적(ephemeral)** 지수이며, $g^a, g^b$가 동시에 논스 역할도 겸한다. DH 공유 비밀 $L = g^{ab}$로부터 세션 키와 MAC 키가 유도된다:

$$K = \mathbf{H}_1(A\|B\|g^a\|g^b\|L), \qquad M = \mathbf{H}_2(A\|B\|g^a\|g^b\|L)$$

(p. 37)

**KE2와의 근본적 차이**: KE3에는 공개키 **암호화**가 전혀 사용되지 않는다. 오직 **서명**만 사용된다. 비밀키 $sk[B]$의 역할이 "비밀값을 복호화하는 키"에서 "DH 공개값들의 진위를 보증하는 서명 키"로 전환된 것이 핵심 설계 전환점이다.

**왜 순방향 비밀성이 성립하는가**: $sk[B]$가 탈취되더라도 공격자가 얻는 것은 향후 메시지를 위조(Bob을 사칭)할 수 있는 능력뿐이다. 과거에 기록된 $g^a, g^b$ 쌍으로부터 $L = g^{ab}$를 복구하려면 CDH 문제를 직접 풀어야 하며, 이는 $sk[B]$의 지식과 무관한 별개의 계산적으로 어려운 문제다. 따라서 일시적 지수 $a, b$가 세션 종료 후 안전하게 폐기되었다면, 과거 세션 키는 영구적으로 안전하다 (p. 38).

이 구조는 TLS 1.3 핸드셰이크의 단방향(서버만 인증되는) 세션 키 교환의 핵심을 정확히 반영하며, 학술적으로는 Krawczyk(2003)의 **SIGMA(SIGn-and-MAc) 프로토콜**에 기반한다 (p. 31, p. 38).

**실무 보안 고찰 — Ephemeral 키 관리**: 실제 구현에서 순방향 비밀성을 달성하려면 $a, b$가 메모리에서 확실히 제거(zeroization)되어야 한다. 메모리 덤프나 스왑 파일에 잔존하는 경우 이론적 순방향 비밀성이 실질적으로 무력화될 수 있다(예: 2014년 Heartbleed 취약점은 비록 다른 메커니즘이지만 메모리 노출을 통한 키 복구의 실제 위험성을 보여준 사례). 또한 동일한 일시적 키쌍을 여러 세션에 재사용하면("semi-static" 키) 순방향 비밀성의 보장 범위가 축소되므로, 표준은 매 핸드셰이크마다 새로운 $a$를 생성할 것을 요구한다.

## Part 6. 종합: 설계 진화의 논리적 궤적

| 프로토콜 | 인증 방식 | 신원 바인딩 | 순방향 비밀성 |
|---|---|---|---|
| 순수 DH | 없음 | 없음 (능동적 공격에 취약) | 해당 없음 |
| KE1 | 서명 + 암호화된 $L$ | 결함 있음 (mis-binding 공격) | 없음 |
| KE2 | 서명 + 암호화된 $L$ + 양방향 MAC | 보강됨 | 없음 ($sk[B]$ 유출 시 전체 붕괴) |
| KE3 | 서명 + 일시적 DH | 보강됨 | 달성됨 |

세션 키 교환은 "단순히 명세하기는 쉽지만 올바르게 만들기는 매우 어려운(easy to specify, hard to get right)" 문제로 알려져 있으며, 그 이유는 능동적 공격자가 다수의 동시 세션을 조작할 수 있는 환경에서 보안을 증명해야 하기 때문이다. 정형 정의와 증명 가능 보안(provable security) 기반 접근은 1990년대 중반부터 2000년대까지 발전해왔고, 오늘날 표준(TLS 1.3 등)은 이러한 증명 기반 설계를 적극적으로 채택하고 있다 (p. 31).
