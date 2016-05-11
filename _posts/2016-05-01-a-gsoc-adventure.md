---
layout: post
title: "A GSoC adventure…"
description: ""
category: [GSoC2016, DUNE] 
tags: [Opening]
---
{% include JB/setup %}

In this first post, I present myself, the reasons of this blog, what you should read in the following weeks and finally some first technical leads about my project.

### About me

My name is Michaël Sghaïer, I'm a second year student in Software Engineering at Polytechnique Montréal and I have the chance to have been selected for the Google Summer of Code 2016! I'm going to work (full time) from now to August, on DUNE - Distributed and Unified Numerics Environment - in order to provide Python bindings for the core interface (which is actually written in C++). I will go back into the technical details later in this post.

Why maintaining a blog? Well, I'm not going to lie: my mentors asked me to. But after some thoughts about it, I see some advantages in it, that I like to share with you. First, I think it is a good way for my mentors to stay up to date with my work and my progresses. It is a good asynchronous support, more centralized than IRC or a mailing list, which is I think, more convenient to share thoughts about a project like that. For me, it's a good way to take a step back about my
work, with all the advantages it implies. And for you, reader, well it depends on your status! If you're a student who never participates in a GSoC, I hope this blog will be a source of motivation for you to attempt the adventure next years! If you're not, I simply hope that you will learn some things through my posts or share your ideas with me.

I should write every week a post on my progresses, so basically you will find writings about my project and all the things related to, in a bulk: Python, C++, boost::python, unit-tests, building system, bindings, DUNE, personnal organization, etc.

Let us now go more into technical details about this GSoC project!

### The project: Python bindings for the grid interface

Like I said, I'm going to work on DUNE - Distributed and Unified Numerics Environment -, which is “a modular toolbox for solving partial differential equations with grid-based methods” (quoted from <http://dune-project.org>). Here, the concept of “grid” is essential because solving differential equations in a continuous space is (very) difficult and thus, you have to basically discretize your space to solve your equations. What does that mean? “Simply” draw a grid in your space (for example, a “real” grid for a 2D space or a
mesh for a 3D space). This is a simplistic vision of the problem but I think It's enough to, at least, get a rough idea about DUNE.

Now, this concept of grid is very rich, in reality. In fact, you may want a grid in a space of any dimension, with a specific geometry for your cells. You may want to be able to refine (in a recursive way) your grid, add some local information in each cell, be able to traverse it, apply algorithms on it or even parallelize it, etc. Consequently, there are many possible implementations for the concept of “grid” and DUNE aims to provide a generic interface to build any grid over it.

DUNE is written in C++, mostly to ensure efficiency in scientific computations and to support high-performance computing applications. So it uses a lot of static polymorphism, especially in its grid interface, through C++ templates and function inlining. And thus, using DUNE can be quite difficult for everyone: beginners have to acquire detailed C++ knowledge and even advanced users are limited in their productivity due to some subtleties.

It is in this context that my GSoC project takes place: writing Python bindings for the grid interface. In fact, it aims to make DUNE more useable because it would give the possibility to use the grid interface through Python. Prototyping and implementation of grids shoud thus be more easier for developers.

To say a few words about the programming details, I will write these bindings with the boost::python library, from the [Boost project](http://www.boost.org/). So far, I used boost::python to write some minimalist bindings for the FieldVector and FieldMatrix classes that you can find on the GitLab : <https://gitlab.dune-project.org/michael.sghaier/dune-common/tree/master/src>. I will talk more deeply about the code and the idea of writing bindings in a future post.

### A final word

This opening post reaches its end, I hope it gave you a good overview of my project! As for me, I just finished my final exams and I'm going to use this week to organize myself for the GSoC (especially returning in France to work with no time difference with my mentors and drawing up a planning for the following weeks). Stay tuned! 
