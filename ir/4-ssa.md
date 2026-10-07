---
layout: video
title: Single Static Assignment (SSA) 
ytid: AeuywNhUS5s
slides: 4-ssa.pdf
parent: Intermediate Representation 
nav_order: 5
---

{: .note }
> **Note on LLVM Opaque Pointers (Video vs. Modern LLVM)**
>
> In the video (recorded with an earlier LLVM version), the mem2reg example uses typed pointers (e.g., `i32* %if_result`).
> In modern LLVM (LLVM 18 used in this course), pointers are **opaque** (`ptr %if_result`).
>
> The downloadable slides have been updated to reflect the modern LLVM syntax.
