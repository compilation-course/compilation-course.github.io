---
layout: video
title: LLVM IR
ytid: chv7ao-0bAw 
slides: 2-llvm-ir.pdf
parent: Intermediate Representation 
nav_order: 5
---

{: .note }
> **Note on LLVM Opaque Pointers (Video vs. Modern LLVM)**
>
> In the video (recorded with an earlier LLVM version), pointers are typed (e.g., `i32*`, `i8*`).
> Since LLVM 15 (and in LLVM 18 used in this course), all pointers are **opaque** and written simply as `ptr` (for example, `store i32 5, ptr %ptr` and `load i32, ptr %ptr`).
>
> The downloadable slides have been updated to reflect the modern LLVM syntax.
