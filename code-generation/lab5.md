---
title: Lab 5 - Static frames
parent: Code Generation
nav_order: 6
---
# Lab 5 - Static frames

In this lab, we will extend our code generator for our
Tiger compiler so that it can access variables declared
outside the current function using the static link.

Setup
-----

This lab must be done by enriching the work done in lab 4. The automated
tests will run as soon as you create (and add in git) a `src/irgen/lab5.check`
file in your `lab4/dragon-tiger` directory. This file can be empty, only
its presence is required.

Frame creation
--------------

▶ Add a new `void IRGenerator::generate_frame()` method which will
be called when analyzing a `FunDecl` (in `irgen.cc`). The steps are:

- Build a vector of the types needed in the frame. If the function has a parent,
  the first field is a pointer onto the parent frame. You can use the `getPointerTo()`
  method on a type (`llvm::Type`) to get a pointer type to it. Other types needed are those
  of the escaping declarations, whose list is accessible through the
  `get_escaping_decls()` method on the `FunDecl` node. The current function is
  stored in the `current_function` field of the `IRGenerator` object.
- Create a new structure named `ft_` followed by the external name (for example
`ft_main.f`) from those types.
- Register this new type into the `frame_type` map which associates a function
  declaration to its frame type.
- Create a new object of the newly created frame type and allocate it on the
  stack, assigning the result to the `frame` field of the `IRGenerator` object.

The insertion point should be set to the entry block right after its creation
before calling `generate_frame()`, in `IRGenerator::generate_function()`,

At this stage, the frame will be created with the right type but will be
empty.

Also, make sure that you to not attempt to create a field for any escaping
`void` value, as those do not (and cannot) require storage.

Finding the right frame
-----------------------

▶ Add a new `std::pair<llvm::StructType *, llvm::Value *> IRGenerator::frame_up(int levels)`
method (in `irgen.cc`) which returns either the current frame information (if `levels` is 0)
or the information of one or several levels above (if `levels` is not 0). The returned pair
contains the frame type and an expression to be able to access it.

A way to do it would be:

- Initialize a variable `fun` with the current function declaration (found in
  `current_function_decl`) and a `sl` value representing the address of the
  current frame (found in `frame`).
- Then, for every level you need to go up, replace `sl` by a load of the
  first field of the frame it currently points to (using `Builder.CreateStructGEP()`
  and `Builder.CreateLoad()`, and `fun` by the declaration of its parent. You
  are not one level up.
- When this is done, you can return a pair made of the frame type of `fun`
  (found in the `frame_type` map) and the current static link from `sl`, which represents
  a succession of loads.

Frame assignment
----------------

▶ It is now time to put escaping variables into the frame. You will
create a new `llvm::Value *IRGenerator::generate_vardecl(const VarDecl &decl)`
method (in `irgen.cc`) which will take care of allocating the variable
either in the frame (if it escapes) or in the stack (if it doesn't escape)
and register its address into the `allocations` map:

- If the variable does not escape, allocate it using `alloca_in_entry`,
  register its address in the `allocations` map which maps a variable
  declaration to the `llvm::Value *` representing its address. You might
  have to modify code you have written already to set the `allocations`
  data here.
- If the variable does escape, you must find its position in the list
  of escaping variables for the current function. This gives you its
  position in the frame structure. Don't forget to add 1 if the current
  function has a parent and thus has reserved index 0 to store the static
  link it received. Do not forget to skip `void` variables either, as those
  are not represented in the frame. This computed index must be recorded
  into the `frame_position` map which associates an escaping variable
  declaration to its position in the frame. Then you can use
  `Builder.CreateStructGEP` with the right frame type (found
  in the `frame_type` map) and the current frame
  (found in the `frame` field of the visitor) to retrieve the address
  of this variable. You must register it in the `allocations` map and
  you can return it.

Now, you can replace your calls to `alloca_in_entry` from lab4 to use
`generate_vardecl` instead. Since the tests in lab4 do not use escaping
variables, this should not break them. However, you might want to
try some code snippets to ensure that escaping variables (and only
them and the static link) are stored into the frame.

Find the variable address
-------------------------

▶ Since a variable may have been created in an outer frame, we must
    use `frame_up` to find its address. Modify `IRGenerator::address_of()`
    in order to:

- return the result of `allocations[&decl]` as before if the variable is used
  at the same depth as its declaration;
- use `frame_up()` and the `frame_position` map to return a pointer to the
  variable in an upper frame otherwise.

Make sure you use `address_of()` and not directly `allocations` in the
visitor for `Identifier` and `Assign`.

Passing the static link
-----------------------

▶ We now must pass the static link to every function which isn't external (primitives
don't need the static link). To do so, we need to do several things:

- Modify the `FunCall` visitor in order to pass the static link to non-external functions as
  the first argument. We will use the recently created `frame_up` method to get the
  right static link. The number of levels to go up is computed from the difference between
  the depth of the call and the depth of the function declaration. We only care for
  the second component of the pair returned by `frame_up`.
- Modify the `FunDecl` visitor if we are analyzing a non-external function. In this
  case, we must insert a parameter which is a pointer to our parent frame type
  before the other arguments.
- Modify `IRGenerator::generate_function()` (in `irgen.cc`) so that it stores the
  first parameter (if we are analyzing a non-external function) into the first
  field of the current frame.
- In the same function, use `generate_vardecl` instead of `alloca_in_entry` to
  store function parameters into local variables. By doing so, you will ensure
  that escaping parameters will get stored in the frame, while non-escaping ones
  will be allocated on the stack (and probably later in physical registers instead).

If everything goes well, at this stage, you will be able to run code which uses
variables defined above the current function.

Congratulations, your compiler is now complete as far as the code generation is
concerned.
