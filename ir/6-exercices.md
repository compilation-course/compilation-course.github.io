---
layout: default 
title: LLVM IR Exercices
parent: Intermediate Representation 
nav_order: 5
---

# LLVM IR Exercices

Consider the following LLVM IR code:

```llvm
%ft_main = type {}

@0 = private unnamed_addr constant [2 x i8] c"N\00"
@1 = private unnamed_addr constant [2 x i8] c"P\00"

define i32 @main() {
entry:
%frame = alloca %ft_main
%s = alloca i32
br label %body

body:
store i32 -4, i32* %s
br label %while_test

while_test:
%0 = load i32, i32* %s
%1 = icmp slt i32 %0, 4
%2 = sext i1 %1 to i32
%3 = icmp ne i32 %2, 0
br i1 %3, label %while_body, label %while_end

while_body:
%4 = load i32, i32* %s
%5 = icmp slt i32 %4, 0
%6 = sext i1 %5 to i32
%7 = icmp ne i32 %6, 0
br i1 %7, label %if_then, label %if_else

while_end:
ret i32 0

if_then:
call void @__print(i8* getelementptr inbounds ([2 x i8], [2 x i8]* @0, i32 0, i32 0))
br label %if_end

if_else:
call void @__print(i8* getelementptr inbounds ([2 x i8], [2 x i8]* @1, i32 0, i32 0))
br label %if_end

if_end:
%8 = load i32, i32* %s
%9 = add i32 %8, 2
store i32 %9, i32* %s
br label %while_test
}
```


1. Annotate each block/line of code with a comment describing its purpose.
2. What does this program do ? Can you write an equivalent pseudo-code in Tiger ?
3. Rewrite the previous code in SSA form. The new code should not use any call to alloca. You can do minor optimizations (like removing the unused frame), but you should preserve the current structure of the control-flow-graph (CFG).
