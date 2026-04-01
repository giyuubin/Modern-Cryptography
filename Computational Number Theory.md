**Computational Number Theory**

- Intro
  - Secret key exchange
  - DH Secret Key Exchange
  - Notation
  - Integers mod N
  - Division and mod
- Groups
  - Definitions, properties, and notations
    - Closure
    - Associativity
    - Identity
    - Inverse
  - Exponentiation
- Computing in $\mathbb{Z}_N$ and $\mathbb{Z}_N*$
  - Computational Shortcuts
  - Group Orders
  - Simplifying exponentiation
- Algorithms on numbers
  - (Extended) gcd
    - EXT-GCD
    - Lemma
    - Modular Inverse
  - Exponentiation
    - Square-and-Multiply Exponentiation Algorithm



- **Intro**
  - Secret key exchange
    - Symmetric Key
      - 방식: 암호화 키 = 복호화 키
      - 장점: 암복호화 속도가 빠름
      - 단점: 통신 상대마다 키가 필요해 관리가 어렵고($\frac{n(n-1)}{2}$ 개), 키 전달 과정에서 탈취 위험이 있다
      - 용도: 대량의 데이터 암호화(파일 저장 등)

    - Public Key
      - 방식: 암호화 키(공개) $\neq$ 복호화 키(비밀)
      - 장점: 키 분배가 필요 없고, 공개키는 노출되어도 무방하여 안전함
      - 단점: 대칭 키에 비해 속도가 매우 느림(수백~수만 배)
      - 용도: 키 교환, 전자 서명
     
    - 실제로는?
      - 속도가 빠른 대칭 키로 실제 데이터를 암호화하고, 그 대칭 키를 안전하게 공유하기 위해 공개 키 방식을 사용하는 하이브리드 방식(Ex: HTTPS/SSL)을 주로 사용한다

  - DH Secret Key Exchange
    - 특징
      - 비밀 키 생성: 데이터를 직접 암호화하는 것이 아니라, 대칭 키 암호화에 사용할 비밀 키를 안전하게 만드는 것
      - 이산대수 문제: 거대한 숫자의 거듭제곱 계산은 쉽지만, 결과값에서 지수(개인 키)를 찾아내는 것은 매우 어렵다는 수학적 원리에 기반
      - 중간자 공격 취약: 상대방의 신원을 확인하는 인증 절차가 없기 때문에, 공격자가 중간에서 가로채는 중간자 공격에 노출될 수 있어 현대에는 디지털 서명 등과 함께 사용됨
     
    - 작동 원리(수학적 절차)
      - Alice와 Bob이 비밀키를 공유하는 과정
        1. 공개 파라미터 합의: 아주 큰 소수 *p*와 생성자 *q*를 공개적으로 정한다
        2. 개인 키 생성: 앨리스는 자신만의 비밀 숫자 *a*, 밥은 *b*를 선택한다
        3. 공개 값 계산 및 교환
           - 앨리스: A = $g^a (\bmod p$)를 계산하여 밥에게 보낸다
           - 밥: B = $g^b (\bmod p$)를 계산하여 앨리스에게 보낸다
        4. 공통 비밀 키 도출
           - 앨리스: K = $B^a$ (mod p) = $(g^b)^a$ (mod p)
           - 밥: K = $A^b$ (mod p) = $(g^a)^b$ (mod p)
           - 결과적으로 두 사람은 동일한 값 K = $g^ab$ (mod p)를 공유하게 됨

  - Notation
    1. $\mathbb{Z}$(정수 집합, Integers)
       - 음의 정수, 0, 양의 정수를 모두 포함하는 집합
       - $\mathbb{Z}$ = {...,-2, -1, 0, 1, 2,...}
    2. $\mathbb{N}$(자연수 집합, Natural Number)
       - 컴퓨터 과학이나 집합론에서는 보통 0을 포함시킴
       - $\mathbb{N}$ = {0, 1, 2,...}
    3. $\mathbb{Z}_+$(양의 정수 집합, Positive Integers)
       - 정수 중에서 0보다 큰 값들만 모은 집합
       - 0을 포함하지 않는 자연수와 동일한 집합
       - $\mathbb{Z}_+$ = {1, 2, 3,...}
  
  - Integers mod N
    - For *N* $\in \mathbb{Z}_+$
      ($\because$ 암호학에서 소수점이나 음수로 나머지를 구하는 건 의미가 없다)
       - $\mathbb{Z}_N$ = {0, 1,..., N-1}
         - 어떤 수를 N으로 나누었을 때 나올 수 있는 모든 나머지 집합(모듈로 N 연산)
         - 참고: 집합 $\mathbb{Z}_N$ 은 덧셈과 곱셈을 모두 할 수 있지만, 곱셈에서 0이나 $p$ 같은 숫자들 때문에 '역원(나눗셈)'을 구할 수 없다. 그래서 $\mathbb{Z}_N$ 은 "링(Ring)은 맞지만, 곱셈에 대한 그룹(Group)이나 필드(Field)는 될 수 없다.
       - $\mathbb{Z}_N^*$ = {a $\in \mathbb{Z}_N : gcd(a, N) = 1}
         - $\mathbb{Z}_N$중에서 N과 최대공약수가 1인(즉, 서로소인) 애들만 따로 뽑아낸 집합(역원을 구하 위해)
       - $\varphi(N)$ = $|\mathbb{Z}_N^*|$
         - $(\mathbb{Z}_N^*)$ 의 집합의 크기(원소의 개수)
         - Ex) <img width="395" height="136" alt="image" src="https://github.com/user-attachments/assets/2349b855-7622-441d-bef3-c3eb45faf7b5" />

  - Division and mod(앞으로 모든 계산은 N으로 나눈 나머지(mod)를 기준으로 돌아간다)
    - INT-DIV(정수 나눗셈): 어떤 수 a를 N으로 나누었을 때, 목 q와 나머지 r을 반환하는 함수
      - INT-DIV(a, N) returns (q, r) such that
        - a = qN + r
        - $0 \leq r < N$

    - The mod operation(모듈로 연산)
      - a mod N = $r \in \mathbb{Z}_N$
      - mod는 +, * 등과 같이 두 개의 인자를 받는(이항) 연산이다

    - Congruences(합동): mod 연산에 대한 설명
      - $a \equiv b \pmod N$ means $a \bmod N = b \bmod N$
        - **$a \equiv b \pmod N$ 이 성립한다는 것은, 두 수의 차이($a - b$)가 $N$의 배수이다!**
        
- Groups
  - 배경
    - 방정식 $a \cdot x = b$ 를 풀어서 미지수 $x$를 구하는(즉, 암호문을 복호화하는) 과정을 생각해보자
      1. 미지수 $x$를 구하려면 $a$를 없애야 하니, $a$의 역원($a^{-1}$)을 양변에 곱해야 한다.
      2. $a^{-1} \cdot a$ 가 무의미해지려면, 곱했을 때 원래 상태를 유지해 주는 항등원($id$)의 개념이 필요하다.
      3. $(a^{-1} \cdot a) \cdot x$ 를 계산할 때 괄호를 마음대로 쳐도 결과가 망가지지 않아야 하니 결합법칙이 필요하다.
      4. 이 모든 계산 결과가 애초에 우리가 정해둔 숫자 범위 밖으로 나가면 안 되니 닫힘이 필요하다.
    - 따라서 무언가를 역연산 할 수 있는 필수적인 세계를 정의하기 위해 이 4가지만 남긴 것이다.
      
  - Binary operation: 연산 기호의 생김새와 상관없이 어떤 집합 안에서 원소 두 개(a, b)를 골라 어떤 '특정 규칙에 넣었을 때 결과값이 나오기만 한다면 그것을 모두 binary operation이라 부른다.

  - Group의 공식 정의
    - 어떤 그룹 *G*가 group으로 인정받으려면 closure, associativity, identity, inverse를 모두 만족해야한다.
    - 양의 정수 *N*에 대하여, 멤버 집합 $\mathbb{Z}_N^*$와 모듈로 곱셈의 조합은 이 4가지 조건을 만족한다.(이제부터 4가지를 하나씩 증명)

  - Definitions, properties, and notations
    - Closure(닫힘)
      - 정의: 집합 안에서 두 원소 a, b를 골라 연산했을 때, 그 결과값도 반드시 원래 집합 *G*안에 있어야 한다.
      - 실패 예시: 기본 집합 $\mathbb{Z}{12}$에서 곱셈을 하면 $7 \cdot 5 = 35$가 된다. 35는 $\mathbb{Z}{12}$에 속하지 않으므로 닫혀있지 않는다.
      - 성공 예시: 정예 멤버 집합 {1, 5, 7, 11}에서 5와 7을 골라 모튤로 12 곱셈을 하면 $5 \cdot 7 \bmod 12 = 35 \bmod 12 = 11$이다. 11은 집합안에 존재한다.
      - 수학적 원리: 어떤 두 숫자가 N과 서로소라면, 그 두 숫자를 곱해서 N으로 나눈 나머지 역시 N과 서로소라는 $\gcd(ab \bmod N, N) = 1$이 적용되기 때문이다.
      - <img width="677" height="131" alt="image" src="https://github.com/user-attachments/assets/94f3e028-9d90-4986-8b74-28149a22c2d2" />

    - Associativity(결합법칙)
      - 정의: a, b, c 세 숫자를 연산할 때, 괄호를 쳐서 앞의 두 개를 먼저 계산하든 뒤의 두 개를 계산하든 최종 결과가 같아야 한다.
      - 배경: 일반 곱하기는 괄호를 마음대로 쳐도 상관없는데, 곱하고 나서 *N*으로 나누는 모듈로 규칙을 씌워도 결합법칙이 여전히 안 깨지고 잘 작동할까? 
      - 수학적 원리: $((ab \bmod N)c) \bmod N = (a(bc \bmod N)) \bmod N$ 이 항상 성립한다.
      - 증명 예시: $\mathbb{Z}_{12}^*$의 원소 5, 7, 11를 이용해 보면,
        1. 앞을 먼저 묶을 때: $(5 \cdot 7 \bmod 12) \cdot 11 \bmod 12 \rightarrow 11 \cdot 11 \bmod 12 = 1$
        2. 뒤를 먼저 묶을 때: $5 \cdot (7 \cdot 11 \bmod 12) \bmod 12 \rightarrow 5 \cdot 5 \bmod 12 = 1$
           - 양쪽 결과가 1로 완벽하게 동일하다.(암호 연산의 순서가 보장됨)
             
    - Identity(항등원)
      - 정의: 어떤 원소 a에 연산을 취했을 때, 자기 자신 a를 그대로 튀어나오게 유지시켜주는 원소($id$)가 집합 안에 존재해야 한다.
      - 적용: 곱셈 모듈로 연산에서 항등원은 당연히 1이다. 어떤 수든 1을 곱하고 $N$으로 나누면 자기 자신의 나머지가 그대로 나오기 때문이다($a \cdot 1 \bmod N = a$). 집합 $\mathbb{Z}_N^*$에는 항상 1이 포함되어 있으므로 조건을 만족한다.
        
    - Inverse(역원)
      - 정의: 집합 안에 모든 원소 a는, 자신과 연산했을 때 결과를 항등원($id$, 즉 1)으로 만들어주는 짝 b를 가져야 한다($a \cdot b \bmod N = 1$).
      - 암호학적 의미: inverse가 복호화 키 역할을 한다.
      - 증명 예시: $\mathbb{Z}_{12}^*$ 에서 5의 역원은 $5 \cdot b \bmod 12 = 1$ 을 만족하는 $b$를 집합 $\{1, 5, 7, 11\}$ 안에서 찾으면 된다.
        - $b = 5$ 를 대입하면 $5 \cdot 5 = 25$ 이고, $25 \bmod 12 = 1$ 이 성립하므로 5의 역원은 5이다.(Lemma part에서 더 자세히 다룰예정)
          
  - Exponentiation(거듭제곱)
    - 배경
      - 그룹 안에서 원소 a를 여러 번 반복해서 연산하는 것을 수학적으로 어떻게 표기할 것인가?
      - 우리는 이제 그냥 숫자가 아니라 모듈로 $N$이 적용된 $\mathbb{Z}_N^*$이라는 특수한 환경에 있다. 그렇다면 우리가 배웠던 지수 법칙들이 이 특수한 세계에서도 에러없이 똑같이 작동하는가? 그렇다!

    - Multiplicative notation(곱셈 표기법)
      - 연산 기호가 곱셈 계열일 경우 $a^n$으로 표기한다
      - $a^0 = id$ (항등원, 보통 1)로 정의한다
      - $a^{-1}$ 은 $a$의 역원(inverse)을 뜻하며, $a^{-n}$ 은 역원을 $n$번 곱한 $(a^{-1})^n$ 을 의미한다.
      - 우리가 아는 지수 법칙($a^{i+j} = a^i \cdot a^j$)을 "평소처럼(as usual)" 쓸 수 있다.

    - Additive notation(덧셈 표기법- 참고)
      - 연산 기호가 덧셈일 경우 $na$ 또는 $[n]a$ 로 표기한다. (타원 곡선 암호에서 가끔 쓰인다)
        
- Computing in $\mathbb{Z}_N$ and $\mathbb{Z}_N*$
  - Computational Shortcuts
    - $abc \bmod N = ?$
      - Slow way: 다 곱해서 큰 수를 만든 다음 $N$으로 나눈다.
      - **Faster way**: 두 개를 곱할 때마다 바로바로 $N$으로 나누어 나머지를 구한다.
      - Ex) ($5 \cdot 8 \cdot 10 \cdot 16 \bmod 21$ 계산하기):
        1. $5 \cdot 8 = 40 \rightarrow \mathbf{19}$
        2. $\mathbf{19} \cdot 10 = 190 \rightarrow \mathbf{1}$
        3. $\mathbf{1} \cdot 16 = \mathbf{16}$

    - 공식: $abc \bmod N = ((ab \bmod N)c) \bmod N$

    - 암호학적 의미: 계산의 매 단계마다 모듈로로 숫자를 쳐내서, 항상 숫자의 크기를 $N$보다 작게 유지해야 컴퓨터가 메모리 초과 없이 암호 연산을 할 수 있다.

    - Ex) $5^8 \bmod 14$
      - $(5^2)^4 \equiv 11^4 \equiv (11^2)^2 \equiv (-3)^2 \equiv 9^2 \equiv 81 \equiv 25 \equiv 11$
        - 앞서 언급한 "$a \equiv b \pmod N$ 이 성립한다는 것은, 두 수의 차이($a - b$)가 $N$의 배수이다"를 이용해 $11 \equiv -3 \pmod{14}$로 만들어 계산.
      
  - Group Orders(군의 위수)
    - 배경: 그렇다면 $5^{100000}$같은 계산은 어떻게 해야하는가..

    - 정의: 어떤 그룹 $G$ 안에 들어있는 **'원소의 총개수'**를 뜻하며, 기호로 $|G|$ 라고 쓴다.
      - $\mathbb{Z}_N^*$ 의 위수는 1부터 $N$까지의 숫자 중 $N$과 서로소인 숫자의 개수이다.
        - 참고) 이 서로소의 개수를 구하는 함수를 오일러 파이 함수($\phi(N)$)라 한다
        - 즉, $|\mathbb{Z}_N^*| = \phi(N)$이다.

      - Ex) $N = 14$,
        - $\mathbb{Z}_{14}^* = \{1, 3, 5, 9, 11, 13\}$
        - 원소의 개수 = 6개
        - $\phi(14) = 6$

    - Fact: 어떤 그룹 $G$의 위수가 $m$이라고 하자. 그러면 그룹 안의 모든 원소 $a$에 대하여, $a^m = id$ 가 성립한다.
      - 앞서 확인한대로 $\mathbb{Z}_N^*$ 의 위수 $m$을 구하는 공식이 바로 오일러 파이 함수 $\phi(N)$이다. 따라서 위 공식 $a^m = id$ 의 $m$ 자리에 $\phi(N)$ 을, $id$ 자리에 $1$을 대입하면 그 유명한 $a^{\phi(N)} \equiv 1 \pmod N$ (오일러의 정리)가 유도된다.
     
    - Euler's Theorem(오일러 정리)
      - **집합 안의 어떤 원소 a**(뒤에서 중요함)를 골라서, 위수 $\phi(N)$ 만큼 거듭제곱을 하면 그 결과는 무조건 항등원 1이 된다.
      - $$a^{\phi(N)} \equiv 1 \pmod N$$

      - Ex) $N=14$, $\phi(14)=6$
        - $3^6 \equiv 1 \pmod{14}$
        - $5^6 \equiv 1 \pmod{14}$
        - $11^6 \equiv 1 \pmod{14}$
        - ..

  - Simplifying exponentiation(거듭제곱 단순)
    - Corollary: 어떤 그룹 $G$의 위수가 $m$이라고 할 때, $$a^x = a^{x \bmod m}$$이 성립한다.
      - 지수 $x$가 아무리 커도, 그 지수를 $x \bmod m$(군의 위수)로 깎아내서 계산한 결과와 같다.

    - 증명:
      - 지수 $x$를 위수 $m$으로 나누었을 때, 몫을 $q$, 나머지를 $r$이라고 하면 $x = q \cdot m + r$ 이 된다. (이때 $r$이 바로 $x \bmod m$ 이다.)
      - 이것을 거듭제곱 수식에 대입하면 $a^x = a^{q \cdot m + r}$
      - 지수 법칙에 의해 수식을 쪼개면 $a^{q \cdot m + r} = (a^m)^q \cdot a^r$
      - 앞서 확인한 오일러 정리에 의해 $a^m = 1$ (항등원)이므로 $1^q \cdot a^r = 1 \cdot a^r = a^r$
      - 따라서 거대한 지수 $x$는 날아가고, 나머지인 $r$(즉, $x \bmod m$)만 남게 된다.
     
    - 정리
      - 우리가 쓰는 집합 $\mathbb{Z}_N^*$ 의 위수 $m$은 오일러 파이 함수 $\phi(N)$ 이다.
      - 따라서 Base(밑수) $a$는 모듈로 $N$으로 크기를 깎고, Exponent(지수) $x$는 모듈로 $\phi(N)$ 으로 크기를 깎는다.
     
      - 주의 사항
        - 당연히 **$a \in G$**여야 한다.
          - 다시 말해, 밑수 $a$와 모듈로 $N$이 서로소가 아니라면 사용해선 안된다.
         
    - Ex) $5^{74} \bmod 21$
      1. $G = \mathbb{Z}_{21}^* = {1, 2, 4, 5, 8, 10, 11, 13, 16, 17, 19, 20}$, $m = 12$
      2. 지수 74를 위수 12로 나눈 나머지로 깎아낸다. $74 \bmod 12 = 2$
      - 따라서 $5^{74} \bmod 21$ = $5^2 \bmod 21$ = 4
        
- Algorithms on numbers
  - 암호학에서 시간을 재는 법
    - 일반적인 알고리즘에서 덧셈이나 곱셈을 할 때 Time(연산 시간)을 $\mathcal{O}(1)$로 취급한다.
    - 하지만 앞서 봐왔듯 암호학에서 다루는 숫자는 $2^{512}, 2^{1024}$ 처럼 수백~수천 비트짜리 초거대 숫자이다.
    - 따라서 암호학에서는 연산 시간을 숫자의 크기가 아니라 숫자를 이진수로 표현했을 때의 비트 길이($|a|$)를 기준으로 측정한다.
      
  - (Extended) gcd
    - EXT-GCD(확장 유클리드 호제법, 확장 유클리드 알고리즘)
      - 배경
        - 앞서 $a \cdot b \equiv 1 \pmod N$ 이 되는 역원($b$)을 무조건 찾을 수 있다고 배웠다.
        - 그 역원을 찾아주는 것이 EXT-GCD이다.
 
      - 기존 유클리드 호제법: 두 수 $a$와 $N$의 최대공약수 $\gcd(a,N)$ 만 구해준다.
      - EXT-GCD(확장 유클리드 호제법): 최대공약수 $r$을 구해줄 뿐만 아니라, $r = a \cdot u + N \cdot v$ 를 만족하는 계수 $u$와 $v$까지 함께 찾아준다.
     
      - 정의(EXT-GCD)
        - 입력: 두 정수 $a$와 $N$
        - 출력: $(r, u, v)$ 세 개의 값

        - $r = gcd(a, N) = a \cdot u + N \cdot v$
          - $a$와 $N$의 최대공약수 $r$을 $a$의 $u$배와 $N$의 $v$배를 더한 형태로 표현할 수 있게 해주는 계수 $u$, $v$를 변환하는 함수이다.
        
    - Lemma(보조정리)
      - 그렇다면 EXT-GCD는 계수 $u$와 $v$를 어떻게 찾아낼까?

      - $(q, r) = \text{INT-DIV}(a, N)$ 일 때, $\gcd(a, N) = \gcd(N, r)$ 이 성립한다.
        - 거대한 숫자의 최대공약수 문제를 훨씬 작은 숫자의 문제로 축소시켜 준다. 컴퓨터는 나머지가 0이 될 때까지 이 Lemma를 반복적으로 사용하여 답을 찾아낸다.
 
      - Ex) $a=5, N=12$ 일 때, $5$의 역원 구하기 (13/44)
        - $5 \cdot u + 12 \cdot v = 1$ 을 만족하는 계수 $u$ 찾기
        - 과거 장부: $r_0=12, u_0=0, v_0=1$ / 현재 장부: $r_1=5, u_1=1, v_1=0$
        1. $12$를 $5$로 나눈다. (몫 $q = 2$)
           - 새로운 나머지: $r_2 = 12 - (2 \times 5) = \mathbf{2}$
           - 새로운 $u$ 계수: $u_2 = 0 - (2 \times 1) = \mathbf{-2}$
           - 새로운 $v$ 계수: $v_2 = 1 - (2 \times 0) = \mathbf{1}$
        - 장부를 한 칸씩 과거로 밀어낸다: 현재 장부가 $r_1=2, u_1=-2, v_1=1$ 이 됨
        2. $5$를 $2$로 나눈다. (몫 $q = 2$)
           - 새로운 나머지: $r_2 = 5 - (2 \times 2) = \mathbf{1}$ (최대공약수 도달!)
           - 새로운 $u$ 계수: $u_2 = 1 - (2 \times -2) = \mathbf{5}$ (역원 발견!)
           - 새로운 $v$ 계수: $v_2 = 0 - (2 \times 1) = \mathbf{-2}$
        - 결과 검산: $5 \cdot (\mathbf{5}) + 12 \cdot (-2) = 25 - 24 = 1$. 완벽하게 성립한다.

    - Modular Inverse(모듈로 역원)
      - 전제 조건: $a$와 $N$은 서로소여야 한다. (즉, $\gcd(a, N) = 1$)
      - 수식 전개 과정
        1. 앞선 예시처럼 EXT-GCD를 돌리면 $a \cdot u + N \cdot v = 1$ 이 되는 계수 $u, v$를 반환한다.
        2. 이 수식의 양변에 모듈로 $N$을 씌워본다.
        3. $N \cdot v$는 $N$의 배수이므로 모듈로 세상에서는 $0$이 되어 날아간다.
        4. 결과적으로 $a \cdot u \equiv 1 \pmod N$ 이 남는다.
      - 결론: EXT-GCD가 찾아준 계수 $u$ 에 모듈로 $N$을 취한 값이 바로 우리가 찾던 완벽한 역원($a^{-1}$)이다!

  - Exponentiation
    - Square-and-Multiply Exponentiation Algorithm
      - 배경: 암호학에서는 $a^n \bmod N$ 을 계산할 때 지수 $n$이 수백 비트에 달할 정도로 매우 크다. 무식하게 곱하면 연산 횟수가 기하급수적으로 늘어나($\mathcal{O}(2^{|n|})$) 계산이 불가능하다.

      - 해결책: 지수를 Binary(이진수)로 변환하여 계산 횟수를 획기적으로 줄인다.
     
      - 작동 원리(Left-to-Right 방식)
        1. 지수 $n$을 이진수로 바꾼다. (예: $107 = 1101011_2$)
        2. 이진수의 왼쪽(가장 큰 자리)부터 오른쪽으로 한 비트씩 읽으며 반복한다.
        3. Square (제곱): 비트를 이동할 때마다 결과값을 무조건 제곱($y \leftarrow y^2$)한다.
        4. Multiply (곱셈): 현재 읽은 비트가 **'1'**이라면, 밑수 $a$를 한 번 더 곱해준다($y \leftarrow y \cdot a$). '0'이면 넘어간다.
      - 결과: 지수가 아무리 커도 이진수의 비트 길이($|n|$)만큼만 루프를 돌면 되므로, 계산 시간이 $\mathcal{O}(|n|)$ 으로 극적으로 단축된다.
