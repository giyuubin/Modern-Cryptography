- Cumputational Number Theory
  1. DH key exchange: 거대 소수의 필요성
  2. Safe Prime
  3. Fermat's Little Theorem
---
1. DH key exchange
   - 용어 정리:
     - Primitive Root(생성원 또는 원시근)
       - Ex: mod 19에서 a = 7을 거듭제곱 (p = 19)
         - => $7^1 = 7$, $7^2 = 49 \equiv 11(mod 19)$, $7^3 = 77 \equiv 1(mod 19)$
         - => 주기 = 3
       - 하지만 어떤 특정한 수들은 거듭제곱 했을 때 1~p-1까지의 모든 정수를 단 한 번씩 중복없이 모두 생성해낸다
       - Ex: mod 19의 경우 2, 3, 10, 13, 14, 15가 이에 해당됨 // 이들은 $\phi(n) = 18$ 제곱해야 1이 나옴
       - 이처럼 모듈러 집합 내의 모든 고유한 값을 빠짐없이 생성해내는 수를 primitive root라 한다
     - Discrete Logarithms(이산대수)
       - p가 소수이고, a가 p의 원시근일 때, 주어진 값 b에 대하여 b 
