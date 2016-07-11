---
layout: post
title: "Post midterm period"
description: ""
category: [GSoC2016, DUNE]  
tags: [Documentation, PDESoft, Finite element scheme]
---
{% include JB/setup %}

Here is a short post to say what I did during the two past weeks.

__Week of June 27:__

* I spent three days trying to improve the performances of my finite element scheme implementation (see my previous post and [the code](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/blob/master/demo/finiteelements.py)). Indeed, after some simple benchmarks with the Python module `time`, I realized that if the solution was computed in a few seconds for a refinement of 10, it took too much time for a refinement of 13 (more than one minute). At a first glance, I thought it was because I used a dense matrix and not a sparse matrix. Thus, I spent two days reading the Numpy
  documentation about sparse matrices, doing tests with several types of matrice (dok\_matrix, lil\_matrix, coo\_matrix, csc\_matrix, csr\_matrix) and benchmarks. Unfortunately, the use of a sparse matrix didn't improve the performances : on the contrary, they made them worse (I'm talking about the running time, not the use of memory). So finally I realized that the problem wasn't about the use of a dense matrix but rather the four nested for loops inside the `assemble` method. I talked about
  that with my mentors and they said to me it was not such a big deal since I should focus on providing DUNE's Python interface itself rather than performances.

* I spent two days writing the previous posts (the date on an article is not its publishing date but the date of its creation)

__Week of July 03:__

* I was at the [PDESoft 2016](www2.warwick.ac.uk/fac/sci/maths/research/events/2015-16/nonsymposium/pde/), at Warwick University. I spent three days seeing the work of the DUNE community and that was interesting. I also talked IRL with my mentors about my work and what should be done for the next weeks. We agreed that I should work on adaptivity and parallelization and so I have a new milestone: implementing the finite volume scheme with a parallelized grid (see dune-grid-howto and its `parfinitevolume` example).

* I worked on the documentation. Powered by Sphinx, you can build it with a `make doc` in the `build-cmake` directory of dune-corepy. For now, there is the user documentation. I'm working on some documentation for the developers who may want to contribute to dune-corepy.
