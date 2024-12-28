---
layout: post
title: "OCaml Quickstart"
description: "A quickstart guide for you to get started with OCaml."
---

## Project setup

* `opam` is the package manager.
* `dune` to OCaml is like sbt to Scala.
* `utop` is the REPL.
* `merlin` provides IDE features for your editor.
* `ocp-indent` helps you indent OCaml code.
* `ocamlformat` helps you format OCaml code.

```shell
# setup opam
opam init
# install OCaml
opam switch create 4.10.0
# update environment
eval $(opam env)
# install packages
opam install dune utop merlin ocp-indent ocamlformat
```

If you just need to meddle with a single file, create a `dune` file along with it and you're good to go:

```lisp
(executable
  (name      main)
  (libraries base other_lib))
```

`main` is nothing but the name correspond to the file `main.ml`, or whatever you file name is. When you run `dune build main.exe` it'll generate an executable inside `./_build/default/main.exe` which you can run with. Or you can use `dune exec ./main.exe` which combines building and running an executable into a single operation. Use `opam` to install whatever libraries you need and add them to `libraries` stanza.

Now to work with multiple files, read the dune [document](https://dune.readthedocs.io/en/stable/quick-start.html) first. This is an example:

```
.:
bin  dune-project  my-proj.opam  src

./bin:
dune  main.ml

./src:
ast.ml  dune  lexer.mll  other.ml  parser.mly
```

```lisp
; file: dune-project
(lang dune 2.6)
(name my-proj)
; you can ignore this.
(using menhir 2.1)
```

```lisp
; file: ./bin/dune
(executable
 (name main)
 (libraries lib))
```

```lisp
; file: ./src/dune
(library
 (name lib))
; you can ignor this.
(ocamllex lexer)
(menhir (modules parser))
```

Note that if you want to run `dune utop` and automatically load modules, it has to be a library. So I setup main.ml as an entry point and keep my source files is `src`, aka a library.

---

## Emacs setup

You can check out my [setup](https://github.com/0x7ffc/lain-emacs/blob/master/lisp/lain-ocaml.el).

* use `C-c C-s` to switch between source file and REPL.
* `lain/ocaml-utop` will ask which directory you want to run the REPL in, default is the project root.
* format the project with `, d F` and many other commands in `, d`.
* eval current expression with `, e e` and many other commands in `, e`.
* goto definition with `M-.` and go back with `M-,`.

---

## OCaml

OCaml doesn't have a function composition operator like the Haskell `.`. [Here](https://discuss.ocaml.org/t/why-dont-we-have-a-composition-operator-in-pervasives/1210/18) is a discussion, TLDR: OCaml doesn't favor point-free style.

OCaml doesn't have ad hoc polymorphism, it's the reason you have `print_string` and `print_int` instead of a single `print`. You may know this as overloading in other languages. [Modular implicits](http://www.lpw25.net/papers/ml2014.pdf) is OCaml's way to support it, but it seems not under active development. [Here](https://discuss.ocaml.org/t/modular-implicits/144/61) is a discussion, TLDR: Multicore is more important.

Haskell's functor and OCaml's functor are not the same thing, although they both got the name from `Category Theory`. OCaml's functor works on module. In short it's a parametrized module which is a module that takes another module(s) as argument(s) and outputs a module.

You may already know that modules took an important role in OCaml. They're not like other languages where they exists just to separate namespaces or import namespaces. OCaml's module system provides abstraction and composition. It also makes compilation very fast because it ensure that each module can be typechecked and compiled incrementally.

Java's Generic and Interfaces provides something similar to OCaml module system, here is an example of OCaml generic Set.

```ocaml
module type OrderedType = sig
  type t
  val compare : t -> t -> int
end

module IntType = struct
  type t = int
  let compare x y = x - y
end

module IntSet = Set.Make(IntType)

(* And since OCaml's Int module already have these, you can just use it *)
module IntSet' = Set.Make(Int)
```

However Java doesn't support higher kinded polymorphism, simply put, you can only abstract over type(Set<T>) but not type constructor(T<Integer>). For example, you want to abstract over the notion of a tree which is something with nodes and leafs and you don't care what the nodes actually are, here is how you do it:

```ocaml
module Tree (F : sig type 'a t end) = struct
  type 'a t = Leaf of 'a | Node of ('a t) F.t
end

(* And here we can create Rose Tree or Binary Tree *)
module RoseTree = Tree (struct type 'a t = 'a list end)
module BinTree = Tree (struct type 'a t = 'a * 'a end)
```

And if you want to get your hands dirty with front end, there is [ReasonML](https://reasonml.github.io/).

That's about it, here are some helpful resources:

* [CS3110](https://www.cs.cornell.edu/courses/cs3110/2020sp/textbook)
* [For Haskell fellas](http://blog.shaynefletcher.org/2017/05/more-type-classes-in-ocaml.html)
* [OCaml manual](http://caml.inria.fr/pub/docs/manual-ocaml)
* [Real World OCaml](http://dev.realworldocaml.org)