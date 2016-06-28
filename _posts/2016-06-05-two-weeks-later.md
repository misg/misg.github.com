---
layout: post
title: "Two weeks later…"
description: ""
category: [GSoC2016, DUNE] 
tags: [C++, Python, Bindings, pybind11, Finite volume scheme]
---
{% include JB/setup %}

In the last post, I talked about the end of the community bonding period and with it, the beginning of the coding period. Two weeks have passed and it's now the time to write a little feedback.

### A first milestone

I didn't talk about it in my previous post but during the community bonding period, I defined a first milestone with my mentors. It's interesting because it's not a milestone I had in mind. Indeed, in [my proposal](http://37.187.19.181/Proposal.pdf), I defined “providing bindings for the most important concepts of the grid interface (Grid, GridView, Entity, Geometry, Iterator, IntersectionIterator, IndexSet, grid construction/output” as my first milestone of this GSoC project. Since it's
quite exhaustive, I had in mind to pass through all the DUNE codebase, seeing what code was relevant to a previously quoted concept and providing bindings for it. But my mentors had in mind an other working method that I found more efficient: a method oriented “concrete results”. In fact, as a starting point for my project, they wondered “what are the things we would like to be able to do with the bindings?”. And it's a good question because there is a document that presents some useful
examples of what you can do with DUNE…

That document, it's [The Distributed and Unified Numerics Environment (DUNE) Grid Interface HOWTO](http://www.dune-project.org/doc/grid-howto/grid-howto.pdf) that gives an introduction to DUNE and especially explains the concepts related to the grid interface and presents examples of what you can do with the grid interface (constructing grid objects, applying quadrature rules, attaching user data to a grid, etc.). It's one of these examples that my mentors suggested to me as a first milestone: `Cell centered finite volumes` (page 43) (also known as a "finite volume scheme"). Basically, this example implements a numerical solution, with DUNE, of a partial differential equation.

I took some time to study this part of the grid-howto document, reading the code, trying to have an overview of this example rather than spending my time trying to understand the details of the underlying Mathematics. This overview I acquired is very close to the structure of the code and be summarized like that:

- `finitevolume.cc`: the global idea is to describe over time the evolution of a concentration in an initial environment (which has been discretized with a grid)  
- `transportproblem2.hh`: definition of the problem with the initial and boundary conditions and a velocity field
- `initialize.hh`: initialization of the concentration relative to each cell of the grid
- `evolve.hh`: time evolution of the concentration, computed with the equations that have been found
- `vtkout.hh`: output function to visualize the solution with ParaView

By studying the code, I have been able to extract the DUNE concepts that I would have to export with pybind11 in order to implement this finite volume scheme in Python. What is it worth to note is that these DUNE concepts are the same as those described in my original proposal: grid, gridView, entity, geometry, intersectionIterator, grid construction (see the grid-howto for more details), etc.

Thus, implementing this `finite volume scheme` example in Python appeared to me as a good milestone to progress in my project and give some concrete results. On top of that, my mentors gave me an initial codebase at the beginning of the coding period, as a good starting point to develop my project and achieve my first milestone. Let me introduce it. 

### An initial codebase

The first challenge to implement these Python bindings was to manage the compile time aspects of the DUNE codebase to provide bindings useable in a runtime environment. Especially since the DUNE concepts are all related to each other and use a lot of C++ compile time technics. The principal example I can give is the grid abstract base class that defines the common interface of a grid with the CRTP (mentionned in my previous post):

```c++
template<int dim, int dimworld, class ct, class GridFamily>
class Grid
{
    typedef typename GridFamily::Traits::Grid GridImp;
    ...
```

all this interface forwards its methods to the real grid implementation (the concrete type behind `GridFamily::Traits::Grid`) defined by the user. Same thing for the GridView class that depends on the grid implementation:

```c++
template<class ViewTraits>
class GridView
{
    typedef typename ViewTraits::GridViewImp Implementation;
    ...
}
```

and similarly, a lot of other classes are templated by a GridView, for example like the `MultipleCodimMultipleGeomTypeMapper` (which is an implementation of the mapper interface/concept to attach data to grid cells):

```c++
template<typename GridView, template<int> class Layout>
class MultipleCodimMultipleGeomTypeMapper
{
    ...
```

all of these types are determined at compile time by the C++ compiler but since the idea behind the bindings is to generate a shared library useable at runtime in Python, these types cannot be determined at runtime. This was a first challenge to solve!

Fortunately for me (I think I would have come up with a bruteforce solution like exporting the bindings for all the existing grids with a predefined range of dimensions, which is not a good solution for obvious reasons), my mentors gave me that initial codebase, which solve this problem with the idea of generating the shared library - relative to all the types that depend on a grid implementation - on the fly, with Python system calls. You can see the tree of this codebase righ here:

```
.
├── CMakeLists.txt
├── data
│   ├── circle.dgf
│   ├── ...
│   ├── unitcube-2d.dgf
├── demo
│   ├── CMakeLists.txt
│   └── grid-demo.py
├── dune
│   ├── CMakeLists.txt
│   └── fempy
│       ├── CMakeLists.txt
│       ├── function
│       │   ├── CMakeLists.txt
│       │   ├── gridfunctionview.hh
│       │   └── simplegridfunction.
│       ├── py
│       │   ├── CMakeLists.txt
│       │   ├── grid
│       │   │   ├── CMakeLists.txt
│       │   │   ├── entity.hh
│       │   │   ├── function.hh
│       │   │   ├── geometry.hh
│       │   │   ├── gridview.hh
│       │   │   ├── hierarchical.hh
│       │   │   ├── range.hh
│       │   │   └── vtk.hh
│       │   └── grid.hh
│       └── pybind11
│           └── ...
├── python
│   ├── CMakeLists.txt
│   ├── database
│   │   ├── CMakeLists.txt
│   │   └── grid
│   │       ├── CMakeLists.txt
│   │       ├── dune-alugrid.db
│   │       ├── dune-grid.db
│   │       └── dune-spgrid.db
│   └── dune
│       ├── CMakeLists.txt
│       ├── common.cc
│       ├── function.py
│       ├── generated
│       │   ├── CMakeLists.txt
│       │   ├── generated_module.cc
│       │   └── __init__.py
│       ├── generator
│       │   ├── CMakeLists.txt
│       │   ├── database.py
│       │   ├── generator.py
│       │   └── __init__.py
│       ├── grid.py
│       ├── __init__.py
│       └── mpihelper.cc
```

Basically, all the bindings implemented with `pybind11` and that depend on a grid implementation are in `dune/fempy/py/grid/` (using the `pybind11` headers located in `dune/fempy/py/pybind11`) and are exported in the `grid.hh` file like that:

```c++
template<class GridView>
void registerGrid(pybind11::module module)
{
    registerHierarchicalGrid<typename GridView::Grid>(module);
    registerGridView<GridView>(module, "LeafGrid");

    ...
}
```

In parallel of this C++ code, you have some Python code in `python/` that manage the generation on the fly of the shared libray (relative to the grid interface) I just talked about.

The idea is to use a database with one entry for each type of grid that describes the concrete type (eg `Dune::AlbertGrid< $(dimgrid) >`) and provide the needed headers (eg `"dune/grid/albertagrid.hh, etc.`). You can look at the `python/database/grid/dune-grid.db` file for a more detailed example.

Then the Python module `generator` retrieves the relevant entry of the database, depending on the parameters providing by the user (eg `"AlbertaGrid"`, `dimgrid=2`) and generates a header `generated_module.hh` that defines a `PYBIND11_PLUGIN` and calls `registerGrid<GridView>` I mentionned above. This header is finally included in a `generated_module.cc`, compiled into a shared library `.so` with the help of CMAKE:

```cmake
add_library(generated_module SHARED EXCLUDE_FROM_ALL generated_module.cc)
target_include_directories(generated_module PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
set_target_properties(generated_module PROPERTIES PREFIX "")
```

It took me some time to understand all of that and start playing with some code. But I finally managed to achieve my first milestone…

### Progress and first achievement

As I said it just before, my first milestone was to implement the finite volume scheme in Python. It took me these two weeks to achieve that and you can find the code [right there](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/demo/finitevolume.py).

The code structure is pretty much the same you can find in dune-grid-howto's example. Here is a list of things I did to manage that:

- writing bindings for the `MultipleCodimMultipleGeomTypeMapper` class that makes it possible to attach data to elements/vertices in a grid (see [mapper.hh](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/dune/corepy/grid/mapper.hh))
- managing definitions of `MultipleCodimMultipleGeomTypeMapper` layouts on the Python side (see [Line 64](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/dune/corepy/grid/mapper.hh#L64) of `mapper.hh` and [Line 95](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/dune/corepy/grid/mapper.hh#L95))
- exporting `Intersection`, `IntersectionIterator` classes to work with element intersections (with the domain boundaries or other elements) (see [intersection.hh](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/dune/corepy/grid/intersection.hh) and [range.hh](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/dune/corepy/grid/range.hh))
- exporting `GeometryType` (see [type.hh](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/dune/corepy/geometry/type.hh))
- managing VTK output (see [vtkout function](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/024f308cebf038ddc4485cd8b26a7f45acad272d/demo/finitevolume.py#L28))

I also spent hours and hours in debugging type errors, quite a challenge since these errors are at runtime and managed by Python even though the problem is in the C++ code… Anyway, you can now run the finite volume scheme with Python and obtain vtu files to see the nice animation of the solution in ParaView!

For now, I'm going to talk with my mentors about what I can improve in the code and work on until midterm. I keep you in touch!
