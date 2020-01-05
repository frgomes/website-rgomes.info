+++
title = "A better implementation of Visitor pattern"
date = 2011-01-29T17:16:00Z
[taxonomies]
categories = ["articles"]
tags = ["java", "visitor", "pattern", "design patterns", "explained"]
+++
_The Visitor pattern is possibly the most complicated design pattern you will face. Not only explanations, implementations and examples you may find on textbooks and articles on the Internet are confusing in general and many times divergent from one another, but also the definition of what the Visitor Pattern is can be many times obscure and rarely explained properly with examples and applications in the real world._

## Visitor Pattern: which one?

In fact, several variations of what we call Visitor Pattern do exist. Many times these variations are simply called _Visitor Pattern_, without any suffixes which would be necessary to differentiate them. This certainly causes confusion. In addition, lots of articles about this pattern fail to explain properly what it is, how it works and how it should be employed in real world applications. Add to that some intrinsic complexity this pattern involves and sometimes confusion caused by multiple nomenclatures or definitions adopted.

This article aims to present a clear, non-ambiguous and non-misleading definition of what the Visitor Pattern is, with examples in the real world. We will demonstrate how a pair of interfaces is plenty enough for solving a vast range of problems, without any need of any modification in this pair of interfaces in order to accommodate unforeseen situations. Complex use cases can be addressed by defining additional interfaces, which extend the original concept, but without changing the original pair of interfaces in any way.

## Definition

From Wikipedia we read:

> In object-oriented programming and software engineering, the Visitor Design Pattern is a way of separating an algorithm from an object structure on which it operates. A practical result of this separation is the ability to add new operations to existing object structures without modifying those structures.

Let's clarify a little bit this definition: In object-oriented programming and software engineering, the Visitor Pattern is a way of separating a _computation_ from the _data structure_ it operates on.

In particular, we will employ the word _computation_ instead of _algorithm_ because the former is more restrictive and more specific than the later. This article shows that obtaining elements from a data structure also involves an algorithm, which could cause confusion. For this reason, it's better to simply avoid the word algorithm entirely.

## Interpretation

The definition says that the Visitor Pattern separates concerns, i.e:

* on one hand you have data access which knows how a data structure can be traversed;
* on the other hand you are interested on a certain computation you need to be executed.

Besides, there's a very important restriction which is: you cannot change the original data structure. In other words:

* the data access knows the data structure but does not know anything about the computation to be performed;
* the computation knows how to operate on a certain element of the data structure but doesn't know anything about the data access needed to traverse the data structure in order to obtain the next element, for example;
* you are not allowed to amend the data structure in any way: imagine a data structure which belongs to a third-party library, which you don't have the source code;

A key point here is the introduction of the concept of data access. In other words, we are saying that the data structure is not accessed directly, but via some sort of data access logic. This will become clear later.

## Use cases

Let's invent scenarios which slowly evolve in complexity. Our aim is to demonstrate why the Visitor Pattern is needed and how it behaves and operates.

### Scenario 1: simple array

Suppose we have a very simple array of integers ``Integer[]``. We need to visit all elements and calculate the sum of all these elements.

Anyone will immediately think on something like below, isn't it?

```java
public int sumArray(Integer[] array) {
    int sum = 0;
    for (int i=0; i<array.length; i++) sum += array[i];
    return sum;
}
```

Yes. It solves the problem. Actually, the problem is so simple that you must be asking yourself why we need anything more complicated than a simple one liner. Isn't it?

OK. Let's go ahead to another use case.

### Scenario 2: different data structures

This scenario is pretty similar to the previous one, but now we can have either an array of integers ``Integer[]``, a list of integers ``List<Integer>`` or a map which contains integers ``Map<T, Integer>``. In this case, one immediately thinks on something like this:

```java
public int sumArray(Integer[] array) {
    int sum = 0;
    for (int i=0; i<array.length; i++) sum += array[i];
    return sum;
}

public int sumList(List<Integer> list) {
    int sum = 0;
    for (int i=0; i<list.size(); i++) sum += list.get(i);
    return sum;
}

public <T> int sumMap(Map<T, Integer> map) {
    int sum = 0;
    for (Entry<T, Integer> key : map.entrySet()) {
    sum += map.get(key);
    }
    return sum;
}
```

OK. In this use case we can start to understand that there are different structures: ``Integer[]``, ``List<Integer>`` and ``Map<T, Integer>`` but, in the end, all you need to do is simply obtain an individual ``Integer`` from whatever data structure it is and simply add it to variable ``sum``.

Let's complicate things a little more now.

### Scenario 3: different data structures and several computations

This scenario is pretty similar to the previous one, but now we have several different computations: sum of all elements, calculate mean value of them, calculate the standard deviation, tell if they form an arithmetic progression, tell if they form a geometric progression, tell if they are all prime numbers, etc, etc, etc.

Well, in this case, you can figure out what will happen: you will face an explosion of methods. Multiply the quantity of computations you are willing to provide by the number of different data structures you are willing to support. This explosion of methods you will have to define may not be desirable.

The point is: it does not make sense to implement the same logic over and over again just because the data structure is slightly different. Even when data structures are very different, what matters is that we are willing to perform a certain computation on all elements stored in it, does not matter if it is a plain array, a map, a tree structure, a graph or whatever.

> The computation only needs to know what needs to be done once an element is at hand; the computation does not need to know how an element can be obtained.

Another aspect is that it may not be desirable or may not be convenient to touch existing code, or existing data structures, to be more precise.

### Scenario 4: do not touch data structures

Imagine now that you have the problem proposed by scenario 3, but you cannot touch any data structures. Imagine that it's a legacy library in binary format; imagine that you don't have the source code. You simply cannot add a single field anywhere in order to keep partial computations or keep response values. You cannot change the provided source code in any way simply because you don't have it. You obviously cannot add any methods intended to perform the computation, because you don't have the source code.

What would be a solution in this case? Well... there's only one: you will have to implement something, whatever it is, outside the provided legacy library.

### A note on memory allocation

Imagine that a certain ``computation`` requires temporary variables or partial results, possibly associated to each element of the original data structure (or several data structures). These additional variables are only needed when you are performing the ``computation`` itself; they are not needed anywhere else. You could store these temporary variables or partial results in the ``data structure``. However, in certain constrained scenarios, you simply don't have enough memory for appending these additional variables to each element of the ``data structure``. In this situation, you would like to have these additional variables allocated only once, on the ``computation`` side itself, not on the ``data structure`` side.

### Conclusion of Use Cases

You may not be allowed to change the original source code in regards to the data structure (or data structures) you have. Or this may not desirable, like in the case of proliferation of methods we mentioned before. This may not be even possible, like in the case of the provided legacy library. This is definitely the most important aspect of the Visitor Pattern: you are not allowed to change the original data structures in any way.

This aspect is commonly ignored by many people posting on the Internet and many times overseen in many academic works. It's very common to see implementations where data structures are changed, like when the object model is changed; or data structures suddenly start to implement new interfaces, etc.

> If you find articles proposing changes in the original data structures, does not matter how, you can be sure that these articles are misunderstanding what Visitor Pattern really means.

## Introducing the Visitor Pattern

It's time to present how the Visitor Pattern works.

The Visitor Pattern is defined as a pair of interfaces: a ``Visitor`` and a ``Visitable``. There's also some additional "glue" code needed when you are about to call implementations of these interfaces.

Let's learn by example, showing how it can be applied to some real world situations.

### A naive Visitor

> Remember: a visitor defines a computation

The idea here is that we need an interface which defines how a computation works in respect to a single data element. It does not matter how data elements are organised in the data structure at this point. Once the interface is defined, we then define a class which performs the computation we need. Since the interface does not know anything about how data elements are organised in data structures, the implementing class does not have any dependency on how data is organised. The class only cares about details strictly related to how the computation needs to be done; not what needs to be done in order to obtain data elements.

For the sake of brevity, lets present the Visitor interface and demonstrate only one of the several computations we may be interested:

```java
public interface Visitor<T> {
    public void visit(T element);
}

private class Sum<T extends Integer> implements Visitor<T> {
    private int sum = 0;

    @Override
    public void visit(T element) {
        sum += element;
    }

    public T value() {
        return sum;
    }
}
```

> The interface ``Visitor`` defines a method which is responsible for receiving a single element from the data structure.

Classes which implement the Visitor interface are responsible for defining the ``computation``, receiving one element at a time, without caring about how such element was obtained in the first place.

### A naive Visitable

> Remember: a Visitable defines data access

The idea is that we need an interface which defines how data elemantes can be obtained in general. Once the interface is defined, we then define a class which knows how data elements can be obtained from a given data structure in particular. It means to say that, if we have several different data structures, we will need several classes implementing the ``Visitable`` interface, one class for each data structure involved.

Notice that nothing was said about the computation. It's not responsibility of interface ``Visitable`` anything involving the computation: it only cares about how single data elements can be obtained from a given data structure.

For the sake of brevity, lets present the Visitable interface and demonstrate only one of the several possible data access expedients we may eventually need:

```java
public interface Visitable<T> {
    public void accept(Visitor<T> v);

}

private class VisitableArray<T extends Integer> implements Visitable<T> {
    private final T[] array;

    public VisitArray(final T[] array) {
        this.array = array;
    }

    @Override
    public void accept(Visitor<T> v) {
        for (int i=0; i<array.length; i++) {
            v.visit(array[i]);
        }
    }
}
```

> The interface ``Visitable`` defines a method which is responsible for accepting an object which implements interface ``Visitor``.

A ``Visitable`` does not know anything about the ``computation``: it knows how a single data element must be passed to another class which is, in turn, responsible for performing the ``computation``. Classes implementing interface ``Visitable`` are responsible for traversing the data structure, obtaining a single data element at a time. Also, once a single element is obtained, the class knows how another step of the computation can be triggered for the current data element at hand.

## Putting it all together

Your code must now perform the following steps:

* instantiate a Visitor;
* instantiate a Visitable
* bootstrap a Visitable with a Visitor;
* request the result of the computation;

It looks like this:

```java
public int computeSumArray(final Integer[] array) {
    // instantiate a Visitor (i.e: the computation)
    final Visitor<Integer>   visitor   = new Sum<Integer>();

    // instantiate a Visitable (i.e: the data access to the data structure)
    final Visitable<Integer> visitable = new VisitArray<Integer>(array);

    // bootstrap a Visitable with a Visitor
    visitable.accept(visitor);

    // returns value computed by the Visitor
    return ((Sum)visitor).value();
}
```

Now imagine you'd like to calculate the mean of an array of integers ``Integer[]``. All you need to do is defining a class called ``Mean`` and reuse everything else, like this:

```java
public int computeMeanArray(final Integer[] array) {
    // instantiate a Visitor (i.e: the computation)
    final Visitor<Integer>   visitor   = new Mean<Integer>();

    // instantiate a Visitable (i.e: the data access to the data structure)
    final Visitable<Integer> visitable = new VisitArray<Integer>(array);

    // bootstrap a Visitable with a Visitor
    visitable.accept(visitor);

    // return value computed by the Visitor
    return ((Mean)visitor).value();
}
```

Now imagine you'd like to obtain the mean value from a list of integers ``List<Integer>`` instead. All you need to do is defining a specific data access for this data structure and reuse everything else, like this:

```java
public int computeMeanList(final List<Integer> list) {
    // instantiate a Visitor (i.e: the computation)
    final Visitor<Integer>   visitor   = new Mean<Integer>();

    // instantiate a Visitable (i.e: the data access to the data structure)
    final Visitable<Integer> visitable = new VisitList<Integer>(list);

    // bootstrap a Visitable with a Visitor
    visitable.accept(visitor);

    // return value computed by the Visitor
    return ((Mean)visitor).value();
}
```

### Extended data structures

Now imagine a situation where you need to aggregate fields to a provided legacy data structure, which you don't have access to the source code.

For the sake of simplicity, let's assume there's a way to unmistakably identify data elements: for arrays, it could be employing an index, for maps, trees and more complex data structures, it could be employing some sort of key.

In the example below we'd like to take coordinates of capitals of countries whilst our data structure only contains country names but do not contain coordinates of their capitals. The solution consists on retrieving coordinates as part of the method ``accept``. In the real world, it may be useful to keep additional fields stored in a data structure which keeps complimentary information relative to the original data structure. This is what we show below:

```java
private class VisitableCapitalCoordinates<T extends Coord> implements Visitable<T> {
    private final T[] countries;
    private final Map<T,Coordinate> capitals; // for storage/caching, if required

    public VisitableCapitalCoordinates(
            final T[] countries; 
            final /* @Mutable */ Map<T,Coordinate> capitals) {
        this.countries = countries;
        this.capitals  = capitals;
    }

    @Override
    public void accept(Visitor<T> v) {
        for (int i=0; i<array.length; i++) {
            String country = countries[i];
            T coord = capitals.put(country, coord(country)) 
            v.visit(coord);
        }
    }


    //
    // private method: not defined in the interface
    //

    private Coordinate coord(T country) {
       return obtainCoordinateOfCapitalSomehow(country);
    }

}
```

In the example above, the most important changes happen in the class constructor and in private methods. The method ``accept`` keeps its original signature and only passes an element of the extended data structure, instead of passing an element of the original data structure.

### Bootstraping: a fancy name for "glue" code

In the method constructor shown in the previous topic, we receive two data structures: the original one and the extended one which must be previously created. We show this in the example below:

```java
    final String[] countries = { "USA", "United Kingdom", "France", "Holland", "Belgium" };
    final Map<T,Coordinate> capitals = new HashMap<T,Coordinate>();
     
    // instantiate a Visitor (i.e: the computation)
    final Visitor<Coordinate>   visitor   = new PlotCoordinate<Coordinate>();

    // instantiate a Visitable (i.e: the data access to the data structure)
    final Visitable<Coordinate> visitable = new VisitableCapitalCoordinates<Coordinate>(countries, capitals);

    // bootstrap a Visitable with a Visitor
    visitable.accept(visitor);
```

### A more complicated scenario with polymorphism

We've seen in the previous section that our naive implementation of the Visitor Pattern

* has an interface Visitor which provides the computation;
* has an interface Visitable which provides the data access;
* does not require any modification on existing data structures.

Now let's complicate a little bit more this scenario.

Let's imagine that an investor has an investment portfolio composed of several financial instruments, such as assets, options, bonds, swaps and more exotic financial instruments. Let's suppose we would like to forecast what would be the payoff of this portfolio given certain conditions.

The problem we are facing now is the fact that the data structure is not uniform anymore, like we have seen previously above. Now we have a portfolio which contains several different objects in it. We can think that a portfolio is actually a tree made of several different kinds of nodes in it. Several nodes are leaf nodes, such as an asset, an option or a bond. Some other nodes are not leaf nodes: they aggregate other nodes under it, such as swaps and exotic instruments.

What this use case adds to the initial problem is the fact that now there's not single ``type <T>`` which represents all nodes. To be more precise: even if all nodes derive from a certain base type, we still need to make sure we are handling a such node properly, according to its type, and we are computing its value properly, according to a given type hierarchy. For example:

```java
Object
    Payoff
        ForwardTypePayoff
        NullPayoff
        TypePayoff
            StrikedTypePayoff
                AssetOrNothingPayoff
                CashOrNothingPayoff
                GapPayoff
                PlainVanillsPayoff
```

At first glance, we can think that we could define a pair of classes ``PolymorphicVisitor`` and ``PolymorphicVisitable`` which take the node type as a generic parameter. Unfortunately, this is not the case due to restrictions in the Java programming languese, in spite the idea would make a lot of sense in principle. Once Java Generics applies type erasure in general, our application will have troubles to decide between a ``PolymorphicVisitor<ForwardTypePayoff>`` and ``PolymorphicVisitor<NullPayoff>`` for example, because both will have types erased and will become just ``PolymorphicVisitor<Object>``. Going straight to the solution, without spending any more time explaining all intricacies, we need something like this:

```java
public interface PolymorphicVisitor {
    public <T> Visitor<T> visitor(Class<? extends T> element);
}
```

The interface above provides a method visitor which is responsible for providing the actual ``Visitor`` for the element data. So, once you have the actual ``Visitor``, you can then delegate to it. If the actual visitor is unknown, you can try again using the ``PolymorphicVisitor`` of the base class, and so on, like this:

```java
@Override
public void accept(final PolymorphicVisitor pv) {
    final Visitor<FixedRateBondHelper> v = (pv!=null) ? pv.visitor(this.getClass()) : null;
    if (v != null) {
        v.visit(this);
    } else {
        super.accept(pv);
    }
}
```

We also need to define interface ``PolymorphicVisitable``, which works in conjunction with ``PolymorphicVisitor``, like this:

```java
public interface PolymorphicVisitable {
    public void accept(PolymorphicVisitor pv);
}
```

Notice that the actual ``Visitor`` needs to be obtained given the class type informed, like shown below:

```java
@Override
public <CashFlow> Visitor<CashFlow> visitor(Class<? extends CashFlow> klass) {

    if (klass==org.jquantlib.cashflow.CashFlow.class) {
        return (Visitor<CashFlow>) new CashFlowVisitor();
    }
    if ( klass == SimpleCashFlow.class) {
        return (Visitor<CashFlow>) new SimpleCashFlowVisitor();
    }
    if (klass==Coupon.class) {
        return (Visitor<CashFlow>) new CouponVisitor();
    }
    if (klass==IborCoupon.class) {
        return (Visitor<CashFlow>) new IborCouponVisitor();
    }
    if (klass == CmsCoupon.class) {
        return (Visitor<CashFlow>) new CmsCouponVisitor();
    }
    ...
}
```

## Sample Code

Please clone project JQuantLib from Github:

```bash
    $ git clone https://github.com/frgomes/jquantlib
```

In particular, you are interested on ``CashFlow`` hierarchy, under folder ``jquantlib/src/main/java``:

```bash
org/jquantlib/util/Visitable.java
org/jquantlib/util/PolymorphicVisitable.java
org/jquantlib/util/PolymorphicVisitor.java
org/jquantlib/util/Visitor.java
org/jquantlib/cashflow/Event.java
org/jquantlib/cashflow/FloatingRateCoupon.java
org/jquantlib/cashflow/Coupon.java
org/jquantlib/cashflow/CmsCoupon.java
org/jquantlib/cashflow/CashFlows.java
org/jquantlib/cashflow/FixedRateCoupon.java
org/jquantlib/cashflow/CashFlow.java
org/jquantlib/cashflow/AverageBMACoupon.java
org/jquantlib/cashflow/SimpleCashFlow.java
org/jquantlib/cashflow/CappedFlooredCoupon.java
org/jquantlib/cashflow/Dividend.java
org/jquantlib/cashflow/IborCoupon.java
```

## Conclusion

The implementation of the Visitor Pattern we present here is scalable and flexible. This implementation does not require any particular interfaces or particulard class methods which are specially crafted for a certain application in particular.

We showed that additional interfaces and classes are necessary only when there's polymorphism involved. We purposely defined ``PolymorphicVisitor`` and ``PolymorphicVisitable`` in order to take decisions regarding which should be the correct ``Visitable`` for traversing a specific data structure and what should be the correct ``Visitor`` needed for a compuation, once we have a certain data element in our hands. These decisions cannot be responsibility of the simple ``Visitor`` and ``Visitable`` interfaces introduced previously. For this reason, additional interfaces are definitely needed.

The Visitor Pattern as defined here conforms to the definition presented on the top of this article and does not require any change in existing data structures.

## Caveats

Even with advantages gained with our implementation, the Visitor Pattern still keeps its relative high complexity. In general, code which employs the Visitor Pattern becomes obscure and difficult to be understood, which means that documentation is key in order to keep the code organised and relatively easy to be understood.


----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
