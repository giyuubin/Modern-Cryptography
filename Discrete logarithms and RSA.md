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
    - p가 소수이고 a가 p로 나누어떨어지지 않는 양의 정수이면(즉, $gcd(a, p) = 1$),
      $a^{p-1} \equiv 1 \pmod{p}$이 항상 성립한다
    - 증명:
      - $1a \times 2a \times 3a \times \cdot \cdot \cdot \times (p-1)a \equiv [1 \times 2 \times \cdot \cdot \cdot \times (p-1)] \pmod{p}$
      - $a^{p-1}(p-1)! \equiv (p-1)! \pmod{p}$
      - (p-1)과 p는 서로소
      - $\therefore$ $a^{p-1} \equiv 1 \pmod{p}$











