- Discrete logarithms and RSA
  - Cyclic groups and discrete logarithms
    - Generators and cyclic groups
    - Discrete Logarithms 
  - Finding cyclics groups
    - Examples of groups
    - DL and CDH games
    - Choosing/Building groups of the form $Z_p^*$
  - DH key exchange: 거대 소수의 필요성
  - Safe Prime
---
- Discrete Logarithms (이산 로그 문제와 대수학적 한계)
  - 순환 군 내 이산 로그(DLP)의 수학적 정의
  - 해독 알고리즘의 계산 복잡도(Time Complexity)와 타원 곡선 암호(ECC)의 도입 배경
- DL and CDH Games (암호학적 난제 공식화)
  - 이산 로그 게임(DL Game)의 의사코드(Pseudocode)와 어드밴티지
  - 전산적 Diffie-Hellman(CDH) 문제의 정의와 상호 환원성
- Finding Cyclic Groups (안전한 순환 군 구축)
  - 부분군 공격 방어를 위한 안전 소수(Safe Prime) 탐색
  - 확률론적 소수 판별 알고리즘과 소수 밀도 정리(Density of Primes)
---
- **Discrete Logarithms**
  - 배경:
    - 우리는 특정 원소 $g$를 거듭제곱하여 전체 공간을 빠짐없이 생성할 수 있을 때, 그 $g$를 Generator(생성자)라고 정의하며 이러한 군을 Cyclic Group(순환군)이라 했다.
    - DLP는 이 순환 군의 특성을 역으로 이용한 대수학적 함수이다.

  - 정의:
    - Order가 $m$인 순환 군 $G = \langle g \rangle$와 군의 생성자 $g \in G$가 존재한다고 가정
    - 이때 군 $G$에 속하는 임의의 원소 $a$에 대하여, 다음 방정식을 만족하는 고유한 지수 $i \in Z_m$이 반드시 단 하나 존재한다.
      
$$g^i = a$$
    
  - 이 수식을 만족하는 지수 $i$를 'Base(밑) $g$에 대한 $a$의 Discrete Logarithm(이산 로그)'라고 정의하며, 다음과 같이 표기한다.

$$i = \text{DLog}_{G,g}(a)$$
    
  - 대수학적으로 이산 로그 함수는 지수화 함수의 완벽한 역함수로 작동한다. 따라서 다음 두 가지의 항등식이 항상 성립한다.
    - $\text{DLog}_{G,g}(g^i) = i$  for all $i \in Z_m$
    - $g^{\text{DLog}_{G,g}(a)} = a$ for all $a \in G$ 
---










