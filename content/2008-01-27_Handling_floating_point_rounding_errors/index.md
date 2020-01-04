+++
title = "Handling of floating point rounding errors."
date = 2008-01-27T21:47:27Z
[taxonomies]
categories = ["articles"]
tags = ["java", "jquantlib"]
+++
_In this article we demonstrate that a very elementary mathematical statement can raise big concerns for numerical applications._

## Reasoning

Try this simple yet tricky code:

```java
System.out.println(0.1 + 0.1 + 0.1);
```

Oh well... the result is: 0.30000000000000004

What??? So... 0.1 + 0.1 + 0.1 is not 0.3 ???

## Problem

This problem arises the way floating point numbers are represented in the processor and how they participate in floating point calculations.

Notice that the comparison ``if (x==0.3)`` jumped to the else branch. After the sum, the result was ``0.30000000000000004`` and not ``0.3`` as we expected. The behaviour of the if statement is correct. In fact, the error is located between the keyboard and the chair: you simply cannot do such comparison!


You have to:

* remember that floating point errors may happen;
* evaluate the epsilon associated to the operations you previously done;
* you have to consider a certain range in your comparisions

like this:

```java
epsilon = blah blah blah; // calculate epsilon somehow
if ((x>=0.3-epsilon) && (x<=0.3+epsilon)) ...
```

## Solution

[JQuantLib](http://jquantlib.org) takes the same approach as [QuantLib](http://quantlib.org). It calculates epsilon after a sequence of mathematical operations which gives us the order of magnitude of the error.

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
