---
layout: default
title: Lab 4 - IR generation
parent: Intermediate Representation 
nav_order: 5
---
# Lab 4 - IR generation

In this lab we will write a code generator for our Tiger
compiler targeting LLVM IR.

The IR code generator is responsible for the following
tasks:

- Creating a IR module in which functions will be defined.
- Creating independent top-level IR functions for every Tiger
  function in the source code, including the nested ones.
- Building stack frames to hold escaping variables, and
  giving pointer to the outer stack frame to every function
  called.
- Building basic blocks to represent the flow of the Tiger
  function.

A completed code generator should never raise an error
related to the source code. All errors should have been
caught in earlier phases already. For example, an unknown
identifier would have been caught and reported by the
binder. A typing inconsistency would have been caught and
reported by the typer.

Setup
-----

Retrieve the lab code and commit it to your git with the following commands,

```bash
$ mkdir lab4
$ cd lab4
$ wget -qO- www.sifflez.org/lectures/compil/lab4/dragon-tiger.tar.gz | tar zxv
$ git add -f dragon-tiger/
$ git commit -m "Import dragon-tiger for lab4"
```

This lab is the first one which requires the headers and libraries for LLVM. You
can usually install them from a `llvm-dev` package. You must use at least version
3.9 and at most version 9.0 of LLVM. Let us build the project:

```bash
$ cd dragon-tiger
$ ./configure
$ make
```

If it doesn't find the LLVM header and libraries, ensure that you have installed them.
If needed, you can use the `--with-llvm=` argument to configure to point it to the
right location.

If everything worked as expected, you should be able to run
the compiler driver `dtiger` as follows,

```
$ src/driver/dtiger --help
Options:
  -h [ --help ]         describe arguments
  --dump-ast            dump the parsed AST
  --dump-ir             dump the generated IR
  -b [ --bind ]         run the binder on the parsed AST
  -t [ --type ]         run the type checker on the parsed AST
  -i [ --irgen ]        run the LLVM IR code generator
  --trace-parser        enable parser traces
  --trace-lexer         enable lexer traces
  -v [ --verbose ]      be verbose
  --input-file arg      input Tiger file
  -o [ --object ] arg   generate object code file

$ echo "print_int(42)" > test.tig
$ src/driver/dtiger -i --dump-ir test.tig
; ModuleID = 'tiger'
source_filename = "tiger"

define i32 @main() {
entry:
  br label %body

body:                                             ; preds = %entry
  call void @__print_int(i32 42)
  ret i32 0
}

declare void @__print_int(i32)
```

Ensure that you test thoroughly and commit each feature. Follow precisely the
instructions, this lab is graded and machine corrected!

Important note
--------------

Some of you may not have completed the previous assignment. To this effect,
the current archive contains a pre-built parser library (so as not
to give you the full solution) and binder and type checker phase
and this lab may be done separately.

However, when you have completed the previous assignment, feel free to
replace the `src/parser/` directory by a copy of the one from your
previous assignment. This will be most satisfactory, as your code from
the first step will be used also in the second step. Do not forget to
add `parser` to the `SUBDIRS` variable in `src/Makefile.am` (it must
be the **first** subdirectory there because it auto-generates files
used in other subdirectories) and in
`src/parser/Makefile` in `configure.ac`. However, you **must**
keep `nodes.hh` as distributed with this lab, because it contains extra
visitors for the AST.

You might also want to propagate your evaluator code although this will not be needed.

Similarly, you can get the `src/ast/` directory from the second lab
as long as you have implemented the binder, type checker and escaper.

Assembly production and IR verification
---------------------------------------

Piping your IR output to the `llc` program will tell you
if there is a typing output in our IR (for example if you try to use a pointer instead
of an integer), and will output Intel assembly code corresponding to the computer
is it currently executing onto.

``` bash
$ src/driver/dtiger -i --dump-ir test.tig | opt -mem2reg | llc
[…intel assembly output…]
```

If you are more familiar with ARM Thumb-2 instruction set, you can obtain it by using
`llc -march=arm -mcpu=cortex-m4` for example.

You can combine this with `opt -mem2reg` to see the effect of removing the unneeded
`alloca` IR instructions:

``` bash
$ src/driver/dtiger -i --dump-ir test.tig | opt -mem2reg | llc -march=arm -mcpu=cortex-m4
[…thumb2 assembly output…]
```


Code organization
-----------------

The IR code generator is called with the `-i` option. It is interesting to
also add the `--dump-ir` option which provides a text dump of the generated IR.

The declarations of the IR code generator can be found in `src/irgen/irgen.hh`.
The implementation is split in two parts:

- `src/irgen/irgen.cc` contains the code to generate new functions, stack
  frames, variable declarations, etc.
- `src/irgen/irgen-visitor.cc` contains the code to transform the AST into IR.

For this lab, you are provided with an incomplete `src/irgen/irgen-visitor.cc`
that you will have to fill according to the instructions. Each `visit` method
must return a `llvm::Value *` which represents the result of the translated
expression. If the expression (or statement) has no useful return value,
`visit` must return `nullptr`.

LLVM Builder
------------

A LLVM IRBuilder object helps you insert code into a basic block. It
remembers the block and the place within the block where it is currently
inserting code (called the *insertion point*). Our `IRGenerator` class
contains a `Builder` field for this purpose.

The IRBuilder contains a lot of methods designed to make code generation
easier. For example, it can move the insertion point to the end of another
basic block, it can generate a branch instruction to another block, or
generate a conditional branch instruction to another block, etc.

(@) Read carefully the provided code and make sure you understand
how the visitor for the `Sequence` AST node works, both for a non-empty and
an empty sequence.

Variables
---------

(@) Using utility methods declared in `irgen.hh`, implement the visitor
    for `VarDecl` nodes. Do not forget to initialize the variable content
    with its initial value if it has one. Look at the IRBuilder provided
    utility to store the initial value at the right address. Also, the
    `llvm::Value *` representing the variable address should be registered
    into the `allocations` map which maps variable declarations to their
    addresses.

To validate your implementation, you can check that the following Tiger
code

```
let var a := 1 in end
```

generates something like

```
define i32 @main() {
entry:
  %a = alloca i32
  br label %body

body:                                             ; preds = %entry
  store i32 1, i32* %a
  ret i32 0
}
```

One can clearly identify the allocation of a `i32` reserved space
onto the stack (whose address is stored into `%a`) in the entry
block of the main function. In the body, the initial value `1`
is stored at address `%a`.

(@) Implement the visitor for `Identifier` nodes.

The following code

```
let var a := 1 in print_int(a) end
```

should now also contain something like:

```
  %0 = load i32, i32* %a
  call void @__print_int(i32 %0)
```

showing that the value stored at `%a` has been read into `%0`
and then passed to function `print_int`.

(@) Implement the visitor for `Assign` nodes.

The following code

```
let var a := 1 in a := 3; print_int(a) end
```

should now also contain something like:

```
  store i32 1, i32* %a
  store i32 3, i32* %a
  %0 = load i32, i32* %a
  call void @__print_int(i32 %0)
```

representing the successive assignments of variable `a`.
Do not worry with the useless initial assignment; once
we will go through the LLVM optimizer, the useless
assignment to `1` will be removed from the final code
automatically and look like:

```
define i32 @main() local_unnamed_addr {
entry:
  tail call void @__print_int(i32 3)
  ret i32 0
}
```

You can see it by yourself by piping the result of the
`dtiger` compiler through `opt -O3 | llvm-dis`:

```
$ src/driver/dtiger -i --dump-ir test.tig | opt -O3 | llvm-dis
```

*Note:* you might need to add some LLVM-specific directory to your `PATH` variable
in order to find the `opt` and `llvm-dis` tools.

Tests and branches
------------------

Translating a `IfThenElse` AST node into IR is simple but
requires several steps. In Tiger, a `if … then … else …`
expression might return a value, as in

```
let var a := if 2 > 3 then 10 else 20 in … end
```

So the various steps needed to translate such a node are:

- Allocate stack space (in the entry block) to represent
  the result (let's call it `%result`).
- Build three new basic blocks: the `if_then` block to
  represent the `then` alternative, the `if_else` block to
  represent the `else` alternative, and the `if_end`
  block as the join point where the execution will
  resume after the `if … then … else …`.
- Evaluate the test condition and either branch to the
  `if_then` block or to the `if_else` block
  depending on the test evaluation (not 0 / 0).
  Do not forget that the conditional branch needs a
  `i1` (boolean) as a parameter, not a `i32`. You
  might want to look for the
  `Builder.CreateICmpNE()` helper function to compare your value
  to the constant 0.
- In the `if_then` block, evaluate the body, assign
  its value to the `%result` variable, and branch to
  the `if_end` block.
- Do the same thing for the `if_else` branch.
- Set the current insertion point to be the
  `if_end` block so that the code generation
  continues from there.
- Load and return the result stored in `%result`.

There is one pitfall however: if the `if …` expression is of type `void`,
  then the `%result` variable must not be
  written to or read from (and `nullptr`
  must be returned by `visit`).

(@) Implement the visitor for `IfThenElse` AST nodes.

Do not forget to test your implementation using tests with
and without `else`, and tests returning either a value or
nothing.

Loops
-----

(@) Implement the visitor for `WhileLoop` without taking care
    of Tiger `break` statements at this time.

(@) Implement the visitor for `Break`. You need to use modify
    the `WhileLoop` and `ForLoop` visitors in order to
    associate the loop AST nodes with their exit blocks
    using the `loop_exit_bbs` map.
