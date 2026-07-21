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

## `cabal` based

Let's now see how to speed up development builds using the [`cabal`](https://www.haskell.org/cabal/) build system. `cabal` has existed for a very long time, and you may be surprised by some of the things it offers!

Speeding up development builds using `cabal` involves three broad optimizations:

* Tuning GHC's runtime options;
* Decreasing the amount of work `cabal` and GHC have to do by disabling compile-time optimizations;
* Increasing resource-sharing for more effective build parallelism.

Before we describe the build optimizations above, it is worth understanding how cabal can be configured. Cabal uses _project_ files to bundle configuration options related to multiple packages. The default configuration file, which is picked up automatically by `cabal`, is `cabal.project`. A typical `cabal.project` file for a project with three packages looks like:

```
packages:
    packageA
    packageB
    packageC
```

Recent versions of cabal allow project files to import other projects files. Therefore, without changing our default `cabal.project` file, we'll create a new configuration file specifically for development builds. Let's call it `cabal.fast.project`:

```
import: cabal.project
```

As we go through build optimizations below, we'll append to our `cabal.fast.project`. Then, a fast build will specify to use `cabal.fast.project`:

```shell
$ cabal build all --project-file=cabal.fast.project
```

That way, we have an easy set-up for fast builds. We could have project files for profiling builds, release builds, etc.

### Tuning GHC's runtime options

GHC is a wonderful piece of technology. In some ways, it might as well be from the future. However, compiling Haskell programs into performant executables is hard work, and GHC is itself a Haskell program. This means we must tune its runtime system to squeeze out more performance.

Take a look at the [GHC 9.14.1 user guide's section on runtime control](https://downloads.haskell.org/ghc/9.14.1/docs/users_guide/runtime_control.html). In our experience, however, simply tuning the garbage collection allocation size is a great first step if your computer is somewhat recent. In particular, increasing the allocation size from the default of 4MB to 64MB, 128MB, er even 256MB may dramatically increase compilation throughput!

Thus, we update our `cabal.fast.project` project file to pass the appropriate flag to GHC. We'll use an allocation area of 64MB, but do experiment on your own workloads:

```
import cabal.project

-- Step 1: tuning GHC
ghc-options: +RTS -A64m -RTS
```

### Disabling compile-time optimizations

Even if GHC is given the appropriate resources to do its job more effectively, it still does a lot of work by default, since GHC's default optimization level is `-O1`. This means that GHC performs *some* optimizations, which necessarily requires more compilation time.

But this is merely the _default_ behavior, and this article is all about speed. Instead, we can disable optimizations via the `optimization` option in cabal project files:

```
import cabal.project

-- Step 1: tuning GHC
ghc-options: +RTS -A64m -RTS

-- Step 2: disable optimizations
optimization: false
```

Note that this optimization flag only applies to your locally-built packages. Your third-party dependencies will be compiled using the default optimization level, `-O1`. This is generally considered acceptable since your third-party dependencies are rarely built, and it allows you to re-use them for other build profiles.

### Better resource-sharing for build parallelism

We've tuned GHC, and minimized the amount of work it needs to do. What is left is speed up the build across _multiple packages_.

`cabal` has an option for parallel builds called `--jobs`. You can either specify a number of capabilities to dedicate to the build (e.g. `--jobs=2`), or let `cabal` decide based on your hardware (using a bare `--jobs`). Higher is not _always_ better; this depends on your build graph!

In our experience, a number from 4 - 8 is appropriate to start with. We therefore update our `cabal.fast.project` project file:

```
import cabal.project

-- Step 1: tuning GHC
ghc-options: +RTS -A64m -RTS

-- Step 2: disable optimizations
optimization: false

-- Step 3: build parallelism
jobs: 6
```

We can do even better! If you use `cabal` 3.12+ in combination with GHC 9.8+, you can take advantage of a new flag, `--semaphore`, which allows `cabal` to share resources better with GHC. While [Well-Typed has a full breakdown of the why and how](https://well-typed.com/blog/2023/08/reducing-haskell-parallel-build-times/), we summarize the result here. In short, if cabal can take advantage of multiple parallel invocations of GHC, it will do so; however, under the `--semaphore` option, if cabal _cannot_ take advantage of all the cores it was allocated, it can tell GHC to use more cores. This can result in compilation time reductions of up to 30%!

We update our `cabal.fast.project` one final time:


```
import cabal.project

-- Step 1: tuning GHC
ghc-options: +RTS -A64m -RTS

-- Step 2: disable optimizations
optimization: false

-- Step 3: build parallelism
jobs: 6
semaphore: True
```

Finally, our builds will be much faster:

```shell
$ cabal build all --project-file=cabal.fast.project
```
