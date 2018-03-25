# __VolgaCTF 2018 Quals__ 
## Folden Antelope

## Information
**Category:** | **Points:** | **Writeup Author**
--- | --- | ---
Crypto | 350 | MiKHalyCH

**Description:** 

> Last year we lost millions of dollars on our casino. Today we suggest that you play in an updated one.
<br>[casino_server.py](casino_server.py)
<br> golden-antelope.quals.2018.volgactf.ru:8888

## Solution
Main complexity of this task is prediction of three "random" generators. But wee don't need to find them all. `RA` and `RB` depends on `RX`.

```py
def H(state):
    return int("".join(map(str, state[-1:-9:-1])), 2)
``` 
From this part we can understand that number from each generator is reversed part of 8 last bits converted to int.

After each step new number is `(last_number & 0b01111111) << 1 + {{0,1}, depends on moved bit}`

Server can give us 29 numbers(`k[]`) for predicting. It means that we can restore at least 36 bits of each generator (8 from last states and 28 shifted before).

Now we can make some "system of equations" like:
```
(RX[0] + L[RB[0]] + L[RB[0]]) MOD 256 = k[0]
(RX[1] + L[RB[1]] + L[RB[1]]) MOD 256 = k[1]
...
(RX[i] + L[RB[i]] + L[RB[i]]) MOD 256 = k[i]
...
and so on
```

My way to solve it:
1) Brute all triples, that make k[0].
2) Generate from them all possible tripples.
3) Filter them to select some of tham, that make k[1]
4) Loop steps 2 and 3 (changing k[i] to k[i+1])

It gives us sequence of triples, that was generated but server. They can restore seeds of server generators definitely.

Full algo of restoring is in [solver.py](solver.py)