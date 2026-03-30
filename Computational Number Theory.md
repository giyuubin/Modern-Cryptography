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
      - a mod N = $r \in \mathbb{Z}^+$
      - mod는 +, * 등과 같이 두 개의 인자를 받는(이항) 연산이다

    - Congruences(합동): mod 연산에 대한 설명
      - $a \equiv b \pmod N$ means $a \bmod N = b \bmod N$
        
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
        - $b = 5$ 를 대입하면 $5 \cdot 5 = 25$ 이고, $25 \bmod 12 = 1$ 이 성립하므로 5의 역원은 5이다.
          
  - Exponentiation(거듭제곱)
    - 배경: 그룹 안에서 원소 a를 여러 번 반복해서 연산하는 것을 수학적으로 어떻게 표기할 것인가?
    - Multiplicative notation(곱셈 표기법)
      - 연산 기호가 곱셈 계열일 경우 $a^n$으로 표기한다
      - $a^0 = id$ (항등원, 보통 1)로 정의한다
      - $a^{-1}$ 은 $a$의 역원(inverse)을 뜻하며, $a^{-n}$ 은 역원을 $n$번 곱한 $(a^{-1})^n$ 을 의미한다
      - 
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

