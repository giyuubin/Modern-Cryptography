- **목차**
- Discrete Logarithms
  - Mathematical Definition of DLP
  - Computing DLog: $Z_p^*$ vs $EC_p$
  - Hardness Record and Quantum Threats
- DL and CDH Games
  - DL Game: Formal Definition & Advantage
  - CDH Problem & Relationship to DL
  - CDH Game: Advantage Analysis
- Finding and Building Cyclic Groups
  - Mathematical Fact for Group Selection
  - Finding Primes & Safe Prime $(p = 2q + 1)$
  - Primality Testind (Miller-Rabin) & Density
  - Algorithm for Testing Generators
- RSA Trapdoor Permutations
  - RSA Math: Phi Function & Modulo Inverse
  - RSA Function and Trapdoor One-wayness
  - Factoring Problem & Attack Complexity
- PKE and Security Notions
  - Syntax and Correctness of PKE Schemes
  - IND-CPA: Indistinguishability under Chosen-Plaintest Attack
  - IND-CCA: Indistinguishability under Chosen-Ciphertest Attack
  - Security Hierarchy: IND-CCA vs IND-CPA
---
- **Discrete Logarithms (이산 로그)**
  - 이름의 유래:
    - Logarithm (로그): 일반적인 수학에서 로그는 지수화(Exponentiation) 연산을 거꾸로 되돌리는 역산 함수이다. (예: $2^3 = 8$일 때, $\log_2 8 = 3$)
    - Discrete (이산적): 암호학에서는 실수 범위가 아닌, 유한한 개수의 원소만 존재하는 '군(Group)' 내부에서만 연산이 이루어진다. 숫자들이 소수 $p$를 법(Mod)으로 하여 불연속적으로 끊어져 존재하기 때문에 '이산(Discrete)'이라는 수식어가 붙는다.
    - 암호학적 중요성: 실수 체계의 로그와 달리, 이산적인 군 내부에서는 지수 연산은 매우 빠르지만 그 역산인 이산 로그를 구하는 것은 극도로 어렵다. 이러한 '일방향성(One-wayness)'이 현대 암호학적 안전성의 핵심 기반이 된다.
---
- Mathematical Definition of DLP
  - 배경:
    - 우리는 특정 원소 $g$를 거듭제곱하여 전체 공간을 빠짐없이 생성할 수 있을 때, 그 $g$를 Generator(생성자)라고 정의하며 이러한 군을 Cyclic Group(순환군)이라 했다.
    - 이산 로그는 이 순환 군의 특성을 역으로 이용한 대수학적 함수이다.

  - 정의:
    - Order가 $m$인 순환 군 $G = \langle g \rangle$와 군의 생성자 $g \in G$가 존재한다고 가정.
    - 이때 군 $G$에 속하는 임의의 원소 $a$에 대하여, 다음 방정식을 만족하는 고유한 지수 $i \in Z_m$이 반드시 단 하나 존재한다.
      
$$g^i = a$$
    
  - 이 수식을 만족하는 지수 $i$를 'Base(밑) $g$에 대한 $a$의 Discrete Logarithm(이산 로그)'라고 정의하며, 다음과 같이 표기한다.

$$i = \text{DLog}_{G,g}(a)$$ 

  - [군 $G$의 환경에서, 밑(Base)이 $g$일 때 $a$에 대한 이산 로그 값]
    
  - 대수학적으로 이산 로그 함수는 지수화 함수의 완벽한 역함수로 작동한다. 따라서 다음 두 가지의 항등식이 항상 성립한다.
    - $\text{DLog}_{G,g}(g^i) = i$  for all $i \in Z_m$
    - $g^{\text{DLog}_{G,g}(a)} = a$ for all $a \in G$ 
---
- Ex: 모듈러 11 환경에서의 이산 로그 역산 전개
  - <img width="433" height="146" alt="image" src="https://github.com/user-attachments/assets/c21e3a57-72bd-4d7c-8bb1-9dba882a9aa8" />
  
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
      - $a = 8$일 때: $2^3 = 8 \equiv 8 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(8) = 3$
      - $a = 5$일 때: $2^4 = 16 = (11 \times 1) + 5 \equiv 5 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(5) = 4$
      - $a = 10$일 때: $2^5 = 32 = (11 \times 2) + 10 \equiv 10 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(10) = 5$
      - $a = 9$일 때: $2^6 = 64 = (11 \times 5) + 9 \equiv 9 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(9) = 6$
      - $a = 7$일 때: $2^7 = 128 = (11 \times 11) + 7 \equiv 7 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(7) = 7$
      - $a = 3$일 때: $2^8 = 256 = (11 \times 23) + 3 \equiv 3 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(3) = 8$
      - $a = 6$일 때: $2^9 = 512 = (11 \times 46) + 6 \equiv 6 \pmod {11}$ 이므로, $\text{DLog}_{G,2}(6) = 9$

    - 이러한 모듈러 지수 연산을 수행하면 결과값 $a \in$ {1, 2, 3, 4, 5, 6, 7, 8, 9, 10} 전체가 지수 $i \in ${0, 1, 2, 3, 4, 5, 6, 7, 8, 9}와 중복 없이 완벽하게 1:1로 대응됨을 확인할 수 있다.

  - Computing Discrete Logs
    - 위수가 $m$인 순환 군 $G = \langle g \rangle$와 생성자 $g \in G$가 주어졌을 때, 임의의 원소 $X \in G$에 대한 이산 로그 값 $x = \text{DLog}_{G,g}(X)$를 찾는 가장 기초적인 알고리즘은 Brute-force(전수 조사)이다.
    - 이 알고리즘은 $g^x = X$를 만족하는 지수 $x$를 찾기 위해, 탐색 공간인 0부터 $m-1$까지의 모든 정수를 순차적으로 대입한다.
    - Pseudocode
      - <img width="302" height="173" alt="image" src="https://github.com/user-attachments/assets/3a479051-c422-42b6-b4a7-fdf357c050a0" />

      - 당연히 탐색 공간 내에 반드시 정답이 존재하므로 수학적으로 항상 올바른 해를 반환한다.
      - 하지만 루프를 1회 수행할 때마다 지수화 연산이 요구되며, 최악의 경우 군의 위수 $m$번만큼 연산을 반복해야 하므로 시간 복잡도는 $O(m)$이 된다.
      - 암호학에서 입력값의 크기는 정수 자체가 아니라 이진수 인코딩의 숫자 p의 비트 길이로 측정되므로, 이 복잡도는 $O(2^{|p|})$이 된다.
        
    - 따라서 $p$가 매우 큰 암호학적 환경에서는 이 역산 알고리즘의 사용이 불가능해지며,
      이것이 Diffie-Hellman 키 교환을 가능하게 하는 이산 로그 문제의 난해성이다.
---
- **Computing DLog: $Z_p^*$ vs $EC_p$**
- Finding Cyclic Groups
  - 특정 유한군이 순환군이 임을 보장하는 데에는 두 가지 Fact가 존재한다.
    - Fact1: Let $p$ be a prime. Then $Z_p^*$ is cyclic.
    - Fact2: Let $G$ be any group whose order $m = |G|$ is a prime number. Then $G$ is cyclic.
  - $|Z_p^*| = p-1$
  - $p$가 2보다 큰 홀수 소수일 때, $p-1$은 항상 짝수이므로 소수가 될 수 없다.
  - 따라서 군의 위수가 소수임을 전제로 하는 Fact2는 Fact1을 증명하거나 함의하지 않으며, 두 정리는 독립적으로 작용한다.

    - Ex: $Z_{11}^*$의 경우:
      - $p = 11$은 소수이므로 Fact1에 의해 $Z_{11}^*$은 순환군이다.
      - 이때 위수 $m = 11 - 1 = 10$이며, 10은 소수가 아니므로 Fact2로는 이 군의 순환성을 설명할 수 없다.
---
- $Z_p^*$ vs $EC_p$
  - 암호 시스템의 안전성은 순환군 내부에서 이산 로그 문제를 해결하는 '최적 알고리즘의 복잡도'에 의존한다.
    - <img width="476" height="190" alt="image" src="https://github.com/user-attachments/assets/9e5cacf3-3ec5-46da-bb84-f4b1a1c53619" />

    - 주로 유한체의 곱셈군인 $Z_p^*$와 유한체 상의 타원 곡선군 $EC_p$가 주로 사용된다.
      - $Z_p^*$의 복잡도: 대략 $e^{1.92(\ln p)^{1/3}(\ln \ln p)^{2/3}}$으로 계산된다.
      - $EC_p$의 복잡도: $\sqrt{p} = e^{\ln(p)/2}$이다.

  - Ex: <img width="626" height="323" alt="image" src="https://github.com/user-attachments/assets/f1d6239d-e5db-4866-868c-877fdca3d0da" />

    - $Z_p^*$의 해독 한계: 2019년 기준으로 약 795비트 크기의 소수 $p$에 대한 이산 로그 계산이 기록 되었다.
    - $EC_p$의 해독 한계: 현재 기록은 약 114비트 수준이다.
  - 이는 동일한 비트 길이에서 타원 곡선의 이산 로그 문제가 훨씬 더 강력한 난해성을 가짐을 시사한다.
---
- Why EC Groups & Moore’s Law
  - 배경:
    - 곱셈군 $\mathbb{Z}_p^*$ 에는 준지수 시간 해독 알고리즘(Index Calculus)이 존재하여 보안 성능이 저하된다.
    - 타원 곡선 군($EC_p$)은 기하학적 점 가산 연산을 채택하여 대수적 해독을 무력화하고, 최적 복잡도를 순수 지수 시간 $\mathcal{O}(\sqrt{p})$ 에 묶어둔다.
    - 연산 능력이 지수적으로 증가한다는 무어의 법칙(18개월마다 2배) 하에서 두 군의 키 크기 확장 효율성은 극명한 격차를 보인다.
  - Ex: 80-bit 보안 수준 및 무어의 법칙 시뮬레이션
    - 1단계: 80-bit 보안을 위한 최소 비트 길이 ($k$) 계산
      - $\mathbb{Z}_p^*$ (준지수 시간 복잡도 함수):
        $$e^{1.92(\ln p)^{1/3}(\ln \ln p)^{2/3}} \approx 2^{80} \implies k = 1024 \text{ bits}$$
      - $EC_p$ (순수 지수 시간 복잡도 함수):
        $$\sqrt{p} = 2^{80} \implies p = 2^{160} \implies k = 160 \text{ bits}$$
    - 2단계: 세제곱 법칙(Cubic Cost) 기반의 연산 비용 비교
      - 모듈러 연산 비용은 비트 길이 $k$ 의 세제곱($\mathcal{O}(k^3)$)에 비례한다.
      - 타원 곡선 군($k=160$)의 연산 비용을 기본 단위 $T = 160^3$ 으로 정의할 때, 곱셈군($k=1024$)의 연산 비용 비율을 구한다.
        $$\frac{1024^3}{160^3} = \left(\frac{1024}{160}\right)^3 = (6.4)^3 = 262.144 \approx 260$$
      - 결론: 동일 보안 수준에서 타원 곡선 군이 곱셈군보다 약 260배 연산 효율이 높다.
    - 3단계: 무어의 법칙($t$년)에 따른 키 크기 확장 속도 비교
      - 공격자 연산력이 $2^{t/1.5}$ 배 증가할 때, $EC_p$ 는 복잡도 매칭을 통해 선형적으로 증가한다.
        $$\frac{k_{new}}{2} = \frac{k_{old}}{2} + \frac{t}{1.5} \implies k_{new} = k_{old} + 1.33t$$
      - 결과 분석: 타원 곡선 군은 매년 약 1.33 비트의 선형 확장만으로 무어의 법칙을 방어하지만, 곱셈군 $\mathbb{Z}_p^*$ 는 준지수 함수의 한계로 인해 초선형적인 폭발적 키 크기 팽창(1024 $\rightarrow$ 2048 $\rightarrow$ 4096-bit)을 강제당한다.











