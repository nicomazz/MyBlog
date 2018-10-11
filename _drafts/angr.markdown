---
layout: single
title:  Practical C symbolic execution with angr 101
date:   2018-10-11 11:23:39 +0100
categories: programming
toc: true
comments: true
header:
  image: /assets/images/milky_way_lagazuoi.jpg
  teaser: /assets/images/milky_way_lagazuoi.jpg
---


# Overview
Excited for the symbolic analysis but do not know where to start? This is a good place.
Here will be covered a really small and insignificant part of what symbolic execution is and how you can use it. Basically, here you will find the way in which I usually use it for solving some types of CTF problems (and not only). 
Other really usefull resources are:
//todo add angr example
Though, in my opinion, it's not easy to start from these resources. After reading this tutorial, you are encouraged to read them!
I'm pretty sure that there are better way to do all the things I'll write here, so if you have better solutions comment down here please!

> If you don't want to read all and only jump to the practical part, click here // todo add link to tutorials



## Motivation
One of the most overlooked problem of the recent times is client-side security. It happens more and more often that critical parts of the function of a service are carried on client-side, behind alleged protections as code obfuscation or flow graph flattening. 
// todo image of graph flattening
Often the real problem for someone how is trying to do reverse-engineering is to deal with an incredible high complexity in the flow graph. Analyze all the graph could be a really time expensive task, carried on only by expert reverser. The use of tools in these situation can make our life way more easier. 




## Types of Analysis
There are two standard types of analysis, and a third one, not so much standard, witch will be covered here. 
**Static Analysis** is meant a type of analysis performed in a non-runtime environment, mainly by inspecting program code. Opposed to this come the **dynamic analysis**, in which all the work is done while the program is in operation. Breakpoints are usually used in this situation. This last type of analysis can provide a lot of more informations, like the state of system memory during runtime and the effective functional behaviour. Some programs may unpack themself during the execution. In this situation, dynamic analysis is the only solution.
The third one is **Symbolic analysis**, which describe the computation creating a set of constraints associated with each flow graph node. After these symbolic expressions have been created, we can extrapolate useful information not available before, manly thanks to constraint solver engines.





## The problem
If we drive into reverse-engineering, the challenging question is often “How can I reach this piece of code?”. A static analysis of the binary is not always enough. Sometimes a mixed approach with static and dynamic analysis is needed. Anyway, the number of times in which even these two technique are vain is really high. 

### An example
Let’s imagine a simple application that only requests an input from the user. Based on this, the flow branch in thousand of different execution path. Let’s image that we have found a security issue in a specific “line of code” somewhere in the application, and we want to find a way to get there. At this time we know where we get the input, we know where we want to arrive, and the elaboration performed by the application. The fact is that these middle elaboration can be really challenging to analyze and understand. What we need is a way to discover the input that can carry us to our target. To do exactly this, we can use Symbolic execution.


## Angr brief description
Prerequisited: //todo list of pre
The things that “Angr” framework can do are practically infinite, so here is done a selection of the most interesting ones. The following discussion will be mainly focused on binary exploitation.
Before drive deeper in the things that Angr allow us to do, is needed a little understanding of how it works “under the wood”.

## How It works under the wood

The first thing to talk about is Boolean Satisfiability Problem (SAT). In a really simple words, the problem states “Does there exist an interpretation that satisfies a given boolean formula?”. A small example is the boolean formula $$A or B$$ . An interpretation is a value for A and B. A non-satisfiable example of boolean formula is $$A and (not A)$$.

In general, it is an NP-complete problem, so we don’t have efficient algorithms to solve it.
Now, we can replace our binary values with expressions like predicates, to obtain an instance of SMT (Satisfiability modulo theories). By predicates we can imagine linear inequalities, Arrays, functions or bit vectors. Some examples are:
$$2x+4y-z \le 5 and y = x+2$$
A way to “solve” this kind of SMT problems is by using “Z3, a theorem prover from Microsoft Research. It has a python API. In the following snippet we will solve the previous sample:
```python
# pip install z3-solver
from z3 import *
x = Int('x')
y = Int('y')
z = Int('z')
solve(2*x + 4*y - z <= 5,y == x+2) #output: [z = 0, x = -1, y = 1]
```
You can find a great course on z3 here: //todo add link

Advances in SMT solvers enabled us to use these kind of technique to analyze executions path of binaries. The next image show us a concrete example.

//todo add image with caption

If we put in the constraint solver the symbolic expression of a node, we can find the input needed to reach the specific node.


## Angr installation
```bash
pip install angr
```
For detailed instruction on what to do with the problem you'll find, refer to the github page //todo add gihub link


## Tutorials 

### Resolve code constraints: finding a special value over a lot of "if" conditions
Let’s start with a really simple example that can demonstrate the angr potential. 
```c
#include "stdio.h"

void target_function(){
   printf("opening shell");
   system("/bin/sh");
}
void wrong_function(){
   printf("WRONG!");
}
void f(int a){
   if ( a  != 10){
       if (a*2 < 86){
           if (a > 11 && a < 12412){
               if(a > -1){
                   if ((a << 2) > 5 && a*425 + 86 > (a^1) + 40){
                       if (a * 2 - 1 > 82 ){
                           printf(":) ");
                           target_function();
                           return;
                       }
                   }
               }
           }
       }
   }
   printf(":( ");
   wrong_function();
}
int main(){
   int a;
   scanf("%d",&a);
   f(a);
   return 0;
}
```
To build and execute the code you can use: `gcc  -g -o main main.c && ./main`
In this case the binary is not stripped, so it keep the name of the functions for an easier address retrieval. Usually this is not the case, and we have to manually analyze it to find out the correct addresses. To do this we can use gdb, r2 or IDA (all of them have free versions).
We have to exploit the program by calling “target_function”, which opens a shell and allow us to execute commands. As we can see, it’s not so trivial to perform this task by hand. We have a lot of constraint to solve, and this process could be really time consuming, if not totally impossible (not in this situation). Here is where symbolic execution comes to help us.

1. Firstly we find the addresses of the target_function and wrong_function.
2. Then we create a simulation_manager (`simgr` in the code). The aim of it is to construct the various states by the symbolic execution of the code. We can think as state the nodes of our flow graph, like in the previous image. So, each state contains a list of constraint to be solved to reach it. Differently from the previous example, where the subjects of the various constraints included only variables, this time are involved a really big number of things: each state contains all the information related to the execution, like memory, registers and files, and assign to each one of them a symbolic variable.
3. We start the exploration of the graph, starting from the main function, avoiding all the branches that carry to the wrong function, and stopping when we finally find out target function.
4. At this time, the simulation manager contains different states. Conceptually each one assigned to a different graph node. One of them will be the state of execution once arrived at the target function.
5. Now we have to use the constraint solver to obtain a real value for the input. stdin is the file with number 0, stdout with number 1. So we read this value from the last state. The output will be the solution of the constraint solver.

```python
#!/usr/bin/env python2
# -*- coding: utf-8 -*-

import angr
import claripy

def get_func_addr(name):
   return p.loader.find_symbol(name).rebased_addr

p = angr.Project("./main",
                load_options={'auto_load_libs': False})
               
to_find = get_func_addr('target_function')
to_avoid = get_func_addr('wrong_function')
start_address = get_func_addr("main")

#our initial state of exploration
st = p.factory.entry_state(addr=start_address)  
#the simulation manager will construct all the constraints
simgr = p.factory.simgr(st)
simgr.explore(find=to_find, avoid=to_avoid)
print simgr

found_state = simgr.found[0]
# let's concretize stdin and stdout
print "output:", found_state.posix.dumps(1)
print "input:", found_state.posix.dumps(0)
```

If we run the previous script, we found this output:
```bash
$ ./expl.py
WARNING | 2018-10-02 11:27:33,669 | angr.analyses.disassembly_utils | Your verison of capstone does not support MIPS instruction groups.
WARNING | 2018-10-02 11:27:33,814 | cle.loader | The main binary is a position-independent executable. It is being loaded with a base address of 0x400000.
<SimulationManager with 1 found, 5 avoid>
output: :)
input: +0000000042
```

So let’s try to use this input:
```
$ ./main
+0000000042
sh-4.4$ echo "pwned!"
pwned!
```

As we can see, the number 42 fulfill all the previous condition, and make the binary to spawn a shell.  


### Serial key cracking

Let’s examine a real life example. It consists in the mechanism of serial key validation of an old computer game.
The binary used by this tutorial can be found here: //todo add url
Let’s try to execute our binary:

```
$ ./starcrack         
Please enter your 13-digit CD-key located on the back of your StarCrack CD case.
CD-key: 123
You entered an invalid CD-key.
```