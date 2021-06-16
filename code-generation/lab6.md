---
title: Lab 6 - runtime
parent: Code Generation
nav_order: 6
---
# Lab 6 - Runtime

In this lab, we will write the runtime so that our
compiler can interact with the underlying operating
system.

Setup
-----

Retrieve the lab code and commit it to your git with the following commands,

```bash
$ mkdir lab6
$ cd lab6
!ifeq(!INSTITUTE)(TELECOM)(
$ curl <some link, see below> | tar zxvf -
)!ifeq(!INSTITUTE)(UVSQ)(
$ wget -qO- www.sifflez.org/lectures/compil/lab6/dragon-tiger.tar.gz | tar zxv
)
$ git add -f dragon-tiger/
$ git commit -m "Import dragon-tiger for lab6"
```

!ifeq(!INSTITUTE)(TELECOM)(
The link to use depends on your operating system. Visit [this page](https://rfc1149.net/tmp/lab6/)
for more information.
)

Now let us build the project,

```bash
$ cd dragon-tiger
$ ./configure
$ make
```

(as in the previous labs, you might need to use the `--with-llvm=` argument to
`configure` to indicate where your LLVM development files are located if
they are not installed in the default system location)

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

$ echo "print_int(42)" | src/driver/dtiger -i --dump-ir -
; ModuleID = 'tiger'
source_filename = "tiger"

%ft_main = type {}

define i32 @main() {
entry:
  %frame = alloca %ft_main
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
the current archive contains pre-built working compiler parts (so as not
to give you the full solution) and the runtime may be done separately.

As you may have done in the previous lab, you can reuse your own code by
importing the `src/parser`, `src/ast` and `src/irgen` directories and
adjusting `src/Makefile.am` (don't forget to put `parser` first in
`SUBDIRS`) and `configure.ac`.

You might also want to propagate your evaluator code although this will not be needed.

**Warning: the automated tests for this lab are most likely to fail if you do not use your own code (except if by chance you use the same LLVM version as the tester does, which is right now LLVM 7.0.1). The pre-built working compiler parts are only useful to run tests locally. Fortunately, only simple function calls will be used (no frames, no assignments, etc.), so your labs 4 and 5 do not need to be 100% completed.**

What is the runtime?
--------------------

You might remember that in lab3 we registered several primitives with the binder, such
as `print_int` to print an integer or `ord` to get the ASCII code of a character. You
might want to look them up in your `lab3/dragon-tiger/src/ast/binder.cc`. We registered
them so that they could be used without being defined explicitly in Tiger code,
hence their "primitive" designation.

However, primitives must now be implemented in order to be found at link time. They
cannot be implemented in Tiger, as we must interact with the underlying operating
system.

In this lab, we will implement all those primitives in C++ in order to build a
`libruntime.a` which will then be linked with the object code produced from the
output of our Tiger compiler. Since we are using a POSIX system (interfaces with
the operating system are well-defined), we will create a runtime which works on
any POSIX system in directory `src/runtime/posix`.

(@) In `src/runtime/posix/runtime.h`, look at the various prototypes of the functions
you will have to implement. We chose to implement the runtime in C because the
C programming language allows to write code whose binary representation drags
less dependencies than C++ and are found everywhere. Since this is a "toy" compiler,
you are allowed to leak memory in the runtime. For example, it may be practical to
call `malloc()` to allocate memory to build a string, even if the memory is not
freed afterwards. In a complete compiler, we would use a garbage collector to free
entities that are no longer referenced.

Compiling and linking
---------------------

A new script `compile` is available at the top-level of the compiler once you have
configured it. You can give it a tiger program or "-" to denote standard input,
and it will produce an executable `a.out` containing your Tiger code linked with
the runtime library.

Try it with:

```bash
$ echo "print_int(42)" | ./compile -
$ ./a.out
UNIMPLEMENTED __print_int
```

At this stage, the program does not work yet, since you haven't implemented the
`print_int` primitive in the runtime.

The `compile` program is a shell script performing those steps:

- Compile your Tiger program into LLVM IR using the `dtiger` compiler with `--dump-ir`.
- Optimize your Tiger program using `opt -O3` (from LLVM), which runs various optimization passes.
- Generate assembly code from the LLVM IR using `llc` (from LLVM).
- Assemble the assembly code into an object file using `gcc` (which in turn calls `as`).
- Link the object file and the runtime library using `gcc` (which in turn calls `ld`).

Don't hesitate to read `compile` and look at the various arguments used if you want to understand what happens behind the scene.

Implement the runtime
---------------------

(@) Implement the runtime in file `src/runtime/posix/runtime.c`. You might want to do it in the order described in the header file to get the best out of the automated tests.

If you want, you might implement another runtime for other systems (such as an Arm-based microcontroller), but you will have to interact with the environment using, for example, the semihosting capabilities which let you exchange data with a monitor program.
