---
layout: default
title:  "CSAPP learning notes"
date:   2018-02-18 15:10:01 -0600
categories: learning_notes CSAPP
---

========================== 2018/02/26 ===========================
- truncate numbers to `k` bits
  - unsigned: `num % (2 ^ k)`
  - two's complement: `U2T(num % (2 ^ k))` - be aware of negative numbers with modulo.
- unsigned addition:
  - `unsigned x, y of w bits`
  - `x + y <> 2 ^ w`: normal situation, no overflow. `x + y >= sum`
  - `2 ^ w <= x + y < 2 ^ (w+1)`: overflow, `x + y < sum`
  - this is basically raw addition with a truncate for overflow
  - principle for overflow detection:
  ```c
  int is_overflow(unsigned x, unsigned y) {
    unsigned sum = x + y;
    return sum < x;
  }
  ```

========================== 2018/02/26 ===========================
- unsigned to two's complment conversion
  - T2Uw(x):
    - x < 0: `2^w + x`
    - x >= 0: `x`
  - U2Tw(x):
    - x <= TMax: `x`
    - x > TMax: `x - 2^w`
  - In C
    - explicitly: casting, ex:
      ```c
      unsigned x = 12;
      (int) x
      ```
      this could also happen with `printf`
    - implicitly:
      - assign expression with one type to another:
      ```c
      unsigned x = 12;
      int y = x;
      ```
      - promotion rule: when two sides of the operator have different type, convert to unsigned, this also works the same for arithmetic operators:
      ```c
      2 < 8u // this will be converted to unsigned
      ```
  - bit extension
    - unsigned: zero extension - padding left with zeros
    - signed: sign extension - padding left with sign bit
    - both operations preserve the original value
    - combine type conversion and shifting, we could write some functions to extract bits with either zero extension or sign extension. Examples on __P116__

========================== 2018/02/25 ===========================
- unsigned numbers
  - Umin: 0b0000...0000 (all zeros)
  - Umax: 0b1111...1111 (all ones)
- signed numbers (two's complement)
  - MSB is sign bit, has negative weight
  - Tmin: 0b10000...0000 (all zeros EXCEPT first / sign bit)
  - Tmax: 0b01111...1111 (all ones EXCEPT first / sign bit)
- casting(type conversion between the two)
  - At least in C, the conversion happens at bit level
  - For both U2T and T2U, it converts to bit patterns first and use target way of interpreting bits

========================== 2018/02/18 ===========================
- shift operations
  - left: drop left bits, padding right bits with `0`
  - right:
    - logical: padding left with `0`
    - arithmetical: padding left with `MSB(most significant bit)`
  - General approach for dealing with shifting greater than word size, example 35 or 66: __doing modulo on word size__.
- Common data types
  - __1 byte = 8 bits = 2 hex__
  - char type: 1 byte
  - short type: 2 bytes
  - int: 4 bytes
  - long: 4 bytes / 8 bytes
  - 32t: 4 bytes
  - 64t: 8 bytes
  - NOTE: be aware of the difference on word size: 32 vs. 64
