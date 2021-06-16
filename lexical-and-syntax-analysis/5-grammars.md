---
layout: default
title: Context Free Grammars
parent: Lexical and syntax analysis 
mathjax: true
nav_order: 3
---

## Limited Expressivity for finite automata

DFA (Deterministic Finite Automata) are very efficient at parsing regular
languages. Unfortunately the expressivity of regular languages is limited.

For example, the language $$L_{ab} = {a^nb^n}$$ with $$n \geq 1$$ is not regular.

Indeed, a DFA has a finite number of states. Recognizing words in $$L_{ab}$$
requires keeping track of the number of $$a$$ symbols to ensure that it matches
the number of $$b$$ symbols. Since $$n$$ is arbitrarily large, this is not possible
with a finite number of states. 

More formally, this can be proved by contradiction with the _pumping lemma (Lemme de l'étoile)_.

***
**Proof**

Suppose $$L_{ab}$$ is regular. 

The _pumping lemma_ tells us that there is a $$n \geq 1$$ such that every string 
$$w \in L_{ab}$$ with $$|w| \geq n$$ can be written as $$w = xyz$$ satisfying

- $$|y| \geq 1$$
- $$|xy| \leq n$$
- $$\forall k \geq 0, xy^kz \in L_{ab}$$

Let us consider $$w = a^nb^n$$ which belongs to $$L_{ab}$$ and is longer than $$n$$.
Therefore $$a^nb^n$$ can be decomposed as three substrings $$xyz$$ satisfying above
properties. Because $$|xy| \leq n$$, $$x$$ and $$y$$ are only composed of $$a$$ symbols.

Let us consider $$w'=xy^2z$$. Since $$y$$ is only composed of $$a$$ symbols, $$w'$$ has
exactly $$n$$ symbols $$b$$ and strictly more than $$n$$ symbols $$a$$. Therefore $$w'$$
does not belong to $$L_{ab}$$, contradicting the pumping lemma. 

***

Most programming languages include recursively nested constructions. For
example nested parenthesis, such as `((3+(7/9)) - 2)` or nested control blocks
such as `if a then (if b then c else d) else f`. These constructs with
arbitrarily deep recursive nesting cannot be parsed with DFA, as we just saw.

Since we cannot use DFA, parsers will use the more expressive formalism of
_Context Free Grammars_.

## Context Free Grammars

### Definition
A context-free grammar $$G$$ is defined by four elements, $$G=(V, S, \Sigma, P)$$, where

1. $$V$$ is the finite alphabet of the _non-terminal_ symbols.
2. $$S$$ is the _start non-terminal symbol_ and belongs to $$V$$.
3. $$\Sigma$$ is the finite alphabet of the _terminal_ symbols. $$V$$ and $$\Sigma$$ are disjoint.
4. $$P$$ is a finite set of grammar rules. $$P \subset V \times (V \cup \Sigma)^*$$

A grammar rule is usually written, with $$\alpha \in V$$ and $$\beta_1, \beta_2, \ldots, \beta_k \in (V \cup \Sigma)$$, as

$$ \alpha \rightarrow \beta_1 \beta_2 \ldots \beta_k $$.

In the above expression,

- $$\alpha$$ is a non-terminal.
- $$\beta_1, \ldots \beta_k$$  are either terminals or non-terminals. 

***
**Example**

Consider a grammar $$G_1$$ with

1. $$V = \{S, T\}$$
2. $$S$$ start symbol
3. $$\Sigma = \{a,b,\#\}$$
4. The following $$P$$ set of grammar rules,

$$ 
\begin{aligned}
S &\rightarrow T\#\\
T &\rightarrow aTb \\
T &\rightarrow ab \\
\end{aligned}
$$

***

### Applying grammar rules 

When applying a grammar rule, we replace inside a word, the left-hand-side
symbol $$\alpha$$ by the right-hand-side symbols $$\beta_1\ldots\beta_k$$.

Let us have two words $$u,v \in (V \cup \Sigma)^*$$.  We will say that $$v$$ is
produced from $$u$$, when $$u$$ is obtained by replacing left-hand-side symbol in
$$u$$ by its right-hand-side counterpart in a grammar. We will note $$u
\Rightarrow v$$.

In case of one or more rule applications, $$u_1 \Rightarrow u_2 \Rightarrow \ldots v$$, we will say that $$v$$ derives from $$u$$ and we will write $$u\stackrel{*}{\Rightarrow} v$$.

---

**Example**

Consider grammar $$G_1$$ from previous example.

The word, $$aaabbb\#$$ derives from $$S$$, indeed
$$
\begin{aligned}
S &\Rightarrow T\# &&\text{(by applying first rule)} \\
T\# &\Rightarrow aTb\# &&\text{(by applying second rule)} \\
aTb\# &\Rightarrow aaTbb\# &&\text{(by applying second rule)} \\
aaTbb\# &\Rightarrow aaabbb\# &&\text{(by applying third rule)} \\
\end{aligned}
$$

---

Context Free Grammars are called _context free_ because rules can be applied
independently of the context, that is to say the neighboring symbols.

### Language recognized by a grammar

We will say that the language $$L(G)$$ recognized by a grammar $$G$$ is the set of
all words composed of terminal symbols that derive from $$S$$.

$$ 
L(G) = \{w \in \Sigma* | S \stackrel{*}{\Rightarrow} w \} 
$$

For instance, in the previous example, $$L(G_1) = L_{ab}$$

### Parse Trees and ambiguous grammars

Given grammar $$G_2$$,

1. $$V = \{S, E\}$$
2. $$S$$ start symbol
3. $$\Sigma = \{num, \#\}$$
4. The following $$P$$ set of grammar rules,

$$ 
\begin{aligned}
S &\rightarrow E\#\\
E &\rightarrow E + E\\
E &\rightarrow num \\
\end{aligned}
$$

Let's consider the word, $$num + num + num$$.
This word belongs to $$L(G_2)$$ because 
$$ 
S \Rightarrow E\# \Rightarrow E+E\# \\
  \Rightarrow E+num\# \Rightarrow E+E+num\# \\ 
  \Rightarrow E+num+num\# \Rightarrow num+num+num\# 
$$

This first derivation can also be represented as a _Parse Tree_,

~~~
Parse tree for first derivation
           S
           |
           E#
           |                       ((num+num)+num)
         E + E#
        /     \
      E + E   num
      |   |
     num num 
~~~

Interestingly, in this grammar a second derivation also produces this same word.
$$
S \Rightarrow E\# \Rightarrow E+E\# \\
  \Rightarrow num+E\# \Rightarrow num+E+E\# \\ 
  \Rightarrow num+num+E\# \Rightarrow num+num+num\# 
$$

~~~
Parse tree for second derivation
           S
           |
           E#
           |                       (num+(num+num))
         E + E#
        /     \
       num   E + E  
             |   |
            num num 
~~~

When the same word admits multiple parse trees, we will say that the grammar is
ambiguous.

## Backus-Naur Form (BNF)

A common notation to represent grammars is the [Backus-Naur Form](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form#Introduction).  
This notation will be used during lab 2. 

