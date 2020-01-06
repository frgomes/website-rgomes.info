+++
title = "Performance of numerical calculations"
date = 2008-02-10T14:24:00Z
[taxonomies]
categories = ["articles"]
tags = ["java", "jquantlib", "numerical", "performance", "benchmark"]
+++
_In this article we explore the controversial subject of performance of Java applications compared with C++ applications. Without any passion and certainly not willing to promote another flame war, we try to demonstrate that most comparisons are simply wrong because compare essentially very different things. We also present alternative implementations which address most issues._

## Problem

Performance of numerical applications written in Java can be poor.

## Solution

First of all, performance is a subject that becomes impossible to debate if we consider precision to the last millisecond. Even a piece of code written in assembly running in the same computer can present different execution times when run several times, due to a number of factors outside our focus in this article.

Talking about code written in Java, the subject becomes more complex because the same Java code will eventually end up on different assembly code on different platforms. In addition, there are different JVMs for different platforms and different JIT compilers for different platforms.

Running the same Java code running on a home PC and running on a IBM AIX server can result on huge differences in performance, not only because we expect a server to be much faster than a home PC but mainly because there are a number of optimizations present in IBM's JVM specifically targeting the hardware platform which is not available in a stock JVM for a generic home PC.

The tagline _"Java programs are very slow when compared with C++ programs"_ is even less precise than what we just exposed. In general what happens is that very skilled C++ programmers compare their best algorithms and very optimized implementations against some Java code they copied from Java tutorials. Very skilled Java developers know that tutorials potentially work as expected but certainly are not meant to perform well.

Another point to consider is that default runtime libraries provided by Java cannot be expected to perform well. The same applies to other languages, including C/C++. This is true that C/C++ have some widely accepted libraries which perform very well but the point is that you will potentially find something which performs better for the specific hardware platform you have, for the specific problem domain you have.

In the specific situation of Java applications, there are lots of techniques intended to improve performance of the underlying JVM and certainly there are several techniques which can be applied to Java programs in order to avoid operations which are not needed. In order to compare a benchmark test written in C++ against one written in Java, these aspects must be considered, otherwise we will be comparing code written by C++ gurus against code written by Java newbies.

When researching this subject, I've found the references listed below but I haven't spent any time copying source code and eventually changing it or anything else in order to perform the comparison the way I'd like to do. I preferred to adopt a certain "threshold" of I'd say 30%, which means that a difference of 30% or less implies we don't have enough precision. It does not mean that involved parties perform equal. It only means we don't have enough elements to compare with accuracy.

Taking the previous paragraphs into consideration, Java code can be compared with more or less equivalent C++ code:

* [32bit integer arithmetic](http://www.ddj.com/java/184401976?pgno=2)
* [64bit double arithmetic](http://www.ddj.com/java/184401976?pgno=12)

On the other hand, big differences in performance tell us that we should try to identify reasons for poor performance and eventually propose something better.

* [sort algorithms](http://www.ddj.com/java/184401976?pgno=3): 50% slower;
* [list operations](http://www.ddj.com/java/184401976?pgno=4): 2x slower;
* [matrix operations](http://www.ddj.com/java/184401976?pgno=5): 2x to 3x slower;
* [nested loops](http://www.ddj.com/java/184401976?pgno=6): 2x slower;
* [trigonometric functions](http://www.ddj.com/java/184401976?pgno=14): poor!

Analyzing the previous list, we can identify that:

* Data structures perform badly: list operations (one dimensional data structures) are 2 times slower whilst matrices can be even 3 times slower.
* Sort algorithms involve data structures and certainly are affected by slowness of data structures.
* Trigonometric functions are not a major concern to the problem domain I have in my hands but... yes... trigonometric operations a very poor.
* Nested loops where not analyzed.


In order to address the major issues, we evaluated these technologies:

* [FastUtil](https://github.com/vigna/fastutil) package which offers Colletions of primitive types. This is where Java code can beat C++ code due to the way C++ handles pointers as opposed to the way Java handles arrays.
* [JAL](http://vigna.di.unimi.it/jal/docs/) (broken link) package which mimics most of C++ STL functionality, providing fast algorithms on arrays of primitive types, which perform much faster than arrays of Objects.
* [Colt](http://dsd.lbl.gov/%7Ehoschek/colt/) package which is an excellent class library used by CERN. In particular Colt has a modified versions of JAL in it.
* [Parallel Colt](https://github.com/rwl/ParallelColt) package is a re-implementation of Colt package and aims to take advantage of modern multi-core CPUs.


See also:

* [Java performance at Wikipedia](http://en.wikipedia.org/wiki/Java_performance) has an overview of the subject and some overall results.
* [Microbenchmarking C++, C#, and Java at DDJ](http://www.ddj.com/java/184401976?pgno=1) has a general test suite and serious evaluation of results.
* [Fixed, Floating, and Exact Computation with Java's BigDecimal](http://www.ddj.com/java/184405721?pgno=1) presents situations when BigDecimal could be used instead of double.
* [Java Performance](http://www.javaolympus.com/J2SE/JavaPerformance/JavaPerformance.jsp) (broken link) has a good starting point on several subjects related to Java performance.
* [Mustang's HotSpot Client gets 58% faster!](http://weblogs.java.net/blog/opinali/archive/2005/11/mustangs_hotspo_1.html) (broken link) suggests that Java performance is getting better and better.
* [How Java's Floating-Point Hurts Everyone Everywhere](http://www.cs.berkeley.edu/%7Ewkahan/JAVAhurt.pdf) is an excellent discussion about accuracy of floating point operations in Java
* [Java and Numerical Computing](http://www.javagrande.org/leapforward/cacm-ron.pdf) An overview on the major difficulties for numerical computing with Java including a list of do's and dont's.

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
