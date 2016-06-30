---
layout: post
title: "Midterm evaluation"
description: ""
category: [GSoC2016, DUNE] 
tags: [Midterm, Finite volume scheme, Finite element scheme, C++, Python, pybind11]
---
{% include JB/setup %}

As the midterm evaluation has passed, it's now the time to write about the things I have done during these weeks. In order to give a complete overview, this post will be more like a list-of-things than a developed write-up.

__Week of June 6:__

* simplify the finite volume scheme's Python code, using lambda functions, thanks to my mentor Andreas Dedner (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/267ff3a6f2ded1e28d4bf2c37bf77e6079ad0398), [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/fef9c45534f78d264f394514339616d6cfd7dfd0) and [[3]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/5efad1bb0aef0a83bc1f655c5d3d4d8d61456886))
* complete and write other bindings for the following types: `Entity`, `Intersection`, `GridView` and `IndexSet` (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/a860c145f9df65336f4df2f4c65c8a8321a36e6c), [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/9f6d36f966d7fb63c2f5e6817882359ce4187485), [[3]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/24b73ddfdec339de137a22ee4361e08439f5d316),
  [[4]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/3835345c180c78e53ac013d17ce59908a0a21fa3), [[5]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/9a3b1ee75288dd0dd62fdece5530a7418b285b46) and [[6]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/9df5aaf3c47ab8222aa06ea8e014ccf38444fc09))
* clean some code (indentation, CamelCase convention (used in the C++ codebase of DUNE, I chose to follow it for the Python bindings even though the `PEP8` recommends using underscore), refactoring)
* talk with my mentors about my code (reviews, improvements, etc.) (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/51f5efcdba09d8d1e7b3b42255ad412585243da1) and [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/97005be23c414a1195a2a956b5b2a80947a9f463), [[3]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/d037a31b0d110ee1d55db6d8672201d65e8994c4),
  [[4]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/49fd3050dcfb5e34dc24857cf092b7463edd5267) for the commits of my mentor Martin Nolte, thanks to him too)

__Week of June 13:__

* improve the finite volume scheme's Python code: remove a non-consistent import (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/dc9486153e12e580e80f0e720bc6edd0a8aa9f47)), add the `dimension`, `dimensionworld`, and `conforming` constants to gridViews (see [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/bca2e8b8d69e388406685d61b3d30d9eed6a3300)), manage dot products between Python `list` and `FieldVector` because of Python reverse operators
  (see [[3]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/ad9d618bba0d5e464eec9c3ac01fa82f7b5292e1)), add `None` return type to some functions (see [[4]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/2fa3e2b0dbcf0c3d030b38e703db8269c5160a1d) and [[5]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/1db09ff715308f58885a9d95266bc0278ec28c1a))
* simplify the export of Python iterators (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/9572b6292616f1e16255073322c92c0c38b86562), [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/4a03be85433bd62e7106e1f86c718d2e1523ddb8), [[3]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/c505f79c40ce4fa1f0cc353dd4539840c6055018) and
  [[4]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/57be9088054bebe0bc414400b1a4f910b064dda3))
* manage the export of Python iterators to iterate over entities of any codimension (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/4c9d917401a050038f6169f39fcc0392e20aab86) and [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/8bf7285ad128e60119bdc9fd19f93ccf27558903)). That was a challenge (it took me a day of work) since the codimension is a (int) template parameter fixed at compile time… My idea was to generate a compile-time array
  (thus, a static array) that contains `N=dimgrid+1` lambda functions, where each lambda manages the creation of iterators for a set of entities of the same codimension (`0` to `dim`), read the code for more details.
* read the finite element scheme example of dune-grid-howto (part “A FEM example: the Poisson equation”) and think about how implement it in Python
* complete bindings for `FieldVector` (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/b3c0819c6bf5f770869197c7b4002e2f6e31ddfa)) and add bindings to export `FieldMatrix`, `DynamicVector` and `DynamicMatrix` (see [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/0c664a112e35ff7341406c53a36f2e7280731734), [[3]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/7bf7a998b5863bcc37e7730d3fba9708251f83c1) and
  [[4]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/c60a86fcccb7c5617d598597f28d6ba51e784884)), used in the finite element example

__Week of June 20:__

* manage linking to AlbertaGrid (used for the finite element example), it took me some hours to find and understand the CMake files related to that and modify the right things… (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/65704591771ba26278270b78e7f21f70751048d6))
* write bindings to export `ReferenceElement`, `ReferenceElements`, `QuadraturePoint`, `QuadratureRule`, `QuadratureRules` and `MultiLinearGeometry` (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/a96e44b1a9bdeb9d338e4c76c5d77e361b21dda3), [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/f8733f6d3d760d266bbbba38186ff0c34b07fefc), [[3]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/2f0e11a67023df1ccf6ac8c2e95d539d9b6e8482) and [[4]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/58a729f6bc37eba6cbffe73cb156584437a221ec))
* implement Python iterators for the previous types ; I spent twelve hours on that since I had segfaults. It turned out that I was exporting several types (with variadic templates) with the __same__ pybind11 name without any compile-time error but a nice segfault at runtime (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/2f0e11a67023df1ccf6ac8c2e95d539d9b6e8482))
* play with `Numpy` to use `ndarray` in the Python implementation of the finite element scheme, I especially exported the buffer protocol of `FieldMatrix` (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/4a44d72610e53993b95f60bf5b2b250ddfda63a3)) to avoid expensive copies (for example when computing the Jacobian Inverse Transposed, see `Line 69` of
  [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/8bbeae1ec03fcc8b23cc8e25505f56cea875caf4))
* play with `Scipy` to solve the problem, using the `scipy.sparse.linalg.bicgstab` solver (Biconjugate Gradient Stabilized iteration solver), see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/b1176d4c2b27ba61ea45d6fbc6016cffa94bf984)
* manage the VTK output of the solution and that wasn't easy since I hadn't compile time error but my VTU files were empty… (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/b632d5437efe56e34b36ca166473bf0b52966530))

But finally, after some hours of debugging (see the bugfixes of June 24 and 27), I managed to have a working implementation of the finite element scheme, see the results [here]({{ BASE_PATH }} /assets/img/fem2d.png) for a 2d solution and [there]({{ BASE_PATH }} /assets/img/fem3d.png) for a projection of the 3d solution. :-)

On an other subject, I just learned that I passed midterm evaluation! So the adventure is still ongoing, I keep you in touch!
