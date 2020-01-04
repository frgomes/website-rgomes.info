+++
title = "Using TypeTokens to retrieve generic parameters"
date = 2011-01-03T20:16:00Z
[taxonomies]
categories = ["articles"]
tags = ["java", "generics", "type erasure", "reify", "design patterns", "typetoken", "anonymouns class", "explained"]
+++
_Super Type Tokens, also known by Type-safe Heterogenerous Container (or simply THC) is very well described in [article by Neal Gafter](http://gafter.blogspot.com/2006/12/super-type-tokens.html), who explains how Super Type Tokens can be used in order to retrieve Run Time Type Information (RTTI) which would be erased otherwise, in a process known as [type erasure](http://download.oracle.com/javase/tutorial/java/generics/erasure.html)._

## Overview

Contrary to what is believed and widely accepted, _type erasure can be avoided_, which means that the callee has ability to know which generic parameters were employed during the call.

There are circumstances where you'd like to have a class which behaves in different ways depending on generic parameters. For example:

Imagine a list which would not rely on Java Collections Framework but on array of primitive types in reality, since performance would be much better than JCF classes. So, you'd like to tell ``List`` that it should allocate an array of ints ``int[]`` or an array of doubles ``double[]`` , depending on a generic parameter you specify, intead of employing ``List<Integer>`` or ``List<Double>``. Something like this:
```java
List<Integer> myList = new PrimitiveList<Integer>()
```
... would be backed by ``int[]`` whilst
```java
List<Double> myList = new PrimitiveList<Double>()
```
... would be backed by a ``double[]``.

## The problem: Type Erasure

When Generics was implemented in Java5, it was decided that this feature would be offered by ``javac`` (the Java compiler) and only very minimum changes would be implemented in other components of the architecture. The big benefit of this decision is that the implementation of this feature was relatively simple and imposed only relatively minimum risk to existing Java applications, guaranteeing compatibility and stability of the Java ecosystem as a whole.

The problem with this implementation is that only ``javac`` knows about generic types you specified in your source code. This knowledge exists only at compilation time. At run time, your callee class has no clue what generic parameters were employed during the call. It happens because information relative to generic types is lost in a process known as [type erasure](http://download.oracle.com/javase/tutorial/java/generics/erasure.html), which basically means that ``javac`` does not put type information it has at compilation time in the bytecode, which ultimately means that your running application does not know anything about type information you've defined in your source code.

Confused? Well ... it basically means that the code below is not possible:
```java
// this code does not compile

class MyClass<T> {
    private final T o;

    public MyClass() {
        this.o = new T();
    }
}
```
... because at run time ``MyClass`` does not know anything about the type of generic parameter ``T``. In spite ``javac`` at compile time is able to perform syntax and semantics validation of your source code, at run time all information regarding generic type T is thoroughly lost. In this case, in particular, the compiler knows that ``T`` is erased at runtime and so, in this case, the compiler rejects the code.

However, in reality, the previous statement may not be 100% correct under all circumstances. There's an exception. This is what we will see in the next topic.

## How type erasure can be avoided

When Generics was implemented in Java5, the type system was reviewed and long story short, information about generic types can be made available at run time under some specific circumstances. This is a very important concept to our discussion here:

> Generic types are available to anonymous classes.

### What an anonymous class is?

Let's debate a little bit what an anonymous class is and also what it is not. Let's suppose we are instantiating ``MyClass`` like this:
```java
MyClass<Double> myInstance = new MyClass<Double>() {
        //
        // Some code here
        //
        // In this block we are adding functionality to MyClass
        //
};
```
We are actually creating an instance of ``MyClass``, but also we are _adding some additional code to it_, which is exactly the code enclosed by curly braces.

It means is that we are creating an object ``myInstance`` of an _anonymous class of ``MyClass``_. It does not mean that ``MyClass`` is itself an anonymous class! ``MyClass`` is definitely not anonymous because it has a name (``MyClass``)... the name you have declared it somewhere else, correct?

In the snippet of code above we are using something which is an _extended class_ made from our original definition of ``MyClass`` plus some more logic. This is the anonymous class we are talking about. In other words, the type of variable ``myInstance`` or ``myInstance.getClass()`` was never declared explicitly, which means it is anonymous.

### How javac handles anonymous classes

When ``javac`` finds an anonymous class, it creates data structures in the bytecode (which are available at run time) which holds the actual generic type parameters employed during the call. So, we have another very important concept here:

> The Java compiler employs type erasure when objects are instantiated except when objects are instantiated from anonymous classes.

In other words, our aforementioned ``MyClass`` does not know any type information when it is called like this
```java
MyClass<Double> myClass = new MyClass<Double>();
```
... but it does know generic type information when it is called like this:
```java
MyClass<Double> myClass = new MyClass<Double>() { /* something here */ };
```
In order to obtain generic type information at run time, you do have to change the call, in order to employ an anonymous class made as an extension of your original class and not made from your original class directly.

In the next topic we will cover what needs to be done in your implementation of ``MyClass`` in order to retrieve generic type information, but a very important concept is that it only works if you call an anonymous class made as an extension of your defined class. So:
```java
MyClass<Double> myClass1 = new MyClass<Double>();     // type erasure DOES happen
MyClass<Double> myClass2 = new MyClass<Double>() { }; // type erasure DOES NOT happen!
```
Notice that the sole requirement is that you need to have an anonymous class. If you don't have any additional logic to be added to the existing logic already present in your ``MyClass``, you just need to to declare an empty block delimited by curly braces.

### Classical solution

Let's review what we are interested here: we are interested on generic types, which are, well... types. Observe that types are ultimately class definitions. So, we would like to give our class ``MyClass<T>`` the ability to know that its ``T`` generic parameter is actually ``T.class``. A classical solution would be simply passing what we need during the call. This is something like this:
```java
MyClass<Double> myClass = new MyClass<Double>(Double.class);
```
Observe that this is not a very good solution because you have to type ``Double`` three times: (1) when you define the type, (2) when you pass the generic parameter and (3) when you pass the formal parameter ``Double.class``. It looks too verbose and too repetitive, isn't it?

Anyway, this is what the great majority of developers do. They simply tell that ``Double`` is generic parameter and then they tell ``Double.class`` just after as a formal parameter during the call. In spite it works, the code does not look optimal and it even may lead to bugs later when your application becomes bigger and you start to refactor things, etc.

### More flexible solution

We already visited a classical solution for the problem of type erasure and we had already seen how an anonymous call can be done. Now we need to understand how generic types can be retrieved at run time without having to pass ``Double``so many times as we did in our classical solution above.

Going straight to the point, let's define an skeleton of our ``MyClass`` which does the job we need. Joining some ideas from the classical solution and using some incantation offered by a class called ``TypeTokenTree``. Below we explain the general concept:
```java
import org.jquantlib.lang.reflect.TypeTokenTree;

public class MyClass<T> {

    private final Class<?> typeT;

    public MyClass(final Class<?> typeT) {
        this.typeT = typeT;
        init();
    }

    public MyClass() {
        this.typeT = new TypeTokenTree(this.getClass()).getElement(0);
        init();
    }

    private init() {
        // perform initializations here
    }
}
```

The code above allows you to call MyClass employing 2 different strategies:
```java
MyClass<Double> myClass1 = new MyClass<Double>(Double.class); // classical solution
MyClass<Double> myClass2 = new MyClass<Double>() { };         // only sorcerers do this
```

Notice that object ``myClass1`` employs the classical solution we described, which is what the great majority of developers do. The object ``myClass2`` was created using the incantation explained in this article and we will explain it better below.

### Digging the solution

Class ``TypeTokenTree`` is a helper class which returns the Class of the n-th generic parameter. In the line
```java
this.typeT = new TypeTokenTree(this.getClass()).getElement(0);
```
... we are building an instance of ``TypeTokenTree``, passing the __actual class of the current instance__ and asking for the 0-th generic type parameter.

Observe what we've written in bold: the actual class of the current instance may be or may not be MyClass. Got it? Observe that the actual class of the current instance will not be ``MyClass`` if you employed an anonymous call. In this case, i.e: when you have an anonymous call... ``javac`` generates code which keeps generic type information available in the bytecode. Notice that:

> ``TypeTokenTree`` fails when a non-anonymous call is done!

This is OK. Actually, there's no way to be anything different from that!. It's an application's responsibility to recover from such situation.

In the references section below you can find links to class ``TypeTokenTree`` and another class it depends on: ``TypeToken``. These files are implemented as part of [JQuantLib](http://www.jquantlib.org) and contain code which is specific to JQuantLib and may not be convenient for everyone. For this reason, we can see below modified versions of these classes which aims to be independent of JQuantLib and aims to explain in detail how the aforementioned incantation works.

First of all, you need to have a look at method [getGenericSuperclass](http://download.oracle.com/javase/6/docs/api/java/lang/Class.html#getSuperclass%28%29) from the JDK. This method is basically the root of the incantation and it basically traverses data structures created in the bytecode by ``javac``. These data structures provide type information regarding the generic types you employed. In general, ``getGenericSuperclass`` returns ``null``, which means that the current instance belongs to a non-anonymous class. In the rare circumstances you employ anonymous classes, ``getGenericSuperclass`` will return something different of ``null``. And this is how we do this magic.

When ``getGenericSuperclass`` does not return ``null``, you have opportunity to traverse the data structure ``javac`` created in the bytecode and you can discover what was available at compile time, effectively getting rid of type erasure.
```java
static public Type getType(final Class<?> klass, final int pos) {
    // obtain anonymous, if any, class for 'this' instance
    final Type superclass = klass.getGenericSuperclass();

    // test if an anonymous class was employed during the call
    if ( !(superclass instanceof Class) ) {
        throw new RuntimeException("This instance should belong to an anonymous class");
    }

    // obtain RTTI of all generic parameters
    final Type[] types = ((ParameterizedType) superclass).getActualTypeArguments();

    // test if enough generic parameters were passed
    if ( pos < types.length ) {
        throw RuntimeException(String.format(
           "Could not find generic parameter %d because only %d parameters were passed",
              pos, types.length));
    }

    // return the type descriptor of the requested generic parameter
    return types[pos];
}
```

### Pros and cons

The big benefit of employing Type Tokens is that the code becomes less redundant, I mean:
```java
MyClass<Double> myClass = new MyClass<Double>() { };
```
... is absolutely enough. You don't need to repeat ``Double`` thre times like this:
```java
MyClass<Double> myClass = new MyClass<Double>(Double.class);
```
On the other hand, the code also becomes obcure, because failing to remember to add the anonymous block will end up on an exception thrown by class TypeToken.
```java
MyClass<Double> myClass = new MyClass<Double>() { }; // succeeds
MyClass<Double> myClass = new MyClass<Double>();     // TypeTokenTree throws an Exception
```

> The point is: this technique is not widely advertised and most developers never heard that this sort of magic be done. If you are sharing your code with your peers, contributors or clients, chances are that you will have to spend some time explaining the magic the code does. In general, developers forget to make the call properly, which leads to failures at runtime, as just explained above.

### Test Cases

OK. Now you visited the theory, you'd like to see how this thing really works. Below you can find some test cases which exercise classes TypeToken and TypeTokenTree'. These test cases cover some varied scenarios and they should be enough to illustrate how the techniques explained here can be used in the real world.

* [TypeTokenTest](http://bazaar.launchpad.net/~frgomes/jquantlib/trunk/view/1362/jquantlib/src/test/java/org/jquantlib/testsuite/lang/TypeTokenTest.java)
* [TypeTokenTreeTest](http://bazaar.launchpad.net/~frgomes/jquantlib/trunk/view/1362/jquantlib/src/test/java/org/jquantlib/testsuite/lang/TypeTokenTreeTest.java)


## References

* [TypeToken.java](http://bazaar.launchpad.net/~frgomes/jquantlib/trunk/view/1362/jquantlib/src/main/java/org/jquantlib/lang/reflect/TypeToken.java)
* [TypeTokenTree.java](http://bazaar.launchpad.net/~frgomes/jquantlib/trunk/view/1362/jquantlib/src/main/java/org/jquantlib/lang/reflect/TypeTokenTree.java)
* [Super Type Tokens](http://gafter.blogspot.com/2006/12/super-type-tokens.html)
* [A Limitation of Super Type Tokens](http://gafter.blogspot.com/2007/05/limitation-of-super-type-tokens.html)

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website.
