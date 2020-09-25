Commenting Code
===============

I think the best way of doing something is based on the understanding of
objectives. It keeps you on the track, gets you signals when you deviate
off course, helps you to get back to the road.

Sometimes we do things mechanically, just following traditions, because
this is how things work here.

And source code commenting is a good example of that. Everyone knows the
value of the documentation. But why is so valuable? What positive impact
does it bring to us? What are the drawbacks?

Please note that I'm talking about the code only not API, integrations
and other inter-system communication stuff.

Example
-------

I often come across the javadoc comments like that:

```java
/**
 * This method is used to find the latest price of the product for a specific store.
 *
 * @param product the product we are looking price for
 * @param store the store where the price is applied to
 * @return the price of the product
 */
public Price getPrice(Product product, Store store)
```

There might be different variations of the description:

* This method will return ...
* This method is responsible for ...
* This method returns ...

What's wrong
------------

The purpose of the documentation is not the documentation itself but
transfering knowledge (to other developers, to future you) in order to

* ease code maintenance
* help to find bugs faster
* get a deeper understanding of the contexts the code is used in

The problem with the documentation above is that it does not help to
reach these goals at all. It provides no extra details we can't get
directly from the method signature.

There is no reason for this comment to exist!

Well, we can make up some, e.g.:

* if your organization requires code comments
* if you want to generate documentation but have an acute phobia of
  empty spaces
* if you use code comments during algorithm implementation

But these problems do not relate to the source code itself and should be
resolved on another level. E.g. for the last one, just remove all the
comments after the algorithm is implemented. Extract and name things
instead of commenting on a code smell.

How to Fix
----------

The best documentation is no documentation if possible. No, really. If
your code is self-explanatory enough if you can get all reasonable
assumptions directly from the code without any extra effort, then any
documentation only makes things worse:

* It clutters space distracting developers from really important things
* It attracts attention wasting developers time
* It should be kept up-to-date to prevent confusing developers

It's all about developers' productivity and source code coherence.

So my advice is to put comments only if you want to
* explain the purpose of the code (not _what_ it does but _why_)
* share the examples of code usages
* describe the context the code is used in

But even before doing it, check if you can do it using code

* Name a function and parameters according to their function
* Name an exception properly to reflect the edge case
* Write a test to demonstrate how the code is used
* Use type system to restrict the contexts where code could be used

In all other cases the code will look better if it's like:

```java
public Price getPrice(Product product, Store store)
```
