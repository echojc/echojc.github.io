---
title: closures vs. structs in go
date: 2016-10-25
---

Today, I was reminded about this classic story about [closures vs.
objects](http://people.csail.mit.edu/gregs/ll1-discuss-archive-html/msg03277.html):

> The venerable master Qc Na was walking with his student, Anton. Hoping to
> prompt the master into a discussion, Anton said "Master, I have heard that
> objects are a very good thing - is this true?" Qc Na looked pityingly at his
> student and replied, "Foolish pupil - objects are merely a poor man's
> closures."
>
> Chastised, Anton took his leave from his master and returned to his cell,
> intent on studying closures. He carefully read the entire "Lambda: The
> Ultimate..." series of papers and its cousins, and implemented a small Scheme
> interpreter with a closure-based object system. He learned much, and looked
> forward to informing his master of his progress.
>
> On his next walk with Qc Na, Anton attempted to impress his master by saying
> "Master, I have diligently studied the matter, and now understand that objects
> are truly a poor man's closures." Qc Na responded by hitting Anton with his
> stick, saying "When will you learn? Closures are a poor man's object." At that
> moment, Anton became enlightened.

The last couple of years for me was a deep dive into a very Haskell-influenced
style of Scala and JavaScript programming. Between trying to grok
[typelevel.scala](http://typelevel.org/projects/) projects and
[advanced](http://haskell-servant.readthedocs.io/en/stable/tutorial/ApiType.html)
[dependent-typing](https://www.youtube.com/watch?v=fVBck2Zngjo) concepts, my
thinking naturally became biased towards modelling problems in terms of types
and functions instead of objects.

(To give a general idea of how deep down the rabbit hole I ended up, I wrote
both [type-level arithmetic
operators](https://github.com/echojc/peano-arithmetic/blob/master/src/main/scala/Main.scala)
(including a primality tester!) and [type-level SK
combinators](https://gist.github.com/echojc/ef509f407850265a2201) in Scala.)

Recently, I've turned my attention to Go. It's been a very refreshing
experience, having the procedural-functional pendulum swing back the other way.
The specific problem that brought this story back to me was in trying to
abstract HTTP handlers in Go.

The standard library provides this interface to handle requests:

```go
type HandlerFunc func(ResponseWriter, *Request)
```

and you'd use it like this, for example:

```go
func MyHandler(w http.ResponseWriter, r *http.Request) {
  http.NotFound(w, r)
}

http.HandleFunc("/", MyHandler)
http.ListenAndServe(":9000", nil)
```

This API is very barebones, but there's nothing wrong with it. I really enjoy
how straightforward and pragmatic standard Go APIs tend to be.

One problem that creeps in, though, is when your handler needs other
dependencies in order to do whatever it needs to do. For me, that was a database
connection. Go provides generic database functionality via the `sql` package,
which exposes the `*sql.DB` type as a database handle of sorts. So how do I get
an instance of this struct into my handler?

A basic solution is to simply share what is essentially a global `*sql.DB`
instance, for instance:

```go
func main() {
  db := sql.Open("postgres", "postgres://...")

  myHandler := func(w http.ResponseWriter, r *http.Request) {
    // use `db`
  }

  http.HandleFunc("/", myHandler)
  http.ListenAndServe(":9000", nil)
}
```

This might be fine for small projects, but there are some common issues with
this implementation: you can't replace the `db` instance with something else for
testing purposes (whether it's a connection to a different database or a mock),
and everything needs to be in the same scope, which doesn't help with
modularization.

Knowing these problems, my immediate thought was to use partially applied
functions. In ES6, it'd be something like this:

```javascript
handler = (db) => (writer, request) => {
  // use `db`
};
```

This way, the function would still close over the connection, but I'm now free
to dictate exactly which instance of `db` the handler should use in each
circumstance.

I did in fact end up writing the equivalent in Go, which looked something
like this:

```go
func CreateHandler(db *sql.DB) func(http.ResponseWriter, *http.Request) {
  return func(w http.ResponseWriter, r *http.Request) {
    // use `db`
  }
}
```

and to test:

```go
func TestFoo() {
  db := getTestConnection()
  handler := CreateHandler(db)

  req := httptest.NewRequest(...)
  res := httptest.NewRecorder()
  handler(res, req)

  // check `res`
}
```

But sitting back and looking at my handiwork, I couldn't shake that niggly
feeling that this wasn't quite right - it felt clumsy and not very idiomatic.

Eventually, I stumbled upon a different way to approach the problem, and it
(surprise!) involved using `struct`s. It wasn't complicated, and, to be honest,
would very likely have been the first thing to spring to mind for anyone who
_hadn't_ been so heavily invested in modelling problems with functions in the
first place. Consider this:

```go
type MyHandler struct {
  db *sql.DB
}

func (h *MyHandler) Handle(w http.ResponseWriter, r *http.Request) {
  // use `h.db`
}
```

By using a method on a struct instead of a partially applied function, the
closing over of the `db` variable is now pushed into the instance of the struct
instead of an instance of a function. This is much more idiomatic Go. And if
there's any doubt that these two implementations are semantically equivalent,
and you can see that it really is the case by comparing the test for this
version with the previous test:

```go
func TestBar() {
  db := getTestConnection()
  handler := MyHandler{db}

  req := httptest.NewRequest(...)
  res := httptest.NewRecorder()
  handler.Handle(res, req)

  // check `res`
}
```

This shows precisely what the opening story tries to convey: that objects and
closures are different ways to express the same thing. Some languages favour one
idiom over the other, but, at the end of the day, the result is the same.

This was a good reminder to me that using the right tool for the right job not
only applies to languages and tooling, but to concepts and abstractions as well.
