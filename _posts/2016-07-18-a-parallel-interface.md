---
layout: post
title: "A parallel interface"
description: ""
category: [GSoC2016, DUNE]
tags: [Parallel interface, Parallel finite volume scheme, PyObject protocol]
---
{% include JB/setup %}

As I said it in the last post, my current milestone is to provide bindings for the parallelization interface in order to run the parallel finite volume scheme (described in the `dune-grid-howto`). Thus, I worked on that last week:

* I firstly exported some enums types relative to the parallel grid partitions (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/51823e58e6c9b04433b9adfd527d615ad84d92e5)), which are needed to iterate through entities in the parallel case (see the section "Common entity ranges for non-standard parallel partitions" [here](https://www.dune-project.org/doc/doxygen/html/group__GIIteration.html)).
* Then I exported the common entity iterators to iterate through entities of any codimension and any partition (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/eb01d14cd689665f3b3712c6251713686d27452e) and [[2]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/78aa77fa9b4011c91270cf3e75731542b5da8e66)). Like the iterators on entities of any codimension, the general idea is to use a `static` array of lambda functions where each lambda manages the creation of an iterator on entities of a specific codimension and a specific type partition (it's a 2d static array generated at compile-time with the use of variadic templates, see the code in `gridView.hh`).

* I also implemented the communication interface (see [[1]](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/7c6acad61ca688cb2697106e2ee8e94b326f7fe0)) that merely consists in a `communicate` method and a `DataHandle` class used to let the user manages the communication between (parallel) partitions of entities. That was tricky since this `DataHandle` class has to derives from `Dune::CommDataHandleIF` (see [here](https://www.dune-project.org/doc/doxygen/html/classDune_1_1CommDataHandleIF.html)), uses the CRTP and has to be implemented in the Python side. My idea was to define a `proxy` class `DataHandle` that respects the CRTP but forwards all the function calls to the (real) Python implementation of `DataHandle`, using a `PyObject*` (pointing to that Python class). Unfortunatly, I haven't tested yet that communication interface because I have to setup my parallel environment (especially running correctly OpenMPI and creating a DUNE grid that handles parallelization).

* I finally wrote the code of the parallel finite volume scheme (see `dune-grid-howto` and my code [here](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/commit/3702891cf54f9fcb27f4216382c0bb54bcd7f001)) but as for the communication interface, I haven't tested it in "real conditions" (eg in a parallel environment).

In brief, I will be working this week on testing (and surely… debugging) my parallel interface with a parallel environment, I keep you in touch!
