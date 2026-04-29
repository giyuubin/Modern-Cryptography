- Discrete logarithms and RSA
  - Cyclic groups and discrete logarithms 2
    - Generators and cyclic groups
    - Discrete Logarithms 4 
  - Finding cyclics groups
    - Examples of groups
    - DL and CDH games 5
    - Choosing/Building groups of the form $Z_p^*$
  - DH key exchange: 거대 소수의 필요성 6
  - Safe Prime 3
  - Fermat's Little Theorem 1
---
- Mathematical Foundations
  - Fermat's Little Theorem
  - Cyclic Groups and Generators
  - Safe Prime
- Discrete Logarithm Problem(DLP)
  - Discrete Logarithm Problem
  - DF key exchange and CDH games
  - Large Prime
---
- Mathmatical Foundations
  - Fermat's Little Theorem
    - 배경:
      - 현대 암호학, DH나 RSA와 같은 공개키 암호 체계는 컴퓨터 메모리의 한계로 인해 숫자가 무한히 커지는 것을 방지해야 한다.
      - 따라서 연산의 결과가 항상 일정한 범위 내에 머물도록 모듈러 연산을 기반으로 하는 Finite Field(유한체) 및 Multiplicative Group(곱셈군)을 설계한다.
      - 이때 암호학적 공간이 안전한 닫힌 우주가 되기 위해서는, 임의의 숫자를 계속 거듭제곱하더라도 언젠가는 출발점(항등원 1)로 되돌아오는 순환성이 반드시 수학적으로 보장되어야 한다.
      - 페르마의 소정리는 소수 $p$를 법으로 하는 모듈러 공간에서 "어떤 수를 몇 번 거듭제곱해야 완벽히 한 바퀴를 돌아 항등원 1로 회귀하는가?"에 대한 해답을 제시한다.
      - 이 정리를 통해 우리는 암호 시스템이 동작하는 곱셈군 $Z_p^*$의 전체 원소 개수(Order)가 $p-1$개로 고정되어 있음을 확정할 수 있으며, 이는 이산 대수 문제(DLP)의 탐색 공간을 정의하는 핵심 근거가 된다.
        
    - p가 소수이고 a가 p로 나누어떨어지지 않는 양의 정수이면(즉, $gcd(a, p) = 1$),
      $a^{p-1} \equiv 1 \pmod{p}$이 항상 성립한다
    - 증명:
      - $1a \times 2a \times 3a \times \cdot \cdot \cdot \times (p-1)a \equiv [1 \times 2 \times \cdot \cdot \cdot \times (p-1)] \pmod{p}$
      - $a^{p-1}(p-1)! \equiv (p-1)! \pmod{p}$
      - (p-1)과 p는 서로소
      - $\therefore$ $a^{p-1} \equiv 1 \pmod{p}$
    - 증명에 대한 대수학적 해석:
      - 위의 증명식의 본질은, $Z_p^*$ 집합의 모든 원소 $p$와 서로소인 $a$를 곱하여 모듈러 연산을 수행하면, 그 결과값들이 겹치지 않고 오직 순서만 바뀐 채 원래의 $Z_p^*$ 집합 전체를 그대로 재현한다는 전단사(일대일 대응, 역함수가 존재) 사상을 의미한다.
      - 앞차시 Group Theory 관점에서 이를 더욱 간결하게 일반화한다. 위수가 $m$인 임의의 $G$와 군에 속한 원소 $a$에 대하여, 항상 $a^m = id$(항등원)가 성립한다.
      - $Z_p^*$는 0을 제외한 ${1, 2, ..., p-1}$의 집합이므로 위수 $m = p-1$이며, 항등원은 1이다.
      - 따라서 대수학적으로 $a^{p-1} \equiv \pmod{p}$가 필연적으로 성립하게 된다.











