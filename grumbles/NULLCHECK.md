Checking for null
=================

Java 7 brought us [Objects](https://docs.oracle.com/javase/8/docs/api/java/util/Objects.html)
utility class with a handful of useful null-tolerant methods such as
`hash`, `equals`, `compare` etc. Java 8 took it one step further
introducing two new methods
[`isNull`](https://docs.oracle.com/javase/8/docs/api/java/util/Objects.html#isNull-java.lang.Object-)
and [`nonNull`](https://docs.oracle.com/javase/8/docs/api/java/util/Objects.html#nonNull-java.lang.Object-).

Some curious developers started using them, straight away. Let's go
through some examples to figure out when and where these methods could
be used.

Example
-------

There is plenty of code using `isNull` and `nonNull` methods as boolean
expressions in if-statement condition, ternary operator's condition, or
as a definition for a logical variable.

```java
if (Objects.nonNull(order) && Objects.nonNull(order.getPromoCode())) {
    // do something with a promotion
}
```

or

```java
String customerName = Object.isNull(order) || Object.isNull(order.getCustomer())
    ? "" : object.getCustomer().getName();
```

What's Wrong
------------

Frankly, there's nothing wrong with it. The code is pretty much the same.
There might be some minor issues in some IDE that may show "possible null
object dereferencing" warning but that's it.

My concern here is that we misuse it. `isNull` and `nonNull` is a good
tool, but `==` and `!=` seem much better for checking nulls. Why? The
honest answer is "why not?". We already have these operators and
everyone is used to them. So when we're bumping into `isNull`/`nonNull`
the question "why" naturally comes to mind. I consider it as an obvious
violation of the principle of least astonishment. One of the fundamental
principles of good design and clean code.

If you're not convinced, I have another argument. Just read the
documentation. It makes it quite clear.

> **API Note:**
> This method exists to be used as a Predicate, filter(Objects::isNull)

Same for `nonNull`. This is the only reason why we have these methods.
To be used as method references in `Predicate`. Not as a replacement for
relational operators.

How to Fix
----------

It's pretty easy.

Never call `isNull` or `nonNull`. Use operators instead:

```java
if (order != null && order.getPromoCode() != null) {
    // do something with a promotion
}
```

or

```java
String customerName = order == null || order.getCustomer() == null
    ? "" : object.getCustomer().getName();
```
