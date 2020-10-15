Not Magic Values
================

Developers are familiar with magic numbers and magic strings. There are
a lot of reasons to avoid this anti-pattern:

* The values are not self-documented. They say nothing about the purpose
  and the context they are used in.
* If we have magic value used in multiple places, we have to change all
  of them which is tedious and error-prone.
* There are no compile checks, so a developer can introduce a bug
  mistyping the value.
* Two similar numbers used for the different purposes could easily
  confuse, e.g. when we search for Atlanta area code 404 and find a lot
  of "HTTP 404 Not Found" values, or when we search and replace the
  value belonging to one abstraction and accidentally change the value
  belonging to other abstraction.

As you can see the problem arises when we have a value that

* has no obvious purpose requiring an explanation
* is expected to be changed
* is supposed to be used for searches

Introducing a well-named constant helps to solve this problem. But we
should not put the cart before the horse and extract a constant
recklessly every time we see a string value in the code. This step
should be pertinent. And to make sure it is, we need to prove that we
have enough reasons. Otherwise, we can make things worse.

Examples
--------

Some constants I came across during code review:

```java
String COMMA_DELIMITER_SPACE = ", ";
String COLON = ":";
String HYPHEN_SYMBOL = "-";
String NETBOOK = "NETBOOK";
String TABLET = "TABLET";
String ADD = "add";
String DELETE = "delete";
String HTTP_URL_HEADER = "http://";
String SLASH_SEPARATOR = "/";
String QUERY_STRING_SEPARATOR = "?";
String TRUE_STR = "true";
```

Zillions actually. Or look at the code below.

```java
String url = new StringBuilder()
  .append(HTTPS_URL_HEADER)
  .append(hostName)
  .append(SLASH_SEPARATOR)
  .append(usersApiPath)
  .append(QUERY_STRING_DELIMITER)
  .append(ACTIVE)
  .append(EQUAL)
  .append(TRUE_STR)
  .append(AND)
  .append(COUNTRY)
  .append(EQUAL)
  .append(country)
  .toString();
```

What's Wrong
------------

Any named thing is the code should represent an abstraction or be a part
of some abstraction. Provided constants do not bring any extra
information compared to their values. It does really make no sense to
introduce `TRUE_STR` constant. It says nothing about the context or the
purposes this constant will be used in. It literally means that any
place we use `"true"` value could be changed to `TYPE_STR` and vice
versa.

No abstraction, no self-documenting, the only reason we can try to make
it fit is that we can put a typo somewhere. I don't think this reason is
good enough to make the code uglier.

Second, the code is really hard to read. We have to go through all these
appends to understand what exactly we are building there. Too many
efforts that people can decide to not pursue.

How to Fix
----------

Treat things based on what they're for not what the values they hold. If
you are really into constants, name them properly `ACTIVE_PARAM` not
`ACTIVE` and so on. I do not think it's a good way to spend your time,
but you're the boss.

What I would suggest is to concentrate on the clean code and easy
maintenance than following "best practices" without understanding the
reasons:

* Remove all these constants and start using them as-is.
* Use formatting tools to build strings:

```java
String url = String.format(
    "https://%s/%s?active=true&country=%s",
    hostname, usersApiPath, country);
```

Much easier to read and maintain.
