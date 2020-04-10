Boxed Values
============

Despite the claim that Java is an object-oriented language, it contains
not only object data types. There are eight types called **primitives**
such as boolean, int and char. Java does not treat them like objects and
handles them in a specific way. As an example, they do not have methods
but they have predefined operators that are applied for them.

Since they are not objects, they are not inherited from `Object` class
and they can't be used, let's say, as the elements of a `List` or as the
keys or the values of a `Map` object. To cover these cases Java
introduces **wrapper types**: objects that wrap over some primitive
value. Now we can just wrap a primitive into a wrapper and use it as an
ordinary object. This is called **boxing**. Extracting the value from
the wrapper, on the other hand, is called **unboxing**.

Boxing and unboxing might look a bit annoying and Java 5 helped us a lot
by bringing **autoboxing** to the table: now we don't need to convert
primitives to wrappers and back. Java compiler can treat primitives as
wrappers and vice verse.

Example
-------

The code I'd like to talk about is mainly about defining the class
fields and methods parameters. For example:

```java
private Boolean active = Boolean.TRUE;

public Boolean getActive() {
    return active;
}

public void setActive(Boolean active) {
    this.active = active;
}
```

or

```java
public void retryOperation(Operation operation, Integer retryCount) {
    boolean done = false;
    int attempts = 0;

    while (!done && attempts <= retryCount) {
        done = operation.process();
        attempts++;
    }
}
```

`Boolean` is a very specific case, and I'm going to talk about it in a
separate article discussing boolean blindness, for example, or apache
`BooleanUtils` using. In this article let's focus on primitive vs
wrapper arguments only.

What's Wrong
------------

First, let's see what's happening here:

```java
if (operation.getActive()) {
    retryOperation(operation, 3);
}
```

Compiler translates this code into:

```java
if (operation.getActive().booleanValue()) {
    retryOperation(operation, Integer.valueOf(3));
}
```

So we know for sure that:

* wrapper values are used
* (un)boxing operations are called during runtime
* some IDEs complain about unboxing without [null check](NULLCHECK.md)

Wrapper values used in place of primitives consume more memory. Let's
see how much. It differs from JVM to JVM, so for simplicity let's take
64-bit JDK. Object there has 12-byte header, and integer primitive value
fits right away into 16 pad (it depends on the memory alignment, let's
pretend that it's 16 bytes). So the total space used by `Integer` class
is 16 bytes. Same for `Boolean`. So you see it's 4 times more than a
primitive value.

The unboxing operation might be really fast if it's optimized the way we
can get the wrapped value directly. I don't know if JIT does it. But
even if it does, it still has to make sure that the wrapper is not null.
So even in this case, we can't avoid the unboxing operation. For boxing
it's even worse: we need to create a new object or get the existing one
from the cache. Obviously, this is not as cheap as just getting the
primitive value.

Now about IDE complaints. There are several ways to suppress the warning
about null-check. We can put `@SuppressWarnings` annotation which
pollutes the code a bit and potentially could lead to null pointer
exception. We can check for null which slows down the code a bit and
makes it less readable. We can turn this warning off and lose some
really critical notes in other places. We can just ignore it and have
this annoying yellow code causing eye bleeding.

One more argument against wrappers: just look at this ugly `getActive`
method. It'd be much better to have a nice `isActive` getter instead
that is so suitable for if-conditions.

So to sum it up:

* primitives consume less memory
* primitives use fewer CPU ticks
* no unboxing operations in the condition expressions
* boolean getters look more natural
* no more NPE ever

How to Fix
----------

And again it depends. Wrapper classes are a great tool and we must use
it responsibly. I can suggest this rule of thumb:

_If your variable could have null values, go with wrappers. Otherwise,
prefer primitives over wrappers._

Of course, it depends. If you use a bunch of boxing operations, for
example, your variable is mostly used as the element of the `List`
object, prefer wrapper. But often it's not justified.

Think about semantics, not technical details. Even if a column in the
database is nullable but we handle it differently whether it's true or
not, use primitive with default false for null values. And the code like

```java
if (BooleanUtils.isTrue(operation.getActive())) {
    // ...
}
```

will turn into beautiful

```java
if (operation.isActive()) {
    // ...
}
```
