---
layout: post
title: "Shall we start to code?"
description: ""
category: [GSoC2016, DUNE] 
tags: [Community bonding period, CRTP, C++, templates, pybind11, "boost::python"]
---
{% include JB/setup %}

In the previous post, I was talking about the community bonding period and the Python bindings that needed to wrap direct calls to boost::python (to ensure maintenability and permit to someday move to pybind11 for example). Now, the community bonding period is officialy finished so the time to write a little feedback about it had coming. I will begin by talking about the choice to finally chose pybind11 over boost::python then I will introduce you the CRTP (“Curiously Recurring Template
Pattern”) and finish by draw up a little planning for the following days.

### From boost::python to pybind11

In the last post, I was talking about writing maintainable Python bindings to ensure the same approach when exporting classes and make it easy to move from boost::python to pybind11 in the future. My idea was to implement a kind of facade pattern, a layer just above boost::python that would make it simple to change to pybind11. As explained in my last post, it was all about metaprogramming. And as I was reading Stroustrup's book, I just discovered [variadic templates](http://en.cppreference.com/w/cpp/language/parameter_pack) that especially allow to work with functions with an unknown number of parameters. I wrote a little code (<http://pastebin.com/gqY6rzah> and <http://pastebin.com/jf9fYnHF>) to experiment this notion of variadic templates and think about a facade pattern for boost::python and pybind11.

The idea was to simply add a layer above boost::python to export C++ classes (in a very simplified way). The principal difficulty was to manage the unknown number of methods that could be exported and this is where variadic templates took place. The solution I found is to use three functions: a first `template<typename T, typename... Fn> void class_(std::string name, Fn... fn);` that defines a `boost::python::class_` with the type and the name of the C++ class the user wants to export and which passes this `boost::python::class_` to a second function (a kind of auxiliary function) `template<typename T, typename F0_str, typename F0, typename... Fn> void class_aux(T& cls, F0_str f0_str, F0 f0, Fn... fn)` used to “unpack” the parameters given in unknown number, which are the methods (and their names in `const char*`) exported from the C++ class. Finally, a third function `template<typename T> void class_aux(T& cls)` is used as base case to terminate the recursion. 

But this little experimentation made me take a step back and think about this API for exporting classes. Indeed, with the previous code, I felt like recoding boost::python with some changes in the syntax but with intrinsically the same interface. Something was going wrong. Finally I realized that implementing a facade pattern for a library like boost::python requires deep knowledges of it in order to produce a work that actually improves the library, making it easier to use it. But obviously
I have not deep knowledges about boost::python and have no idea about what should be improve and what should be keep in this library. In addition, I knew the principal libraries to export C++ code to Python were just boost::python and pybind11. So I was thinking “well, boost::python already implements an interface to export C++ code, why not adapting pybind11 to this interface if we move to it in the future?”. In reality, pybind11 follows the same interface than boost::python, it's just the
implementation that differs (boost::python uses a lot of preprocessor/templates tricks while pybind11 takes advantages of the C++11 features like variadic templates, tuples or lambda functions) and thus, implementing an API for these libraries didn't make sense anymore.

In parallel, it appears that my mentors came at the same conclusion. Indeed, I received an email from them, suggesting me to move my experimentations from boost::python to pybind11 and give them some feedbacks about it. They made a list of advantages/disadvantages about these two libraries that I find interesting to share if you are interested in Python bindings for some C++ code:

- pybind11 is small and lightweight: about 2.5k lines of code distributed in a few header files, ready to be used while boost is a huge library, not so nice to have it as dependency
- pybind11 have direct bindings for std containers, bindings for NumPy
- the code of pybind11 is readable in the case you want to look at it to understand what is going on under the hood, like I said, it's not the case for boost::python

but:

- pybind11 is mainly maintained by only one man (but as the library is new, it should be more easier to contribute to pybind11 than to boost::python (we are not even sure than boost::python is still maintained))
- there is less available documentation for pybind11 than for boost::python (the community is smaller)

That concludes my thoughts about moving from boost::python to pybind11. I had the opportunity to read about metaprogamming in this context and I discovered the following concept: CRTP (for “Curiously Recurring Template Pattern”) that I found interesting to talk about.

### A curiously recurring template pattern

You can find several articles on the Internet talking about the Curiously Recurring Template Pattern (on [stackoverflow](http://stackoverflow.com/questions/4173254/what-is-the-curiously-recurring-template-pattern-crtp), on [Wikipedia](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern) and finally on [Eli Bendersky's website](http://eli.thegreenplace.net/2011/05/17/the-curiously-recurring-template-pattern-in-c) (that I just discovered)). So I'm not going to give you an
other definition of it, basic examples, etc. but just try to explain how I understand the CRTP and talk a little about its place in the DUNE codebase.

So basically, just to have the same starting point, CRTP is this:

```c++
template<class Derived>
class Base
{
    ...
};

class Derived : public Base<Derived>
{
    ...
};
```

If it is indeed recursive in the syntax, it's not recursive in the semantics (no reason to be afraid of a dark infinite loop when running it).

I don't have the knowledge to understand what is going on under the hood (typically how the compiler manages that) but I can try to have a simplified overview that I use to understand and memorize the examples of this concept. It's also a starting point to dig into several directions in order to improve my C++ knowledges.

So basically, I see a class `Base` that defines some methods, attributes, etc. and uses a template parameter named `Derived`. It means that `Base` will be specialized later when the compiler sees a declaration like `Base<Foo>`.

Then, I see a class `Derived` that inherits publicly from `Base<Derived>`. For me it means that `Base` is specialized at this moment and that the code of `Base` is duplicated with the template parameter replaced by `Derived`. That also means that the class `Derived` should be a model for the concepts defined in `Base` (`Derived` should be able to be used as the template parameter of the Base class's definition). It's important to highlight this fact that gives an application of the CRTP:
defining interfaces and making the compiler ensuring that their implementations are consistent. 

The other thing to highlight is that `Derived` (obviously) inherits from `Base` and thus `Derived` inherits all of the public and protected methods and attributes of `Base`. That gives other applications like static polymorphism or mixin clases.

In fact, as `Derived` inherits from `Base` that is specialized by `Derived`, CRTP defines a special relation "parent-child" between these two classes. For each class `Derived1`, `Derived2`, ..., `DerivedN` that inherits from `Base<Derived1>`, `Base<Derived2>`, ..., `Base<DerivedN>`, there is a specialization of `Base` with its derived class `DerivedX` (I see it like N "parent-child" pairs).

This special relation gives sense to this cast: `static_cast<Derived*>(this)` used in the `Base` class and that makes possible to refer to the derived class inside the base class. Because of this cast, CRTP provides ways to deal with static polymorphism and mixin classes as described in [Eli Bendersky's article](http://eli.thegreenplace.net/2011/05/17/the-curiously-recurring-template-pattern-in-c).

The other application of the CRTP I mentionned is to be used as a way to define interfaces and statically insuring the conformity of their implementations. It appears that this application is often used in the DUNE codebase. In fact, the goal of DUNE is to provide a generic interface of the grid concept (see my first post introducing DUNE) and thus, it uses a lot the CRTP to define interfaces and make it possible to the user to define his/her own
implementations while insuring at compile time these implementations are compliant with the interfaces. 

### What's next?

I just received an email from my mentors last night: they managed to implement an initial codebase to spare me the difficulties of starting from scratch. So I'm going to study their code and then work on it to achieve the first milestone we defined, I will talk about that in an upcoming article.
