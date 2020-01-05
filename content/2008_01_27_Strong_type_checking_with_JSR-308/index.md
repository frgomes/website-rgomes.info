+++
title = "Strong type checking with JSR-308"
date = 2008-01-27T21:22:00Z
[taxonomies]
categories = ["articles"]
tags = ["java", "type check", "jquantlib"]
+++
_In this article we demonstrate that strong type checking sometimes is not enough for the level of correctness some applications need. We will explore a situation where we semantic checking is needed to achieve higher levels of correctness. Then we will explorer how something like this can be done in C++ and how future improvements in JDK will allow us to do it in Java, obtaining better results than C++ can offer._

## Problem

Imagine the following code snippet:
```java
  double calc(double rate, double year) {
    return Math.exp(1+rate,year);
  }

  double rate = 0.45;
  double year = 0.5;

  double result1 = calc(rate, year);
  double result2 = calc(year, rate);
```
Notice the two calls of the method calc. Yet syntactically and semantically correct for the compiler, humans can easily determine that something will probably go wrong. Ideally, we'd like the compiler warn us about the error in order to shorten the development cycle.

### The C++ solution

In C++ we can avoid such mistakes by...
```cplusplus
typedef double Rate;
  typedef double Year;

  double calc(Rate rate, Year year);
```
If you are using a powerful IDE, it will tell you what are the correct order of arguments and you will be able to avoid mistakes.

Important: The C++ compiler still does not prevent you to pass arguments in the wrong order. You will not get a compilation error if you pass arguments in the wrong order!

### The Java solution

In Java you dont have typedefs. One possible alternative would be defining mutable objects intended to hold primitive types but this solution is very inefficient. I will avoid spending your time visiting all the candidate solutions and ending up showing you they are not adequate for our needs. Let's go directly to what we need:

In the upcoming JDK7, we will have the possibility to use annotations in accordance with JSR-308. In a nutshell, it allows you to use annotations wherever you have a type in your code, and not only on declarations of classes, methods, parameters, fields and local variables. Let's examine an example:
```java
private Double calc(@Rate double rate, @Time double time) {
  return new Double( Math.exp(1+rate, time) );
}

public method test() {
  @Rate double rate = 0.45;
  @Time double time = 0.5;

  // This call pass
  Double result1 = calc(rate, time);

  // This call *should* give us a compiler error
  Double result2 = calc(time, rate);
}
```
In the above specific example the code compiles fine in JDK6 but does not offer us the strong type checking it will be possible to use with JDK7.

If you have a nice IDE, it will tell you the correct order of parameters, showing you ``@Rate double`` instead of simply `double`.

Much like the C++ compiler, which was not able to detect the wrong order or parameters, javac will suffer from the same illness and will not detect the wrong order of parameters because annotations are meaningless to javac.

On the other hand, javac is able to execute annotation processors you specify in the command line. In particular, we can write an annotation processor which verifies if you are passing a ``@Rate`` where a ``@Rate`` is expected and so on. It means that JDK7 with help of JSR-308 annotation processors is able to detect the mistake we pointed out in the beginning of this article.

Going ahead, with JDK7 we can produce code with all the very strong type checkings we need to obtain very robust code. See the example below:
```java
private @NonNull Double calc(@Rate double rate, @Time double time) @ReadOnly {
  // The following statement will fail because the return type cannot be null 
  if (condition) return null;
  // The following statement pass
  return new Double( Math.exp(1+rate, time) );
}

public method test() {
  @Rate double rate = 0.45;
  @Time double time = 0.5;

  // This call pass
  Double result1 = calc(rate, time);

  // This statement will fail because the receiver of calc became read-only
  result = 1.0;

  // This call *should* give us a compiler error
  Double result2 = calc(time, rate);
}
```

## Conclusion

Thanks to new upcoming features of JDK7, programmers can now obtain robust code with very strong type checkings in Java which compare the quality of code written in C++.


## See also

* [JSR-308](http://groups.csail.mit.edu/pag/jsr308/)
* [Java Annotation Design](http://groups.csail.mit.edu/pag/jsr308/java-annotation-design.html)

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
