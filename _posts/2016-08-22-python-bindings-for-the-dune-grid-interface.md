---
layout: post
title: "Python bindings for the DUNE grid interface"
description: ""
category: [GSoC, DUNE]
tags: [The end, Work product submission, Python bindings, Grid interface]
---
{% include JB/setup %}

Since we are at the end of the Google Summer of Code 2016, It is now the time to write a final post to talk about what I have done during this summer.

But let me re-introduce myself first: I am Michaël Sghaïer, french third-year student in Software Engineering at Polytechnique Montréal, Canada. I am mostly interested in C++ programming, Python and algorithms and this is why I have taken part in this Google Summer of Code 2016 as developer for the [DUNE organization](https://www.dune-project.org/) where [my project](https://summerofcode.withgoogle.com/projects/#6395085230964736) was to implement Python bindings for the DUNE grid interface.

### What I have done

As described in [my proposal](https://summerofcode.withgoogle.com/serve/5660370790252544/) and in the [first post](http://misg.github.io/gsoc2016/dune/2016/05/01/a-gsoc-adventure) of this blog, my project was to export to Python the grid interface that DUNE provides. This [grid interface](https://www.dune-project.org/doxygen/master/group__GridInterface.html) is the core of the DUNE project since each grid implementation has to fulfill it to provide a specific discretization of space, in a transparent way for the user who can then solve partial differential equations using the grid of his choice.

To achieve this project, I dived myself in the huge C++11 codebase of DUNE, learnt how to use pybind11, discussed a lot with my mentors, worked with sub-milestones and finally did a lot of debugging. You can find the final result as a DUNE module named `dune-corepy` just here: **<https://gitlab.dune-project.org/michael.sghaier/dune-corepy>** and see all my activity (commits, issues, merge requests) here: **<https://gitlab.dune-project.org/u/michael.sghaier>** (entirely related to this
project). The project is in a useable state and you can test it after a few steps, as explained in the
[README](https://gitlab.dune-project.org/michael.sghaier/dune-corepy).

With the help of my mentors, I implemented bindings for almost all of the concepts listed in my proposal:

- (hierarchical) grid
- gridView
- entity
- geometry
- iterator
- intersectionIterator
- indexSet
- grid construction
- parallel computing/communication
- load balancing

As you can see, I did not export any mechanism to use adaptivity (which is the name of the process of refining a grid) in Python. Indeed, my mentors would like to provide a more advanced mechanism that the one provided by default by the grid interface, so they asked me to not care about that. 

Regarding the unit tests and the build system I talked about in my proposal, there have been some changes. Indeed, I did not provide unit tests but instead I implemented the finite volume, finite element and parallel finite volume schemes described in the [grid-howto.pdf document](https://www.dune-project.org/doc/tutorials/grid-howto.pdf) in Python, using my bindings. It gave me sub-milestones to achieve and that was crucial to know where I was going, in addition to helping me debugging my bindings. As for the build system, I simply used DUNE's so I
did not develop anything new.

Finally, I maintained this blog to give regular feedbacks about my project (see the archive [here](http://misg.github.io/archive.html)).

### What is left to do

As the end of this GSoC was coming, I did some cleaning and gave a thought about what is left to do (or at least, what is left to think about):

- providing a mechanism to use adaptivity from Python
- fixing the [bugs listed in the GitLab](https://gitlab.dune-project.org/michael.sghaier/dune-corepy/issues)
- maybe developing a system for testing grid prototypes

### A final word

It have been a real pleasure taking part in this adventure! I learnt a lot: about C++ (11 and 14), about the possible interactions between C++ and Python, about how to manage an open source project and finally about myself. Indeed, after this GSoC, I think I am more interested in computer science itself rather than a scientific field that uses computing capabilities (like computational science in the case of DUNE, for example). So I am not sure I will continue to contribute to DUNE, although
I will maintain `dune-corepy`.

Anyway, It has been definitely a great experience and I warmly thank Google, DUNE and my mentors for having given me this opportunity! :-)
