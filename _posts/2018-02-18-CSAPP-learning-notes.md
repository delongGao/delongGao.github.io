---
layout: default
title:  "CSAPP learning notes"
date:   2018-02-18 15:10:01 -0600
categories: learning_notes CSAPP
---

========================== 2018/03/03 ===========================
- two's complement addition
  - it's using the same bit pattern for addition as unsigned and then at the last step convert the unsigned result to two's complement. Based on the book, for lots of systems, they are actually using the same machine instruction for signed and unsigned addition.
  - `Tadd = B2U(T2B(x) + T2B(y))`
  - One very important property of this is: if you are performing Tadd for multiple two's complement integers, it only performs the conversion at the last step, which means you could have overflows for intermediary results, but still get the same results at the last step. This is why `x + y - y` is always going to be `x`, because even if `x + y` is possible to overflow, it will not be truncated in the middle
  - Detecting overflow
    - two types of overflows:
      - negative overflow: `x >= 0, y >= 0, x + y < 0`
      - positive overflow: `x < 0, y < 0, x + y > 0`
    - detecting: basically detecting none of above two types overflows happened
  - Special case for `Tmin`:
    - one of the exercise question:
    ```
    what's the problem of using two's complement add overflow detection to detect overflows of two's complement subtract?
    ```
    - the problematic value in this case is `Tmin`, the reason is: `-Tmin = Tmin`. This could be explained in the following ways:
      - Approach 1: for two's complement `-x = ~x + 1`, in the case if `Tmin`, for example of `w = 4`:
      ```
      Tmin(w=4) = x = 0b1000
      ~x = 0b0111 (Tmax)
      ~x + 1 = 0b1000 = Tmin
      ```
      - Approach 2: by definition, x + (-x) = 0, for w = 4, possible value range is - 2 ^ 3 ~ 2 ^ 3 - 1, `Tmin` = `-8`, the only possible value to meet the condition of `x + (-x) = 0` is `Tmin` itself (with a overflow).
    - as a result, when using Tmin to detect subtract overflow, with two `Tmin`s, add overflow will be true, whereas subtract overflow should be false.


========================== 2018/03/01 ===========================
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
