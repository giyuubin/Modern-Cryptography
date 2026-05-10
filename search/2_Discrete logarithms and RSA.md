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

$$i = \text{DLog}_{G,g}(a)$$ // [군 $G$의 환경에서, 밑(Base)이 $g$일 때 $a$에 대한 이산 로그 값]
    
  - 대수학적으로 이산 로그 함수는 지수화 함수의 완벽한 역함수로 작동한다. 따라서 다음 두 가지의 항등식이 항상 성립한다.
    - $\text{DLog}_{G,g}(g^i) = i$  for all $i \in Z_m$
    - $g^{\text{DLog}_{G,g}(a)} = a$ for all $a \in G$ 
---
- Ex: 모듈러 11 환경에서의 이산 로그 역산 전개
  - <img width="617" height="145" alt="image" src="https://github.com/user-attachments/assets/9bc25879-d734-47f8-b35a-72dc731e86bc" />
  
    - 곱셈군 $Z_{11}^*$를 가져와 이산 로그를 수치적으로 매핑.
    - 앞 시간에 이 군의 위수는 $m = 10$이며, 원소 $g = 2$는 $Z_{11}^*$의 생성자임을 증명했다.

    - 우리의 목표:
      - 주어진 원소 $a$에 대하여 $2^i \equiv a \pmod{11}$을 만족하는 지수
        $i \in Z_{10}$ $(0 \le i \le 9)$를 역산하여
        <img width="232" height="37" alt="image" src="https://github.com/user-attachments/assets/fcacdf33-546a-40fc-8b46-4ef07b54f4ef" />

    - 역산 계산:
      - $a = 1$일 때: $2^0 = 1 \equiv 1 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(1) = 0$
      - $a = 2$일 때: $2^1 = 2 \equiv 2 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(2) = 1$
      - $a = 4$일 때: $2^2 = 4 \equiv 4 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(4) = 2$
      - $a = 8$일 때: $2^0 = 8 \equiv 1 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(8) = 3$
      - $a = 5$일 때: $2^0 = 16 = (11 \times 1) + 5 \equiv 5 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(5) = 4$
      - $a = 10$일 때: $2^0 = 32 = (11 \times 2) + 10 \equiv 10 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(10) = 5$
      - $a = 3$일 때: $2^0 = 256 = (11 \times 23) + 3 \equiv 3 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(3) = 8$

    - 이러한 모듈러 지수 연산을 수행하면 결과값 $a \in$ {1, 2, 3, 4, 5 ,6 ,7 ,8 ,9 ,10} 전체가 지수 $i \in$ {1, 2, 3, 4, 5, 6, 7, 8 ,9}와 중복 없이 완벽하게 1:1로 대응됨을 확일할 수 있다.








