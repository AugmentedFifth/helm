---
layout: guide
title: Introduction - Helm, a functionally reactive game engine
section: Introduction
permalink: /guide/
---

## Introduction

Helm is a functionally reactive game engine written in Haskell and built around
the [Elerea FRP framework](https://github.com/cobbpg/elerea). Helm is heavily
inspired by the [Elm programming language](http://elm-lang.org) (especially the
API). All rendering is done through a vector-graphics based API. At its core,
Helm is built on SDL and the Cairo vector graphics library, but is designed
with the ability to interface with other graphical engine back-ends.

In Helm, every piece of input that can be gathered from a user (or the
operating system) is hidden behind a **signal**. For those unfamiliar with FRP,
signals are essentially a value that changes over time. These signals can then
be combined with other signals, producing new signals that map the old data
into new data. This sort of architecture, when used for a game, allows for
pretty simple, clean (and in my opinion, artistic) code.

This guide is still heavily a work-in-progress. The aim of the guide is to give
you tips and tutorials that are not easily gathered from the documentation,
which can be daunting at first.

### Background Reading

* [The Elm Architecture](https://guide.elm-lang.org/architecture/)
* [Eventless Reactivity from Scratch](http://sgate.emt.bme.hu/documents/patai/publications/PataiIFL2009Draft.pdf)
* [Arrowized FRP](http://haskell.cs.yale.edu/wp-content/uploads/2011/02/workshop-02.pdf)
