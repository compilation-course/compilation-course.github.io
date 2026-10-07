---
layout: video
title: Frames
ytid: ZCrOneSQ-Xc
slides: 1-frames.pdf
parent: Code Generation 
nav_order: 6
---

{: .note }
> **Note on LLVM Opaque Pointers (Video vs. Modern LLVM)**
>
> In the video (recorded with an earlier LLVM version), frame structures and functions use typed pointers (such as `%ft_main*` or `%ft_main.f*`).
> In modern LLVM (LLVM 18 used in this course), all pointers are **opaque** (`ptr`).
>
> The downloadable slides have been updated to reflect the modern syntax.
