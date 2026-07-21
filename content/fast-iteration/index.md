+++
title = "Quick tips for fast iteration in Haskell"
description = "Quick tips about tools and techniques for fast iteration when developing Haskell"
date = 2026-07-22
[taxonomies]
authors = ["Tom Ellis", "Laurent P. René de Cotret"]
categories = ["Ecosystem"]
tags = ["Practices", "Tooling"]
+++

## *Notes for while we're writing:*

> In scope: tools and techniques to help speed type checking or
> compilation of an existing Haskell codebase
>
> Out of scope: structuring a codebase to compile fast (unless Laurent
> think it's really easy to describe to how to do that an act on it. I
> prefer out-of-the-box solutions for this article)

## `ghci`-based

GHCi is GHC's interactive interpreter (executable name `ghci`) which
comes bundled with every installation of GHC.  It is a REPL
("[read-eval-print
loop](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)")
which means you can type expressions into it and it will run them,
printing the result.  There are a variety of ways to use `ghci` for
fast iteration.

### `ghci` itself

You can invoke `ghci` with a collection of source paths.  Then
`ghci` will compile and load those modules, for example:

```
ghci src/Path/Module1.hs src/Path/Module2.hs
```

or, perhaps more useful, use a `ghci` invocation wrapper that comes
with your build tool, `cabal` or `stack`:

```
stack ghci
cabal repl
```

For a simple `ghci` workflow, make some changes to the files in your
project, then navigate to your `ghci` window and type `:reload` (or
`:r` for short) so `ghci` type checks and compiles your changes,
printing any errors or warnings from GHC.  Do you feel like `:r` ought
to be automated away? If so then check out `ghcid`!

### `ghcid`

`ghcid` is a wrapper around `ghci` that automates issuing `:reload`
when any file in your project changes, so its workflow is easier than
that of `ghci`: make some changes to the files in your project and
then merely *look at* your `ghcid` window; the result of type check
and compilation will appear automatically.

#### Automatically running tests

`ghcid` also allows you to run tests (or indeed any code) when your
source files change.  There are two ways to do this:

1. Embed comments with expressions to be evaluated. For example add
   this comment to a source file to evaluate `expr` after loading:

   ```
   -- $> expr
   ```

   (Evaluating embedded comment expressions requires using the
   `--allow-eval` flag.)

2. Pass an expression to the command line option `--test`, to run
   after the code is loaded successfully, for example:

   ```
   ghcid --test myTestFun
   ```

  (by default the test expression will only run if the code is
  warning-free.  To run even if there are warnings, also pass
  `--warnings`.)

For more information on these features, see the
[Evaluation](https://github.com/ndmitchell/ghcid#evaluation) section
of the `ghci` README.

#### Installation

`ghcid` is a normal executable package on Hackage, so you can install
it with, for example `cabal install ghcid`.

### `ghcid-check`

It's just a single `bash` script.  Download it from
<https://github.com/tomjaguarpaw/ghcid-check/>

### Tricorder

`tricorder` is a normal executable package on Hackage, so you can
install it with, for example `cabal install tricorder`

<https://github.com/atelier-hub/tricorder>

### `ghciwatch`


#### References

* <https://academy.fpblock.com/blog/2018/08/haskell-development-workflows-4-ways/>
* <https://www.parsonsmatt.org/2018/05/19/ghcid_for_the_win.html>
* <https://functor.tokyo/blog/2019-04-07-ghcid-for-web-app-dev>
* <https://haskellweekly.news/episode/6.html>
* <https://mercury.com/blog/announcing-ghciwatch>
* <https://jeancharles.quillet.org/posts/2024-09-04-Haskell-dev-workflow-with-ghcid-and-neovim.html>
* <https://www.well-typed.com/blog/2023/03/cabal-multi-unit/>
* <https://github.com/alexfmpe/semantic-satiation/blob/main/posts/002-cheaper.md>
* <https://ghc.gitlab.haskell.org/ghc/doc/users_guide/ghci.html>
* <https://hackage.haskell.org/package/rapid/docs/Rapid.html>

## `cabal`-based

1. Update to recent `cabal` and GHC, ideally `cabal` 3.12+ and GHC
   9.8+.  This combination offers support for better parallelism using
   the semaphore functionality.

2. Consider an allocation of `cabal build`:

    2.1 Increase the allocation area for GHC itself. This can
    dramatically increase compilation throughput, for example:

    ```
    cabal build --ghc-options="+RTS -A128m -n2m -RTS"
    ```

    2.2 Disable optimizations if you don't need them:

    ```
    cabal build --disable-optimizations
    ```

    2.3 Take advantage of parallelism with job sharing, for example to
    allocate 8 cores for compilation:

    ```
    cabal build -j8 --disable-optimizations
    ```

    2.4 Parallelism and semaphore, for example to allocate 8 cores in
    total, with better sharing of resources with GHC
    <https://well-typed.com/blog/2023/08/reducing-haskell-parallel-build-times/>

    ```
    cabal build -j8 --semaphore --disable-optimizations
    ```
