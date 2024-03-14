Title: My favourite C++ footgun
Date: 2021-06-13 13:30

I'm currently learning C++ to be able to do meaningful contributions to
[OpenMW](https://openmw.org), so I've been reading a lot of code, and a 
[couple](https://stroustrup.com/Tour.html)
[of](https://en.wikipedia.org/wiki/The_C%2B%2B_Programming_Language)
[books](https://www.oreilly.com/library/view/more-effective-c/9780321545190/)
on the topic.

But it's in [Effective C++](https://www.oreilly.com/library/view/effective-c-55/0321334876/),
by [Scott Meyers](https://en.wikipedia.org/wiki/Scott_Meyers), Item 17,
that I found the most amazing trigger-happy footgun example:

```C++
processWidget(std::shared_ptr<Widget>(new Widget), priority());
```

[`std::shared_ptr`](https://en.cppreference.com/w/cpp/memory/shared_ptr) is a
[reference counting](https://en.wikipedia.org/wiki/Reference_counting#C++)
implementation: it will automatically destroy the object it owns when there is
no more `std::shared_ptr` owning it.

Yet this code can result in a memory leak.

At first, I thought about a weird interaction between `std::shared_ptr` and
`new`, when the constructor of `Widget` fails, but this case is [handled
by the C++ runtime](https://isocpp.org/wiki/faq/exceptions#ctors-can-throw).

The first trick here is that function parameters evaluation order is unspecified,
meaning that `new Widget` might be called, then `priority()`, then the value
returned by `new Widget` is passed to `std::shared_ptr<Widget>(…)`.
The second one is there since this is C++, `priority()` might raise an exception,
meaning that `new Widget` will be called, but never passed to
`std::shared_ptr`, and thus never deleted!

As [AnyOldName3](https://github.com/AnyOldName3) added after reading this
blogpost, this might be why there is [`std::make_shared`](https://www.cplusplus.com/reference/memory/make_shared/),
with `std::make_shared<Widget>()` being kinda equivalent to `std::shared_ptr<Widget>(new Widget)`.

edit: Made it to [hackernews' frontpage](https://news.ycombinator.com/item?id=27577601), with a web server running on 64MiB of ram and a single core.
