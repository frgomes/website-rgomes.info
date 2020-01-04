+++
title = "Strong type checking in Python"
date = 2014-02-13T11:36:00Z
[taxonomies]
categories = ["articles"]
tags = ["python", "type check", "sphinx_typesafe", "explained"]
+++
_This article describes a Python annotation which combines documentation with type checking in order to help Python developers to gain better understanding and control of the code, whilst allowing them to catch mistakes on the spot, as soon as they occur._

Being a Java developer previously but extradited to Python by my own choice, I sometimes feel some nostalgy from the old times, when the Java compiler used to tell me all sorts of stupidities I used to do.

In the Python world no one is stupid obviously, except probably me who many times find myself passing wrong types of arguments by accident or by pure stupidity, in case you accept the hypothesis that there's any difference between the two situations.


When you are coding your own stuff, chances are that you know very well what is going on. In general, you have the entire bloody API alive and kicking inside your head. But when you are learning some third party software, in particular large frameworks, chances are that your code is called by something you don't understand very well, which decides to pass arguments to your code which you do not have a clue what they are about.

## Regarding documentation

Documentation is a good way of sorting out this difficulty. Up-to-date documentation, in particular, is the sort of thing I feel extremely happy when I have chance to find one. My mood is being constantly crunched these days, if you understand what I mean. Outdated documentation is not only useless but also undesirable. Possibly for this reason some (or many?) people prefer no documentation at all, since absence of information is better than misinformation, they defend.

## Strong type checking

I'm not in the quest of convincing anyone that strong type checking is good or useful or desirable.  Like everything in life, there are pros and cons. On the other hand, I'd like to present a couple of benefits which keep strong type checking in my wishlist:

* I'd like to have the ability to stop the application as soon as a wrong type is received by a function or returned by a function to its caller. Stop early, catch mistakes easily, immediately, on spot.

* I'd like to identify and document argument types being passed by frameworks to my code, easily, quickly, effectively, without having to turn the Internet upside down every time I'm interested to learn what argument x is about.

## Introducing sphinx_typesafe

Doing a bit of research, I found an interesting library called IcanHasTypeCheck (or ICHTC for short), which I ended up rewriting almost from scratch during the last revision and I've renamed it to sphinx_typesafe. Let me explain the idea:

* In the docstring of a function or method, you employ Sphinx-style documentation patterns in order to tell types associated to variables.

* If your documentation is pristine, the number of arguments in the documentation match the number of arguments in the function or method definition.

* If your logic is pristine, the types of arguments you documented match the types of arguments actually passed to the function or method at runtime, or returned by the function or method to the caller, at runtime.

* You just need to add an annotation @typesafe before the function or method, and sphinx_typesafe checks if the documentation matches the definition.

* If you don't have a clue about the type of an argument, simply guess some unlikely type, say: None. Then run the application and sphinx_typesafe will interrupt the execution of it and report that the actual type does not match None. The next step is obviously substitute None by the actual type.


## Benefits

A small example tells more than several paragraphs. Imagine that you see some code like this:
```pyrhon

import math

def d(p1, p2):
    x = p1.x - p2.x
    y = p1.y - p2.y
    return math.sqrt(x*x + y*y)
```

Imagine that you had type information about it, like this:
```python
import math
from sphinx_typesafe import typesafe

@typesafe
def d(p1, p2):
    """
    :type p1: shapes.Point
    :type p2: shapes.Point
    :rtype  : float
    """
    x = p1.x - p2.x
    y = p1.y - p2.y
    return math.sqrt(x*x + y*y)
```

Now you are able to understand what this code is about, quickly!.

In particular, you are able to tell what it is the domain of types this code is intended to operate on. When you run this code, if this function receives a shapes.Square instead of a shape.Point, it would stop immediately. Notice that, eventually, a shape.Square may have components x and y which would make the function return wrong results silently. Imagine your test cases catching this situation!

So, I hope I demonstrated the two benefits I was interested on.

## Missing Features

### Polymorphism

Sometimes I would like to tell that an argument can be a file but also a str. At the moment I can say that the argument can be types.NotImplementedType meaning "any type". But I would like something more precise, like this:

    :type f: [file, str]

This is not difficult to implement, actually, but we are not there yet.

### Non intrusive

I would like to have a non intrusive way to turn on type checking and a very cheap way of turning off type checking, if possible without any code change.

Thinking more about use cases, I guess that type checking is very useful when you are developing and, in particular, when you are running your test suite. You are probably not interested on having the overhead of type checking on production code which was theoretically exhaustively tested.

Long story short, I would like to integrate sphinx_typesafe with pytest, so that an automatic decoration of functions and methods would happen automagically and without any code change.

If pytest finds a docstring which happens to contain a Sphinx-style type specification on it, @typesafe is applied to the function or method. That would be really nice! You could also run your code in production without type checking since type checking was never turned on in the first place.

The idea looks to be great, but my ignorance on pytest internals and my limited time prevents me of going ahead. Maybe in future!

### Python3 support

The sources of sphinx_typesafe itself are ready for Python3, but sphinx_typesafe does not handle properly your sources written in Python3 yet. It's not difficult to implement, actually: it's just a matter of adjusting one function, but we are not there yet. Maybe you feel compelled to contribute?

## References

https://pypi.python.org/pypi/sphinx_typesafe


## Credits

Thanks Klaas for inspiration and his IcanHasTypeCheck (or ICHTC for short).


----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
