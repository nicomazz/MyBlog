---
layout: single
title:  Take an L - CSAW 2018 writeup
date:   2018-09-16 01:14:12 +0100
categories: programming 
comments: true
---

The description of the challange was

```
Take an L
Fill the grid with L's but avoid the marked spot for the W

nc misc.chal.csaw.io 9000
```

With this pdf as description
![]({{"/assets/images/take_an_l_description.png" | absolute_url}})


I've found the inspiration in this stackoverflow answer: ()[https://math.stackexchange.com/questions/461252/proof-a-2n-by-2n-board-can-be-filled-using-l-shaped-trominoes-and-1-monomi]

A very fast copy-paste of the first aswer:
>This is an old chestnut of combinatorial geometry. The proof is a fairly simple induction. We show that the 2n×2n board can be covered by trominoes except for one square.
>If n=1, the solution is trivial.
>Otherwise, assume that we can cover a 2n−1×2n−1 board with trominoes except for one chosen square. Divide the 2n×2n board into four 2n−1×2n−1 square quadrants. One quadrant contains the square we want to leave uncovered by trominoes, and by induction we can cover this quadrant, except for the one square, with trominoes.
>For the remaining three quadrants, cover each of these except for one of its corners with trominoes. Rotate the three quadrants so that their uncovered corners lie together at the center of the board. These three remaining squares can then be covered with one more tromino.
>I first saw this in Polyominoes by Solomon W. Golomb. it appears on page 5 of the revised edition, Princeton University Press, 1994.

# The final script, that you can't find on internet

```python
#!/usr/bin/python2
# -*- coding: utf-8 -*-
from pwn import *

# rotate coordinates 90deg clockwise
def rot90(coords, dim):
    result = []
    for coord in coords:
        r, c = coord
        result.append((c, dim-r-1))
    return result

# r,c
up_dx = [(0, 0), (1, 0), (1, 1)]
dw_dx = rot90(up_dx, 2)
dw_sx = rot90(dw_dx, 2)
up_sx = rot90(dw_sx, 2)

def rot90_array_of_array(L_coords, dim):
    return [rot90(coords, dim) for coords in L_coords]

def L_with_offset(coords, start_r, start_c):
    return [(r+start_r, c+start_c) for (r, c) in coords]

# 0 | 3
# -   -
# 1 | 2
def get_quadrante(r, c, dim):
    if r < dim/2:
        if c < dim/2:
            return 0
        else:
            return 3
    else:
        if c < dim/2:
            return 1
        else:
            return 2

# firstly rotate, then shift
def L_group_rotate_shift(L_group_coords, dim, n_rotation, st_r, st_c):
    rotated = L_group_coords
    for i in range(n_rotation):
        rotated = rot90_array_of_array(rotated, dim)
    shifted = [L_with_offset(L_coords, st_r, st_c) for L_coords in rotated]
    return shifted

# fill a 2**N matrix keeping a hole in the top left corner
def fill2N(N, st_r, st_c, w_r, w_c):  
    dim = 2**N
    w_quadrante = get_quadrante(w_r, w_c, dim)

    if N == 1:
        if w_quadrante == 0:
            return [L_with_offset(up_sx, st_r, st_c)]
        if w_quadrante == 1:
            return [L_with_offset(dw_sx, st_r, st_c)]
        if w_quadrante == 2:
            return [L_with_offset(dw_dx, st_r, st_c)]
        else:
            return [L_with_offset(up_dx, st_r, st_c)]

    # let's put the hole in the top left quadrant
    for i in range(w_quadrante):
        w_r, w_c = rot90([(w_r, w_c)], dim)[0]

    # let's resolve the other 3 easy quadrants
    fill_prec_N = fill2N(N-1, 0, 0, 0, 0)

    result = []
    result += L_group_rotate_shift(fill_prec_N, dim/2, 0, dim/2, dim/2)
    result += L_group_rotate_shift(fill_prec_N, dim/2, 1, dim/2, 0)
    result += L_group_rotate_shift(fill_prec_N, dim/2, 3, 0, dim/2)
    # Let's add the L to complete the tris
    result += [L_with_offset(up_sx, dim/2-1, dim/2-1)]

    # let's fix the square with the hole
    last_fix = fill2N(N-1, 0, 0, w_r, w_c)

    result += last_fix
    # return back to the original coordinates
    for i in range(4-w_quadrante):
        result = rot90_array_of_array(result, dim)
        w_r, w_c = rot90([(w_r, w_c)], dim)[0]

    return result


io = remote('misc.chal.csaw.io', 9000)

print io.recvuntil("block: (")
w_r, w_c = io.recvuntil(")").split(",")
w_r = int(w_r)
w_c = int(w_c[:-1])

print "wr,wc=(", w_r, w_c, ")"

result = fill2N(6, 0, 0, w_r, w_c)
print len(result)
for L in result:
    io.sendline(",".join(["("+str(a)+","+str(b)+")" for a, b in L]))
print io.recvline()
io.interactive()
```

## Execution result
```
[+] Opening connection to misc.chal.csaw.io on port 9000: Done
each L shaped block should be sent in its
own line in a comma separated list
eg: (a,b),(a,c),(d,c)

grid dimensions 64x64
marked block: (
wr,wc=( 37 29 )
1365


[*] Switching to interactive mode
flag{m@n_that_was_sup3r_hard_i_sh0uld_have_just_taken_the_L}
[*] Got EOF while reading in interactive
$
```