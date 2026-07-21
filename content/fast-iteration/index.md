+++
title = "Quick tips for fast iteration in Haskell"
description = "Quick tips about tools and techniques for fast iteration when developing Haskell"
date = 2026-07-22
[taxonomies]
authors = ["Tom Ellis", "Laurent P. René de Cotret"]
categories = ["Ecosystem"]
tags = ["Practices", "Tooling"]
+++

WIP notes:

In scope: tools and techniques to help speed type checking or
compilation of an exisitng Haskell codebase

Out of scope: structuring a codebase to compile fast (unless Laurent
think it's really easy to describe to how to do that an act on it. I
prefer out-of-the-box solutions for this article)

## `ghci`-based

* `ghci` itself: type `:r`

  Comes bundled with your GHC installation


* `ghcid`

   `ghcid` is a normal executable package on Hackage, so you can
   install it with, for example `cabal install ghcid`

   eval comments - `-- $> execute this code`

* `ghcid-check`

  It's just a single `bash` script.  Download it from
  <https://github.com/tomjaguarpaw/ghcid-check/>

* Tricorder

   `tricorder` is a normal executable package on Hackage, so you can
   install it with, for example `cabal install tricorder`

  <https://github.com/atelier-hub/tricorder>

## `caabl` based

1. Update to recent `cabal` and GHC, ideally `cabal` 3.12+ and GHC
   9.8+.  This combination offers support for better parallelism using
   the semaphore functionality.

2. Consider an allocation of `cabal build`:

    2.1 Increase the allocation area for GHC itself. This can
    dramatically increase compilation throughput: `cabal build
    --ghc-options="+RTS -A128m -n2m -RTS"`, for example.

    2.2 Disable optimizations if you don't need them: `cabal build
    --ghc-options="..." --disable-optimizations`

    2.3 Take advantage of parallelism with job sharing:

      2.3.1 `cabal build -j8 --ghc-options="" --disable-optimizations`
      allocates 8 cores for compilation

      2.3.2 `cabal build -j8 --semaphore
       --ghc-options=... --disable-optimizations` allocates 8 cores in
       total, with better sharing of resources with GHC
       <https://well-typed.com/blog/2023/08/reducing-haskell-parallel-build-times/>
