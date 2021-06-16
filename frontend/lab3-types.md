---
layout: default
parent: Frontend
title: Lab 3 (second part) - Type checker
nav_order: 4
---
# Lab 3 (second part) - Type checker

In this lab we will write a type checker for our tiger compiler.
The type checker is responsible for the following tasks:

- assigning types to expressions and declarations lacking them;
- checking types consistency.

For example, after running the typer, the expression

```
let var a := 3
    var b := a
in
  let var c := a + b in print_int(c) end
end
```

will be typed as *void* and will be displayed as

```
function main(): int =
  (
    let
      var a: int := 3
      var b: int := a
    in
      let
        var c: int := (a + b)
      in
        print_int(c)
      end
    end;
    0
  )
```

Note that main always return an *int*: its return value is used by the operating system as the program exit code (0 meaning that everything worked fine – indeed if we reach the end of main, we had no error).

Setup
-----

You will work in the same directory as the first part of lab 3. You do
not need to retrieve and unpack an archive for this part.

You do not need to do any particular setup.

Adding the type checker file and command-line argument
------------------------------------------------------

The type checker is a visitor similar to the binder. It **must** be
located in `src/ast/type_checker.hh` (and `src/ast/type_checker.cc`
if needed, which will probably the case) in order to be picked up
by the automated grader.

You must add a `--type/-t` command to the driver which will
run your AST tree through the type checker. Once you have added
this command like argument, the `--help` output should look like:

```
$ src/driver/dtiger --help
Options:
  -h [ --help ]         describe arguments
  --dump-ast            dump the parsed AST
  -b [ --bind ]         run the binder on the parsed AST
  -t [ --type ]         run the type checker on the parsed AST
  --trace-parser        enable parser traces
  --trace-lexer         enable lexer traces
  -v [ --verbose ]      be verbose
  --input-file arg      input Tiger file
```

The binder must be run before the type checker even if
the user does not explicitly give `--bind` on the command line,
as it makes no sense to type check the tree if it has not been decorated.

Also, in order to see the results of the type checker, the dump of the AST
(if requested) must take place after the type checker has run.

For interoperability purpose with later labs, the class of the type checker
must be `TypeChecker` and be declared in namespace `ast::type_checker`.

Implementing the type checker
-----------------------------

The type checker must perform the following operations:

- `IntegerLiteral` and `StringLiteral` nodes are given respectively
  the `t_int` and `t_string` types.
- `Sequence` nodes have the same type as their last expression
  if they have one, otherwise they are `t_void`.
- `IfThenElse` nodes must have type-compatible branches,
  whose type become their type.
- `Let` expressions have the same type as their last expression
  if they have one, otherwise they are `t_void`.
- `VarDecl` nodes without an explicit type (in the `type_name` field) take their type from
  their expression. This type cannot be `t_void` since a variable
  is either an integer or a string.
- `VarDecl` nodes with an explicit type given in the source
  must check that the expression (if any) is compatible with this type,
  which will become the type of the node.
- `BinaryOperator` nodes must check that their arguments have
  types compatible with the operation and with each other, and
  set the resulting type.
- `Identifier` nodes inherit the type of their declaration.
- `Assign` nodes must check the validity of the assignment
  and be `t_void` themselves.
- `WhileLoop` nodes must have integer conditions and a body of type
  `t_void`, and are themselves `t_void`.
- `ForLoop` nodes must have integer bounds, an integral index, and
  a body of type `t_void`, and are themselves `t_void`.
- `Break` nodes are `t_void`.
- `FunDecl` nodes have either an explicit type (in the `type_name` field)
  which matches the type of their expression, or no explicit type in which
  case their expression and themselves must be `t_void`.
- `FunCall` nodes inherit the type of their declaration. The type checker
  also ensures that number of arguments matches the number of parameters,
  and that they all have the right type.

Note that in some cases you might analyze a `FunCall` targeting
a `FunDecl` which has not yet been analyzed. It may happen in two
cases:

- you are calling a primitive function, and primitives are not located
  in the tree;
- you are calling a function defined after the current one in the
  same declarative block (mutually recursive functions).

In those cases, you might note that the target has not been analyzed
because its type is still `t_undef`. When this happens, you should
recurse and analyze the declaration of your target.

It also mean that before analyzing a `FunDecl` you must make sure
that it has not been analyzed yet, as it may have been analyzed
during the processing of a `FunCall` in the second case described
above. In this case, the visit of the `FunDecl` should not have
any effect since it has been done already.

Note on automated test results
------------------------------

Since by default the binder accepts anything without checking
the types consistency, some type checker tests may appear to pass very
early. This is due to the fact that some constructs are valid and will
not be rejected. However, as you add more type checking, some tests
that were passing may start to fail because you are rejecting valid
constructs.

Optional: implementing the escaper
----------------------------------

This part will not be tested automatically, but must be implemented if you
want to be able to use your own code in the next lab.

You need to build yet another visitor, the escaper, which will run right
after the binder. Its role is, for every `VarDecl` which escapes (this flag
has already been set by the binder), to add it to the current function
`get_escaping_decls()` vector.

This is necessary so that when we build the frames for the functions, we
can put the escaping variables in the frame structure (since they might
be accessed from outside the function and need to be stored in memory),
while other variables may be stored separately and put into registers
as they will never be read or modified outside the function by nested functions.

So your work is simple:

- When visiting a `FunDecl` node, save the current function, set the
  current function to the one you are visiting, recurse, then restore
  the current function.
- When visiting a `VarDecl` node, if it escapes, add it to the list of
  escaping declarations for the current functions (and recurse into its
  expression if any).
- For every other node, recurse in case you encounter a `FunDecl` or
  `VarDecl` somewhere lower in the tree.

For interoperability purpose with later labs, the class of the escaper
must be `Escaper` and be declared in namespace `ast::escaper`.
