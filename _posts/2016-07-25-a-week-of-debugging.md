---
layout: post
title: "A week of debugging"
description: ""
category: [GSoC2016, DUNE] 
tags: [Parallel interface, Parallel finite volume scheme, Parallel grid, gdb, MPI, Segmentation faults, ALUGrid]
---
{% include JB/setup %}

As I expected (see the conclusion of my previous post), last week was mostly a week of debugging. Here the things I have done:

* I firstly setup my parallel environment. It wasn't that easy and I had to take one entire day to achieve this. Indeed, I had to generate a parallel grid that could be used with my parallel finite volume implementation since the example of dune-grid-howto constructs a `YaspGrid` using the following code:

```c++
UnitCube ()
{
    Dune::FieldVector<double,dim> length(1.0);
    std::array<int,dim> elements;
    std::fill(elements.begin(), elements.end(), size);

    grid_ = std::unique_ptr<Dune::YaspGrid<dim> >(new Dune::YaspGrid<dim>(length,elements))
}
```

which is not (obviously) useable in dune-corepy. But now that I'm writing down this remark, I'm wondering if using a simple DGF file like

```text
DGF
Interval
0 0   % first corner 
1 1   % second corner
64 64 % 64 cells in x and 64 in y direction
# 

BOUNDARYDOMAIN
default 1    % all boundaries have id 1
#BOUNDARYDOMAIN
# unitcube.dgf 
```

would generate a parallel `YaspGrid` grid (the "trick" being using a "high" number of cells in x and y directions (64 here) to force a load-balancing of the grid). I'm going to test that and give you some news about the result. Anyway, last week I didn't have that idea and thus, I wanted to use an other type of grid. After managing to compile and install the `UGGrid` library, I tried to use it in my volume scheme but I had a lot of errors at compilation. It appears that dune-corepy was
trying to export some types unavailable in `UGGrid` but I didn't take the time to understand in details the errors and try to fix them.
Instead, I compiled and installed the `ALUGrid` library (the `dune-alugrid` module, to be precise) and managed to link it to dune-corepy and use it in my parallel finite volume scheme. Although, I hadn't a (real) parallel grid at the first try. In fact, I was using the following DGF file (representing a unit cube):

```text
DGF
Interval
0 0   % first corner 
1 1   % second corner
1 1   % 1 cells in x and 1 in y direction
# 

BOUNDARYDOMAIN
default 1    % all boundaries have id 1
#BOUNDARYDOMAIN
# unitcube.dgf 
```
and applying a global refinement of 6 (using `gridView.hierarchicalGrid.globalRefine(6)`) and then a call to `loadBalance`. I thought it was enough to get a parallel grid but after putting some `std::cout << PRETTY_FUNCTION << std::endl;` in my (C++) DataHandle class (see my previous post), I realized there was no real communication between partitions of entities because… my grid was still a serial grid. It's by re-reading carefully the dune-grid-howto pdf that I found using a
bigger number of cells in x and y directions was the solution to obtain a real parallel grid (like I said it previously). Finally.   


I also lost some hours trying to figure out why the initialization of openmpi was failing (when trying to execute `mpirun -np 2 python demo/parfinitevolume.pyc` to test my code). Thanks to my mentors, invoking `from mpi4py import MPI` as __first__ line of my Python code fixed that.  

* After this setup, I began to test my parallel finite volume implementation with `mpirun -np 2 demo/parfinitevolume.pyc` and it's here the problems appear, especially with segmentation/type faults. In fact, it took me three days of debugging to finally have a working implementation of a C++ DataHandle class for double values only. Here the lessons I learnt:

> always think carefully about the `pybind11::return_value_policy` to use when writing C++ code that interacts directly with Python (eg `DataHandle`)

>  do not forget about the boolean parameter `borrowed` of `pybind11::object`'s constructor (I had a lot of segfaults because I forgot about it)

With these two lessons, I finally managed to implement a working (C++) `DataHandle` class, using a `pybind11::list` as buffer for the Python `DataHandle` (see the code [here](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/master/dune/corepy/grid/gridview.hh#L104))

* Finally, I tried to be generic by providing a C++ `DataHandle` parameterized by `pybind11::object` (and no more `double`), to give some freedom to the Python user. And again, that was the source of a lot of segmentation faults. Since I haven't yet succeed to do that, here is the email I sent to my mentors:

> I tried to be generic and replaced `double` by `pybind11::object`. As you,
Martin, pointed out: "A Python type is an object itself, so we can pass
this on data handle construction.". That's what I was thinking too, but
unfortunately, that's not so simple.
>
> When I did that and adapted in consequences `ProxyDataHandle::gather` and
`ProxyDataHandle` (see  <http://paste.awesom.eu/7Ogk> ), I got segfaults again.
But that time, I managed to use gdb to trace my two (openmpi) processes and
I found some interesting issues:
>
> 1) The first segfault was caused by a call to `static void copy ( void
*dest, const T *src, std::size_t n)` line 444 of
`dune-alugrid/dune/alugrid/impl/serial/serialize.h`, called by `inline void
writeT (const T & a, const bool checkLength )` line 133. Why? Because in
that function `copy`, there is this line `static_cast< T * >( dest )[ i ] = src[ i ]`. When `T` is `pybind11::object`, that call to `operator=` induces a
call to `pybind11::handle::dec_ref()` that segfaults. And why? Because "dest"
is actually a buffer that was allocated with `malloc` but never initialized
with a call to `new` (see <http://paste.awesom.eu/Piig> in serialize.h) and
thus `static_cast< T* >( dest )[ i ]` refers to an uninitialized part of
memory instead of a `pybind11::object`.
>
> 2) Same thing with `inline void readT (T& a, bool checkLength )` line 156
that caused a segfault in a call to `pybind11::handle::inc_ref()` for the
same reasons.
>
> Good news is: I fixed these two segfaults by modifying these writeT and
readT functions to call operator `new` on the allocated buffers, see
> <http://paste.awesom.eu/Eg7T> . After recompiling dune-alugrid and
dune-corepy, these segfaults disappeared.
>
> Bad news is:
>
> there is an other exception that I didn't manage to fix. That time it's a
"Unable to cast Python object to C++ type.", see <http://paste.awesom.eu/nBf=5> gdb showed that it's this line 869 `std::vector< ObjectStream > out( 1 );`
in the function `void receive( DataHandleIF& dataHandle )` of
dune-alugrid/dune/alugrid/impl/parallel/mpAccess_MPI_inline.h that causes
the exception. Problem is: it's during the initialization of a std::vector
and that is libstdc++. In Archlinux, we don't have debug compiled version
of the libstdc++ so gdb couldn't step into the libstdc++ (at least, not
entirely, see my gdb trace: <http://paste.awesom.eu/XEwB> ) But I think the
problem is about ObjectStream and the class BasicObjectStream that probably
does something wrong that induces a pybind11 cast that shouldn't happen
(since pybind11::object is already a C++ type...) But I don't know how to
confirm that hypothesis (at least, without recompiling the libstdc++ in
debug mode to step with gdb).

I don't know if I will continue to debug that issue. In fact, my mentors began to clean `dune-corepy` and `dune-fempy` in order to use `dune-corepy` in `dune-fempy` and there is a lot to discuss and work on. We will skype on this tomorrow morning and I think this conversation will set my next work to do.

I keep you in touch.
