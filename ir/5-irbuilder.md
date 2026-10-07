---
layout: video
title: IRBuilder 
ytid: v-5ajBUzAAA 
slides: 5-irbuilder.pdf
parent: Intermediate Representation 
nav_order: 5
---

{: .note }
> **Note on LLVM IRBuilder API (Video vs. Modern LLVM)**
>
> In the video (recorded with an earlier LLVM version), `Builder.CreateLoad(result)` inferred the pointee type from the typed pointer.
> With opaque pointers in modern LLVM (LLVM 18 used in this course), `CreateLoad` requires the loaded type explicitly: `Builder.CreateLoad(llvm_type(ite.get_type()), result)`.
>
> The downloadable slides have been updated to reflect the modern syntax.
