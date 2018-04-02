---
layout: post
category: interest
title: Tag dispatching with a selector class
---

Tag dispatch is a method in C++ to call different functions depending on a
runtime tag passed as a parameter to the function.

There are times when you might want to allow a user to provide a selector to
choose the particular implementation of a function, and use a tag to dispatch
the call to the correct implementation. However there are some problems that
arise from trying to use this method while also allowing the user to return a
runtime value from a selector method.

<!--end-excerpt-->

### The original code


{% highlight cpp %}
enum class Algo {
  Impl1,
  Impl2,
};
{% endhighlight %}

{% highlight cpp %}
struct LotsOfParameters {/* ... */};
template<Algo algo>
int function(LotsOfParameters const&);

template<>
int function<Algo::Impl1>(LotsOfParameters const& params) {
  // do stuff with params
  return 1;
}
template<>
int function<Algo::Impl2>(LotsOfParameters const& params) {
  // do other stuff with params
  return 2;
}
{% endhighlight %}

{% highlight cpp %}
class Selector {
 public:
  virtual Algo select(LotsOfParameters const& params) = 0;
};
{% endhighlight %}


{% highlight cpp %}
int call_function(Selector& selector, LotsOfParameters const& params) {
  Algo algo = selector.select(params);
  switch(algo) {
  case Algo::Impl1:
    return function<Algo::Impl1>(params);
  case Algo::Impl2:
    return function<Algo::Impl2>(params);
  }
}
{% endhighlight %}



### Tag dispatch

{% highlight cpp %}
struct Impl1 {};
struct Impl2 {};
{% endhighlight %}


{% highlight cpp %}
struct LotsOfParameters {/* ... */};
template<typename T>
int function(LotsOfParameters const&);

template<>
int function<Impl1>(LotsOfParameters const& params) {
  // do stuff with params
  return 1;
}
template<>
int function<Impl2>(LotsOfParameters const& params) {
  // do other stuff with params
  return 2;
}
{% endhighlight %}

{% highlight cpp %}
template<typename ImplType>
int call_function(LotsOfParameters const& params, ImplType) {
  return function<ImplType>(params);
}
{% endhighlight %}

{% highlight cpp %}
int main() {
  LotsOfParameters params{};

  Impl1 impl1_tag{};
  // Will call function<Impl1> and assign x to 1
  int x = call_function(params, impl1_tag);

  Impl2 impl2_tag{};
  // Will call function<Impl2> and assign y to 2
  int y = call_function(params, impl2_tag);

  return x + y;
}
{% endhighlight %}

You can see that this works [on compiler explorer][tag-dispatch-mk1], and that
both gcc and clang can inline the function calls, so that the main function
will just return 3 and not need to call the functions explicitly.

### A selector class

The original selector class could return enum values, however in order to use
tag dispatching a new selector class must return different types for different
implementations. This is where problems start to sneak in, as it requires a
single method to return multiple different types.

In C++ one way to return different types from the same method is to use
inheritance, and return different derived classes of the same base class.

{% highlight cpp %}
struct Algo {};
struct Impl1 final : public Algo {};
struct Impl2 final : public Algo {};

struct LotsOfParameters {
  int param1;
  // ...
};
{% endhighlight %}

{% highlight cpp %}
class Selector {
 public:
  virtual Algo const& select(LotsOfParameters const& params) = 0;
};
{% endhighlight %}

{% highlight cpp %}
class Always1 final : public Selector {
 public:
  Algo const& select(LotsOfParameters const& params) {
    static Impl1 constexpr impl1;
    return impl1;
  }
};
class Even1Odd2 final : public Selector {
 public:
  Algo const& select(LotsOfParameters const& params) {
    static Impl1 constexpr impl1;
    static Impl2 constexpr impl2;
    if(params.param1 % 2 == 0) {
      return impl1;
    } else {
      return impl2;
    }
  }
};
{% endhighlight %}

### Inheritance and tag dispatching

Putting together everything we've looked at so far you might think that we had a
solution to the original problem.

{% highlight cpp %}
int some_function() {
  LotsOfParameters params{1};
  Always1 sel{};
  Algo const& tag = sel.select(params);
  // Does not call function<Impl1> as expected, but function<Algo>
  return call_function(params, tag);
}
{% endhighlight %}

However looking at the [the code][not-compiling] we see that the `ImplType`
template deduced in `call_function` is not the derived tag type, but the base
`Algo` type. Therefore no matter which `Algo` type the selector returns the base
type is always used.

### Double dispatch to the rescue

In order to force the compiler to make use of the actual type of the tag, rather
than the common base type, we need to make use of the tools provided by
polymorphism. The easiest way to do this is use a virtual function call on the
tag reference, and so force a lookup through the tag's vtable.

{% highlight cpp %}
struct Algo {
  virtual int call_function_impl(LotsOfParameters const&) const = 0;
};
struct Impl1 final : public Algo {
  int call_function_impl(LotsOfParameters const&) const override;
};
struct Impl2 final : public Algo {
  int call_function_impl(LotsOfParameters const&) const override;
};
{% endhighlight %}

And later on, after defining the `function` implementations:

{% highlight cpp %}
int Impl1::call_function_impl(LotsOfParameters const& params) {
  return function<Impl1>(params);
}
int Impl2::call_function_impl(LotsOfParameters const& params) {
  return function<Impl2>(params);
}
{% endhighlight %}

Now our main `call_function` function has to use the passed in tag to call the
virtual `call_function_impl` method which goes through the tag's vtable and ends
up calling the correct implementation of the `function`. Note that the tag has
to be passed by reference, as the `ImplType` will still be deduced to be `Algo`,
but that is an abstract class and so cannot be instantiated.

{% highlight cpp %}
template<typename ImplType>
int call_function_with_tag(LotsOfParameters const& params, ImplType const& tag) {
  return tag.call_function_impl(params);
}
{% endhighlight %}

Have a look at the generated code in [compiler explorer][double-dispatch] and
notice that both gcc and clang will devirtualize the function calls and so can
still inline everything.

### Putting it all together

In our original code the user facing interface consisted of a single function
which took the selector and parameters, then used those to choose the correct
implementation of the algorithm to run. Now we are in a position to provide
something similar.

{% highlight cpp %}
int call_function(Selector& selector, LotsOfParameters const& params) {
  Algo const& tag = selector.select(params);
  return tag.call_function_impl(params);
}
{% endhighlight %}

Because of the polymorphism of the Selector class this function does not need
to be templated and the implementation could be hidden from the user in a
precompiled library. However the extra virtual call in the Selector can
introduce additional overhead and if you are happy exposing some of the
implementation to the user, a templated function will allow the compiler to
inline more and prevents the indirect call through the Selector's vtable:

{% highlight cpp %}
template <typename Selector>
int call_function(Selector& selector, LotsOfParameters const& params) {
  Algo const& tag = selector.select(params);
  return tag.call_function_impl(params);
}
{% endhighlight %}

### Adding new implementations

One of the main advantages to using tag dispatching over a switch and enum is
that it is much easier to add new implementations. In fact all you have to do is
provide a new derived class of `Algo` and the implementation of `function`.

{% highlight cpp %}
struct NewImpl final : public Algo {
  int call_function_impl(LotsOfParameters const& params) const override;
};
template<>
int function<NewImpl>(LotsOfParameters const&) {
  return 100;
}
int NewImpl::call_function_impl(LotsOfParameters const& params) const {
  return function<NewImpl>(params);
}
{% endhighlight %}

Then any selector which returns your new implementation tag will automatically
call the new implementation of the function.

{% highlight cpp %}
class NewSelector final : public Selector {
 public:
  Algo const& select(LotsOfParameters const&) override {
    static NewImpl constexpr impl;
    return impl;
  }
};
int use_new_implementation() {
  LotsOfParameters params{0};
  NewSelector selector{};
  return call_function(selector, params);
}
{% endhighlight %}

### Concluding thoughts

You can see everything put together on the [compiler explorer][all-together].
Note that clang does a great job of devirtualising everything and so most of the
functions are able to instantly return the `int` specified by the function
implementations. On the other hand gcc is less able to devirtualise the multiple
function calls and so does not have quite as nice output.

In general use this will not matter a huge deal, as the idea behind this
technique is to allow a library developer to hide away implementation details of
complicated functions. This means that a typical usecase will have much more
complex function implementations which would probably not get inlined by the
compiler, and may in fact be designed to be in different translation units to
the user compiled code.

[tag-dispatch-mk1]: https://godbolt.org/g/YfrKQk
[enum-switch]: https://godbolt.org/g/A4xxMc
[not-compiling]: https://godbolt.org/g/m462q7
[double-dispatch]: https://godbolt.org/g/uRvK2Y
[all-together]: https://godbolt.org/g/hocFv8
