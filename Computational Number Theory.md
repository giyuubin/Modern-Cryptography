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
         - 어떤 수를 N으로 나누었을 때 나올 수 있는 모든 나머지 집합
       - $\mathbb{Z}_N^*$ = {a $\in \mathbb{Z}_N : gcd(a, N) = 1}
         - $\mathbb{Z}_N$중에서 N과 최대공약수가 1인(즉, 서로소인) 애들만 따로 뽑아낸 집합
       - $\varphi(N)$ = $|\mathbb{Z}_N^*|$
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

