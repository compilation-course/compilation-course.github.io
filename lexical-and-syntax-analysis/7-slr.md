---
layout: default
title: Introduction to SLR(1) Parsers
parent: Lexical and syntax analysis 
nav_order: 3
---

# Pushdown automata

A pushdown automaton (PDA) is build by extending finite automaton with a stack,

~~~
                 .------------------.
Input ------->   | Finite automaton |   -------> Accept / Reject input
                 .------------------.
                          |
                          v
                     |_________|
                     |_________|
                     |_________|
                     |_________|
                     |_________|

                        Stack
~~~

A PDA reads the input symbol by symbol. At each read, the PDA can:

- transition the finite automaton to another state by looking at the read
  character and the stack.

- push or pop elements into the stack

The finite automaton has a set of accepting states. If at the end of the input,
the automaton is in an accepting state, then the input is accepted. Otherwise,
the input is rejected.

# SLR(1) Parsers

There are many different types of parsers for Context Free Grammars. In this
course, we will concentrate only on one specific parser SLR(1), which is a
simplified version of the LALR parser used by Bison. 

SLR(1) belongs to the family of bottom-up parsers: that means that it builds a
parse tree starting from the leafs and tries to reach the root start symbol.
Because SLR is a bottom-up parse, it will use grammar rules on the other
direction since it begins from the final word and tries to build the derivation
backwards towards $S$.

SLR(1) stands for **S**imple **L**eft-to-right **R**ightmost-derivation Parser with 1 symbol lookahead.
Let's unpack that:

- Left-to-right means that it reads the input symbol by symbol from left to right
- Rightmost-derivation means that the parser will always apply grammars rules starting from the right. That is to say, rules will be applied to the symbols more recently read.
- 1 lookahead, means that it looks only at the first character in the input to decide what action to take. (It can also look at the stack). 

SLR(1) parsers use a PDA to parse context free grammars.
At each step, the PDA can take two actions:

- **shift**, which will read a symbol from input and push it to the stack

- **reduce**, which will apply one grammar rule backwards and replace a set
  of symbols in the stack, by their left-hand-side in the grammar rule.

After a shift or reduce, we will transition to a new state.

***
**Example**

Consider $G_3$, with $V=\{S, E\}$ and $\Sigma=\{id,\#\}$, with rules

0. $S \rightarrow E\#$
1. $E \rightarrow id\ E$
2. $E \rightarrow id$

This grammar can parse words like $id\ id\#$ which belong to $L(G_3)$.

The automata is represented by the following table,

|   | id | #      | E |
|---|----|--------|---|
| 0 | s1 |        | 2 |
| 1 | s1 | r2     | 3 |
| 2 |    | s4     |   |
| 3 |    | r1     |   |
| 4 |accept|accept|accept|

The first column, contains the states of the automaton. The initial state is 0.
The first line, contains the different symbols (terminals and non terminals).
Each cell in the table, describes the actions to take.

Let us show how this SLR(1) would parse input $id\ id\#$.
Initially, the input contains the word to parse and the stack contains the initial state (state 0). 

~~~
Step 0:
         v
input = [id id #]
         v
stack = [0]
~~~

To know which action to take, we look to the state that is on top of the stack (here 0) and we look at the first symbol in input (here $id$). The above parse table, shows $s1$ for $(0,id)$; this means we will shift and move to the state 1.

Shifting means that we take the first symbol in the input and push it on the stack.
Then we push the new state (here 1) to the stack

~~~
Step 1:
         v
input = [id #]
              v
stack = [0 id 1]
~~~

Now we look at the table for state 1 and symbol $id$, the action is again to shift and move to state 1.

~~~
Step 2:
         v
input = [#]
                   v
stack = [0 id 1 id 1]
~~~

Here we look at the table for state 1 and symbol $\#$, the action is r2. This means to reduce by grammar rule 2.

Grammar rule 2 is $E \rightarrow id$, we therefore remove the right-hand-side ($id$) from the stack and associated states; and replace it by the left-hand-side ($E$). Remember that in this bottom-up-parser we reverse grammar rules. 

~~~
stack = [0 id 1 id 1]
                ----
                |
                v
stack = [0 id 1 E]
              ^ ^   
~~~

After this replacement, we look at the top of the stack and read state and symbol, here $1\ E$. The table tells us that when we reduce to $E$ in state 1, we should move to state 3.

~~~
Step 3:
         v
input = [#]
                  v
stack = [0 id 1 E 3]
~~~

Here we look at the table for state 3 and symbol $\#$, the action is r1.
Now we reduce by grammar rule 1, which is $E \rightarrow id\ E$.
We replace the right-hand-side by the left-hand-side.

~~~
stack = [0 id 1 E 3]
           --------
           |
           v
stack = [0 E]
         ^ ^   
~~~

The table tells us that a reduction to $E$ on state 0, transition the automaton
to state 2.

~~~
Step 4:
         v
input = [#]
             v
stack = [0 E 2]
~~~

The table tells us that for state 2 and symbol $\#$ we should shift to state 4,
which accepts the word !

***

# How to build an SLR(1) Parser 

Now that we understand with an example how the SLR(1) operates. Let's show how to build one on the next video.:w

