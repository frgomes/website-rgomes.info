+++
title = "Implementation of multiple inheritance in Java"
date = 2008-03-11T00:22:00Z
[taxonomies]
categories = ["articles"]
tags = ["java", "design patterns"]
+++
_In this article we demonstrate how multiple inheritance can be implemented easily in Java._

Sometimes you need that a certain class behave much like two or more base classes. As Java does not provide multiple inheritance, we have to circumvent this restriction somehow. The idea consists of:

* Create an interface for exposing the public methods of a certain class;
* Make your class implement your newly created interface;

At this point, extended classes of your given class can now implement the interface you have created, instead of extending your class. Some more steps ate necessary:

* Supposing you have an application class which needs multiple inheritance, make it implement several interfaces. Each interface was created as explained above and each interface has a given class which implements the interface.
* Using the delegation pattern, make your application class implement all interfaces you need.

For example, imagine that you have

* interface A and its implementation ClassA
* interface B and its implementation ClassB
* class ClassC which needs multiple inheritance from classes ClassA and ClassB

```java
interface A {
    void methodA();
}

interface B {
    void methodB();
}

class ClassA implements A {
    void methodA() { /* do something A */ }
}

class ClassB implements B {
    void methodB() { /* do something B */ }
}

class ClassC implements A, B {

    // initialize delegates
    private final A delegateA = new ClassA();
    private final B delegateB = new ClassB();

    // implements interface A
    @Override
    void methodA() {
        delegateA.methodA();
    }

    // implements interface B
    @Override
    void methodB() {
        delegateB.methodB();
    }
}
```

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
