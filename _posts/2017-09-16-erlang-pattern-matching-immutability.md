---
layout: default
title:  "Some Erlang learning notes"
date:   2017-09-16 16:16:01 -0600
categories: learning_notes erlang
---

### Immutability
- All the values are immutable by default, variables or literals
- variables can be assigned exactly once
- better consistence and predictability comes with the price of less flexibility and (maybe?) some efficiency penalty

### Pantten matching
- `=` operator
- essentially does the compare operation
- only succeed if both sides are the same
- if either side is unbound and the comparison is successful, the unbound side will be assigned with the value from the other side
- So I can think of this as an assignment operation with a safety check to always make sure the equality.
- Used closely with immutability: pattern matching and immutability is the basis of each other and also replys on each other to accomplish their functionality

### Example
```erl
>> Temperature = {celsius, 23.45}.
>> {kelvin,T} = Temperature.
  # exception will be raised, since kelvin doesn't match celsius
```
- This safety guard makes sure that temperature of different unit system will not be assigned / used incorrectly. 
- tuple with the first element as atom is also called `tagged tuple`, which seems to be a good way of passing data around.
