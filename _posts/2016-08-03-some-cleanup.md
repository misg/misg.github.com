---
layout: post
title: "Some cleanup"
description: ""
category: [GSoC2016, DUNE]
tags: [cleanup, dune-fempy, dune-corepy, documentation, debugging]
---
{% include JB/setup %}

Last week was dedicated to some cleaning of `dune-corepy` and `dune-fempy`. In fact my mentors, who are working on `dune-fempy` for a long now, wanted to base `dune-fempy` on `dune-corepy`. Since I worked a lot on `dune-corepy`, it was the time to take some actions in that direction. In consequences, my mentors cleaned up the codebase of `dune-corepy` and moved some stuff from `dune-fempy` to `dune-corepy` (especially some features for grid constructions). I finally reviewed and tested their changes and they merged that into the master branche (see the commits in the gitlab).

I also continued to debug my parallel interface when using a `DataHandle` class specialized with `pybind11::object` (see my previous post) but still without success. Thus, I chose to move on, writing some documentation for the developers who may want to contribute to `dune-corepy`. In fact, after having a skype conversation with my mentors, we decided to prepare for the end of this GSoC: documentation, cleaning, tests and bug fixes. Thus, I made some bug fixes after the merge of my mentors
(the iterators were broken) and wrote some documentation with sphinx.

Next week will also be dedicated to documentation, cleaning and bug fixes, I keep you in touch!
