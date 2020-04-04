Code Formatting
===============

There is no need to justify code formatting. No doubt, every developer
understands the benefits of following the code style guidelines. We can
argue about what end should we break our eggs at. But to break them or
not to break, that's not even a question.

Personally, I don't care what exactly guidelines to follow. Go along to
get along. Work out the conventions and follow them. That's easy.

Also, I fully agree that code formatting issues should not be mentioned
as a part of code review but rather be a matter of automatic tools as
linters or formatters. Unfortunately, in the real world, automatic
formatting is not always possible.

Example
-------

This is just an example of a Spring MVC controller method:

```java
public List<Book> getBooks(@RequestParam(value = "authorId") long
    authorId, @RequestParam(value = "inStock", required = false,
    defaultValue = "true") boolean inStock) {

    if (!authorService.exists(authorId) || (inStock && !authorService
        .hasBooksInStock(authorId)) {

        return Collections.emptyList();
    }

    return bookService.findBooks(authorService.findAuthor(authorId,
        STATE_ACTIVE));
}
```

Another example is for Spring MVC Security:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers("/", "/home").permitAll()
        .anyRequest().authenticated().and().formLogin().loginPage(
        "/login").permitAll().and().logout().permitAll();
}
```

The most often I saw the code like that in Stream API calls. E.g.:

```java
authorService.listAuthors().stream().filter(author -> author.isActive())
    .flatMap(author -> bookService.findBooks(author).stream()).filter(book ->
    book.isInStock()).map(book -> book.getTitle()).collect(Collectors.toList());
```

What's Wrong
------------

I see one crucial issue with the code above: a lack of readability. As
developers, we read code many times more than we write it. And the most
important thing is that we read it to change it: add this piece, move
that piece. I won't go into the details. I believe we share the value
of readability.

Let me show you what I mean based on the last example:

<pre>
authorService.<i>listAuthors()</i>.stream().<b>filter</b>(author -> author.<i>isActive()</i>)
    .<b>flatMap</b>(author -> bookService.<i>findBooks(author)</i>.stream()).<b>filter</b>(book ->
    <i>book.isInStock()</i>).<b>map</b>(book -> book.<i>getTitle()</i>)<b>collect</b>(Collectors.toList());
</pre>

This is how we read it:

* For all authors
* filter only active ones
* map to their books (and flatten the result)
* filter only available books
* map to their title
* collect everything to the list

To read it we have to go along the lines and identify semantically
important tokens. The code isn't helping us to find out what it does.

How to Fix
----------

It's hard to formulate the rule, that's why it's close to impossible
to automatize it.

Let me explain to you one more time what the last example does:

_For all authors filter only active ones, then map to their books (and
flatten the result), filter only available books, map to their title,
and collect everything to the list._

To me reading the list of bullet points is much easier than this
sentence. This is what your code must be to be readable: it must be
poetry, not prose. The verse where the pivots are placed at the
beginning of the lines:

```java
authorService.listAuthors().stream()
    .filter(author -> author.isActive())
    .flatMap(author -> bookService.findBooks(author).stream())
    .filter(book -> book.isInStock())
    .map(book -> book.getTitle())
    .collect(Collectors.toList());
```

Now it's much easier to read, right? Use it even you use only one method
in your stream pipeline, like `map` or `filter`. Even then, it should be
three lines:

```java
authorService.listAuthors().stream()
    .filter(author -> author.isActive())
    .collect(Collectors.toList());
```

<a name="b1"></a>
It's easy to set it like that. Just tell your IDE to keep line breaks
when reformatting [<sup>1</sup>](#f1).

OK. In this example, we can handle one-level indentation to get it. But
there are more sophisticated examples in Fluent API. For example, Spring
Security code is a little bit harder. This is how it looks like in
Spring documentation:

```java
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
            .antMatchers("/", "/home").permitAll()
            .anyRequest().authenticated()
            .and()
        .formLogin()
            .loginPage("/login")
            .permitAll()
            .and()
        .logout()
            .permitAll();
}
```

It's much easier to read it because we can identify three sections here,
and deal with each section separately. Nice.

<a name="b2"></a>
Unfortunately, when we reformat the code automatically, it will break
it. So we need to disable the formatting for certain pieces of code.
Luckily, mainstream IDEs support special `formatter` tag
[<sup>2</sup>](#f2) :

```java
protected void configure(HttpSecurity http) throws Exception {
    // @formatter:off
    http
        .authorizeRequests()
            .antMatchers("/", "/home").permitAll()
            .anyRequest().authenticated()
            .and()
        .formLogin()
            .loginPage("/login")
            .permitAll()
            .and()
        .logout()
            .permitAll();
    // @formatter:on
}
```

Just make sure that your IDE has this option turned on.

The last (the first) example. The rule is the same: _keep semantic units
on a separate lines, do not mix them_. Note that semantic unit is not
necessarily one method call. It also might be two calls in a row (that
is why we put "technical" but not "semantical" call of `stream()` method
to the very end of the line. It's just not so important to get the code.

```java
certificateService.listCertificates().stream()
    .filter(cert -> cert.isVerified())
    .findFirst().orElse(null);
```

We can put `findFirst` and `orElse` on the same line: they belong to
the same semantic unit. It's not "get me the first result, and **then**
do something with it", no. It ain't nothing but "get me the result
**or** null if it's absent".

Fine. The example itself how I see it:

```java
public List<Book> getBooks(
    @RequestParam(value = "authorId") long authorId,
    @RequestParam(value = "inStock", required = false, defaultValue = "true") boolean inStock) {

    if (!authorService.exists(authorId) ||
        (inStock && !authorService.hasBooksInStock(authorId)) {

        return Collections.emptyList();
    }

    return bookService.findBooks(
        authorService.findAuthor(authorId, STATE_ACTIVE));
}
```

Close, but not a cigar. I mean it's really fine. I would approve the PR
with this code in it. But we can do better. Let's talk about someday.

---

<a name="f1"/>[<sup>1</sup>](#b1) Keep line breaks when reformatting:
[Eclipse](https://stackoverflow.com/a/19697144/8081752),
[Intellij IDEA](https://stackoverflow.com/a/7162569/8081752) (by default
it's turned on so the question is about turning it off).

<a name="f2"/>[<sup>2</sup>](#b2) Turn on formatter tags:
[Eclipse](https://stackoverflow.com/a/5115143/8081752),
[Intellij IDEA](https://stackoverflow.com/a/19492318/8081752).
