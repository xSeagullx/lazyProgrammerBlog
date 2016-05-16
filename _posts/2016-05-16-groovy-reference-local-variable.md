---
layout:     post
title:      "Groovy Closures and local variables"
date:       2016-05-16 03:00:00
summary:    "As you may know, in Java 8 lambda can access outer variables. These variables have to be 'effectively final',
            and you can't reassign them from lambda. Groovy seems free from that limitation and here I'll show - how it's achieved."
categories: groovy
draft: true
---

We were talking about Java lambdas and Groovy closures, when my colleague asked me an interesting question.
_"How Groovy handles local variables referenced in closures?"_
The thing is, that Java lambdas and anonymous classes require local variables to be "effectively final",
even if from Java 8 you can omit `final` keyword.
Being final means you can read variable, but can not reassign it (you can modify value behind that final reference, if it's mutable).
While Groovy closures can easily update these variables. I've investigated bytecode, to grasp what exactly happens there.

To begin with, let us set the ground with two examples. Both pieces of code reside in some method, and `sum` is a local variable.

Groovy:

```
int sum = 0
[1, 2, 3].each { sum += it }
println(sum) // prints 6
```

Java 8:

```
int[] sum = { 0 };
Arrays.asList(1, 2, 3).forEach(it -> sum[0] += it);
System.out.println(sum[0]); // prints 6
```

You see, that Groovy can directly reassign sum variable, while in Java code I have to use a common trick,
transforming local variable to a one-element array.
Any variable accessible from lambda shall be "effectively final", i.e. you can omit `final` keyword, but it will
still behave as a final one. This is the limitation coming from the way Java passes arguments around.
Java uses call-by-value, which means, that every time variable is passed around, a copy of it's value will be made available for receiver.

If you are familiar with C++ world, you may know the difference between call-by-value and call-by-reference. Former is creating a copy of variable, while passing it around callsite. Latter is actually passing 'location of variable in memory', allowing it to be rewritten, and variable - reassigned.
In case of Java, everything is passed by value, thus producing limitation about 
Passing immutable pointer of Reference object, you can still modify value held by that Reference. Moreover, that Reference is kept in Closure object, making it accessible and non garbage-collectable even when we left callsite, allowing to return closure with upvalue from methods.
{: .spoiler data-title="More about call-by-value and call-by-reference…" }

In first example copy of pointer to array will be passed to lambda. Both lambda and callsite can access and modify memory referenced by that pointer.
Adding indirection in this manner overcomes call-by-value limitation and is one of the simplest ways to affect outer scope variable's value from lambda.
Actually, same trick is done in Groovy, but it's hidden from user by pretty syntax.

Let's look what happens on bytecode level:

First, look at local variables: `LOCALVARIABLE sum Lgroovy/lang/Reference; L2 L5 2`. As we see, `sum` is not an `int` but `groovy.lang.Reference`.

At variable declaration we find, how variable `sum` is created. I've commented bytecode

    ICONST_0 // load initial variable 0
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer; // box it
    NEW groovy/lang/Reference // Create Reference instance
    DUP_X1
    SWAP
    INVOKESPECIAL groovy/lang/Reference.<init> (Ljava/lang/Object;)V // And init it with boxed value
    ASTORE 2 // Save Reference as a local variable.

We load constant, box it with Integer and then wrap it with container object of class `groovy.lang.Reference`. This container is then stored in local variable `2`.
Later, this container object is used exactly same way as one-element array in Java equivalent. And here is the bytecode for callsite place i.e. `println(sum)`.

    ALOAD 2 // Load Reference
    INVOKEVIRTUAL groovy/lang/Reference.get ()Ljava/lang/Object; // Get inner value.
    CHECKCAST java/lang/Integer
    INVOKEINTERFACE org/codehaus/groovy/runtime/callsite/CallSite.callCurrent (Lgroovy/lang/GroovyObject;Ljava/lang/Object;)Ljava/lang/Object; // Call println, passing unwrapped value.

We load Reference variable and call `get` method to obtain it's real value.

So, that's all, underneath we are working with familiar concepts. Here is an approximated Java version of what happens in Groovy code.

```
Reference sum = new groovy.lang.Reference(new Integer(0));
Arrays.asList(1, 2, 3).forEach(it -> sum.set(sum.get() + it)); // Note, that array is created differently in Groovy!
System.out.println(sum.get());
```

## Other observations

Before we finish, some interesting facts, I've got from experiments with bytecode.

1. Only variables accessed in closures are converted to `Reference`. Other local variables declared in same scope are left untouched.
2. Fields are accessed through the reference to outer object.
3. Reference for primitives require boxing. It should make one-element array a bit more performant solution.
4. Groovy (at least in 2.3.3) wraps variable in Reference even if it's used only for read, and not modified in Closure.
   I believe, that in read-only case it is not necessary, but it seems to be an implementation details.

<hr>

I was using Groovy Console's "Show AST Tree" function to see callsite's bytecode, and `javap -c` for inspection of closures.
