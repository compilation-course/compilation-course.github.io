---
layout: video
title: Basic Blocks 
ytid: BJALZ-kxp38
slides: 3-basic-blocks.pdf
parent: Intermediate Representation 
nav_order: 5
---

{: .note }
> **Note on LLVM Syntax (Video vs. Modern LLVM)**
>
> In the video (recorded with an earlier LLVM version), pointers are typed (`i32*`).
> In modern LLVM (LLVM 18 used in this course), all pointers are **opaque** (`ptr`). Additionally, unconditional branches explicitly require the label prefix: `br label %target`.
>
> The downloadable slides have been updated to reflect the modern LLVM syntax.
