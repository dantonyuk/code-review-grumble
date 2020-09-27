Know All the Ways
=================

OK. Maybe it's not a good title for what I want to share. Anyway, I need
to name it somehow, so let that be it.

Programming languages and natural languages are alike in some respect.
Both are used to express intentions, and in both of them, there is more
than one way to achieve it. Which one to pick is a hard problem that
depends on various factors, for example for programming languages:
maintainability, performance, language idioms, conventions, team
expertise, etc. However, developers often go down some road just because
they are not aware of better options, or because they do not care much.
That results in slow, hard-to-read code.

For example, in Java, we can handle collections using old good loops and
if-statements, on the one hand, or new Stream API, on the other.
Apparently, this subject is too broad, I'm going to discuss it in
further grumbles. I'm going to cover different areas in this one.

These examples are supposed to demonstrate not so much the ways we can
get our job done but how to pick the one that works best.

Example
-------

As usual, the examples I provide are collected from real code
reviews, just slightly modified. I'm going to give you a bunch of them,
but what I aim is not to force you to remember them as is, but instead
developing a nose for such kind of smells. Watch.

```java
LocalDateTime getFiredTime(Event event) {
    LocalDateTime fired = eventFiredMap.get(event);
    if (fired == null) {
        fired = LocalDateTime.now();
        eventFiredMap.put(event, fired);
    }
    return fired;
}
```

This piece of code checks if `eventFiredMap` holds the date when the
event was fired and returns it, otherwise, it stores the current time
and returns it.

```java
void addGroup(Group group, Long userId) {
    Optional<User> userOpt = findUser(userId);
    if (userOpt.isPresent()) {
        User user = userOpt.get();
        group.addUser(user);
    }
}
```

Using `Optional` in the code conveys the fact that `findUser` can return
either a user instance or nothing (no result). So after getting the
result, it checks if a user was returned, and add it to the specified
group if it was.

```java
boolean hasEmptyComponents(Collection<Event> events) {
    long emptyCount = events.stream()
        .map(event -> event.getTarget())
        .distinct()
        .filter(component -> component.isEmpty())
        .count();
    return emptyCount > 0;
}
```

This method based on a stream of events finds all the target components
and calculates how many of them are empty. If the amount of empty
components is greater than zero, it returns true. Otherwise false.

```java
String nameRepr(Collection<String> names) {
    return names.stream().collect(Collectors.joining(", "));
}
```

Frequently requested operation to create a comma-separated
representation of a collection of strings.

```java
void setParameters(Query query, Map<String, String> parameterMap) {
    parameterMap.entrySet().stream()
        .forEach(e -> query.setParameter(e.getKey(), e.getValue()));
}
```

This code just sets parameters of a `query`. No matter what exactly type
of queries it is: database or HTTP or something else. What matters is
that this query interface has `setParameter` method.


What's Wrong
------------

First of all, it must be said that all those methods work as expected.
There are no logical errors there. They do the stuff they intended to
do, they return proper values. However, each of those examples sends a
signal that something is wrong with it. Let's try to get those signals
classified in some way.

### getFiredTime

Looking at the code with a map we can notice that there are two calls to
the map that search internally for the proper place to get/put a value.
Knowing that it's a common pattern, we can expect that the `Map`
developers could think about it and maybe they already provided some
solution for that. Note, that it's not the code that is bad, but it's
about signals that it sends us to look for a better solution if it
exists. It might not. But as responsible developers, we need to find out
it first.

### addGroup

We can find a similar situation in the `Optional` example. The `get`
operation is not safe, so we check it first by calling `isPresent`. And
this pattern should be so widely used that we have to check if there is
another way covering it.

### hasEmptyComponents

OK, we got all the empty components and checked if they exist by
comparing their count with zero. What signal should we catch here? Just
imagine how would you write this code without Stream API? It's hardly
going to be counting empty components. I bet you will return true right
after you met the first empty component in your loop. Something like:

```java
for (Event event: events) {
    if (event.getComponent().getTarget().isEmpty()) {
        return true;
    }
}
```

Why wouldn't we look if there is something existing that could help us?

### nameRepr

The code looks pretty good. Concise and expressive, right? But again,
what sets off alarm bells here is that we use a very common `String`
pattern that is being implemented using `Stream`s. It does not have to
be a problem but we need to check other ways.

### setParameters

This one is tough a bit. But if you feel that a `Map` should be able to
iterate over its entries, you're moving in the right direction.

How to Fix
----------

This grumble is not about "fixing" things but the ways we are looking
for improving them.

So the result will not help much. Anyway, below is the code, just to
demonstrate the difference.

### getFiredTime

```java
LocalDateTime getFiredTime(Event event) {
    return eventFiredMap.computeIfAbsent(event, k -> LocalDateTime.now());
}
```

### addGroup

```java
void addGroup(Group group, Long userId) {
    findUser(userId).ifPresent(group::addUser);
}
```

### hasEmptyComponents

```java
boolean hasEmptyComponents(Collection<Event> events) {
    return events.stream()
        .anyMatch(event -> event.getTarget().isEmpty());
}
```

### nameRepr

```java
String nameRepr(Collection<String> names) {
    return String.join(", ", names);
}
```

### setParameters

```java
void setParameters(Query query, Map<String, String> parameterMap) {
    parameterMap.forEach(query::setParameter);
}
```
