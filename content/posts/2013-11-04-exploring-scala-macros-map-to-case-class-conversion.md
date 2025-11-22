---
title: "exploring scala macros: map to case class conversion"
date: 2013-11-04
tags: [scala, macros]
---

Recently I had a go at writing some [Scala
macros](http://docs.scala-lang.org/overviews/macros/overview.html). Macros
allow you to write code that generates code, at compile time. They can [bring
about benefits](http://docs.scala-lang.org/overviews/macros/usecases.html) such
as automatic code generation via implicits, static type safety within strings
(when interpolating), and even allow for the creation very fluent DSL
interfaces.

My best understanding of Scala macros was the code generation aspect of it, so
I decided to tackle a problem that has probably plagued every budding developer
who's tried to roll their own ORM in a statically typed language: persisting a
case class to the database and reading it back without using reflection.

The crux of the problem is always the conversion between a type-safe case class
and the database layer. In more mainstream languages like Java, there is simply
no way to automatically call some function for each field based on its type
without using reflection. With macros, however, the code to do this can be
generated at compile time.

## the problem

Let's reduce the problem to a very specific one: taking any
arbitrary case class and producing converter functions to and from a
`Map[String, Any]` where the keys are the names of the case class's constructor
parameters pointing to their respective values.

[Note: many of the problems I faced while writing this macro were solved by
looking at this [implementation on
StackOverflow](http://stackoverflow.com/questions/19544756/scala-macros-accessing-members-with-quasiquotes),
hence the similarity.]

To take advantage of [implicit
macros](http://docs.scala-lang.org/overviews/macros/implicits.html) (we'll get
back to them later), we'll use a type class to provide the conversion:

```scala
trait Mappable[T] {
  def toMap(t: T): Map[String, Any]
  def fromMap(map: Map[String, Any]): T
}
```

Any implementation of the `Mappable[T]` trait can now be used to convert a type `T`
to and from a map. For example, we can define one manually:

```scala
case class Person(name: String, age: Int)

val PersonMapper = new Mappable[Person] {
  def toMap(p: Person) = Map(
    "name" -> p.name,
    "age" -> p.age)
  def fromMap(map: Map[String, Any]) = Person(
    map("name").asInstanceOf[String]
    map("age").asInstanceOf[Int])
}
```

There's a big problem with defining the mapper explicitly though: any time the
case class changes the mapper must also be updated accordingly. Take, for
example, the case of adding a new parameter to the `Person` case class:

```scala
case class Person(name: String, age: Int, height: Double)

val PersonMapper = new Mappable[Person] {
  // toMap: compiles even though it's incorrect
  def toMap(p: Person) = Map(
    "name" -> p.name,
    "age" -> p.age)

  // fromMap: fails to compile (as it should)
  def fromMap(map: Map[String, Any]) = Person(
    map("name").asInstanceOf[String]
    map("age").asInstanceOf[Int])
}
```

When only the case class has changed, the compiler can catch the error in the
`fromMap` method because it's one parameter short, but the compiler can't catch
the semantic error in the `toMap` method missing the new `height` parameter.

## using macros

The reason for this is that explicitly defining the mapper leads to code that's
not very [DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself). It
introduces multiple points in the code that have to change in order for some
changes to be semantically correct. Ideally, the mapper should be able to
figure out what fields are needed by looking directly at the class it's defined
for rather than having each field explicitly listed in its methods.

It turns out that macros let you do this really easily. Let's start by defining
a barebones macro in the companion object of the `Mappable` trait:

[Note: you can clone [this template
repo](https://github.com/echojc/scala-macro-template) to follow along. With the
2.11.0-M5 compiler, macros must be compiled separately from the code that uses
them. With this template, the macro subproject can be used for this purpose.]

```scala
import scala.reflect.macros.Context
object Mappable {
  implicit def materializeMappable[T]: Mappable[T] =
    macro materializeMappableImpl[T]

  def materializeMappableImpl[T: c.WeakTypeTag](c: Context): c.Expr[Mappable[T]] = {
    import c.universe._
    val tpe = weakTypeOf[T]

    c.Expr[Mappable[T]] { q"""
      new Mappable[$tpe] {
        def toMap(t: $tpe) = ???
        def fromMap(map: Map[String, Any]) = ???
      }
    """ }
  }
```

Even to a seasoned Scala user, if you've never used macros before this probably
looks like gobbledegook! [Dependent
types](http://danielwestheide.com/blog/2013/02/13/the-neophytes-guide-to-scala-part-13-path-dependent-types.html),
[quasiquotes](http://docs.scala-lang.org/overviews/macros/quasiquotes.html),
even a [context bound](http://stackoverflow.com/a/4467012) thrown into the mix.
Behind all the flashiness, however, it's actually fairly straightforward. Let's
go through this one part at a time.

## implicit function to trigger macro

We start off with the implicit method that triggers the macro:

```scala
implicit def materializeMappable[T]: Mappable[T] =
  macro materializeMappableImpl[T]
```

It's easy to see that this method returns a `Mappable` corresponding to
whatever type is passed when the function is called. This method doesn't have
an implementation; the `macro` keyword instructs the compiler to expand the
corresponding macro implementation instead, in this case,
`materializeMappableImpl`.

The reason we make this method implicit is that this allows the compiler to
automatically create mappers for types as required (the aforementioned
[implicit macros](http://docs.scala-lang.org/overviews/macros/implicits.html)).
Without it, one would need to explicitly create a mapper before using it:

```scala
def personToMap(p: Person) = {
  val mapper = materializeMappable[Person]
  mapper.toMap(p)
}
```

By marking the method implicit, we give the compiler the opportunity to
automatically insert this method call whenever an implicit parameter of type
`Mapper[T]` is required. For example,

```scala
// the compiler will insert materializeMappable[T] as the implicit parameter
def mapify[T](t: T)(implicit mapper: Mappable[T]) =
  mapper.toMap(t)
```

We can even use context bounds to not explicitly specify the extra parameter:

```scala
def mapify[T: Mappable](t: T) =
  implicitly[Mappable[T]].toMap(t)
```

In this case, the mapper is implicitly inserted into the function by the
compiler. We don't have a reference to it, but it's there, so we use the
implicitly function to summon it from the nether world.

## macro boilerplate

Let's move on to the macro implementation. The structure of the macro function
looks at first sight to be some strange incantation:

```scala
def materializeMappableImpl[T: c.WeakTypeTag](c: Context): c.Expr[Mappable[T]] = {
  import c.universe._
  // _...
}
```

Again, however, it's actually fairly straightforward. Macros work with code, so
we manipulate it with abstract syntax trees. The Context variable contains
information the compiler would have pertaining to the current invocation of the
macro (such as call site, parameters, etc.). This is passed as a parameter to
the macro expansion. All the other information about the macro invocation are
then passed the same way the original function is written as a dependent type
of the current Context: parameters and return types as c.Exprs (essentially
typed ASTs [according to the
docs](http://www.scala-lang.org/files/archive/api/2.11.0-M5/#scala.reflect.api.Exprs%24Expr)),
and type parameters as c.WeakTypeTags (see [this
commit](https://github.com/scala/scala/commit/788478d3ab) for an explanation
about why it must be a WeakTypeTag and
[here](http://stackoverflow.com/a/12232195) for more information about TypeTags
in general).

Finally, we import everything inside the universe of the Context to bring all
the common utility functions into scope.

## macro implementation

Now we get into the nuts and bolts of the macro:

```scala
val tpe = weakTypeOf[T]

c.Expr[Mappable[T]] { q"""
  new Mappable[$tpe] {
    def toMap(t: $tpe) = ???
    def fromMap(map: Map[String, Any]) = ???
  }
""" }
```

To start things off, we first get the type of the case class we're creating a
mapper for out of the WeakTypeTag. This tpe variable can then be used directly
within quasiquotes.

[<s>Note: it looks like WeakTypeTags should also be directly usable within
quasiquotes since [they also have a Liftable
implementation](http://www.scala-lang.org/files/archive/api/2.11.0-M5/#scala.reflect.api.StandardLiftables)
but I couldn't get it to work. I didn't look too closely at it though.</s>
[densh has pointed out](#comment-1138314276) that you need a variable of type
`WeakTypeTag` and not a type of one for this to work.]

Now,
[quasiquotes](http://docs.scala-lang.org/overviews/macros/quasiquotes.html). I
found this part the most awesome part about writing Scala macros. They're
somewhat of a replacement for the [earlier `reify`/`splice`
style](http://docs.scala-lang.org/overviews/macros/overview.html#a_complete_example)
of writing macros. They work just like [interpolated
strings](http://docs.scala-lang.org/overviews/core/string-interpolation.html),
but instead of a string you write normal Scala code and instead of splicing
string versions of variables with `$variable`, you splice ASTs. The most obvious
distinction between the two is that `reify` returns an `Expr`, while quasiquotes
return an AST which must then be wrapped into an `Expr` explicitly.

With that understanding, the rest of this code snippet should be easy to
understand. We define an `Expr` of type `Mappable[T]` and use quasiquotes to create
the AST from normal code. Note the use of the `tpe` variable inside the
quasiquotes in place of `T`. We use `???` here because we've yet to discuss the
real implementation of the `Mappable` instance.

## getting fields

Our instance of `Mapper` needs to iterate over the fields of the case class it's
used for. We don't want _all_ fields though; just the ones used in the
constructor are all we want.

There are many ways we can get at that information. Methods have an
`isCaseAccessor` flag that signifies whether they are used to access the
parameters in the constructor. We can also look at the primary implementation
of the copy function. However, because we'll eventually need the exact order of
parameters in order to implement the `fromMap` method, we'll use the _primary
constructor_ to get the list of fields we need.

To do this, we'll inspect the `tpe` variable describing our case class to get a
list of all its _declarations_. [Note: `declarations` are members declared directly
in this class, while `members` include inherited ones.] One of these will be the
primary constructor, so we use a pattern match with a guard to get it out. Once
we have the constructor, we can extract the list of parameters in the order
that we need.

This can be translated directly into code:

```scala
val declarations = tpe.declarations
val ctor = declarations.collectFirst {
  case m: MethodSymbol if m.isPrimaryConstructor => m
}.get
val params = ctor.paramss.head
```

`paramss` looks like a typo, but in fact it's a list of lists (of parameters),
hence the double 's'. There's only ever one primary constructor, so in our case
we're fine taking the `head` of that list, but methods in general can be
overloaded to take different parameter lists which is why it's there.

## writing `toMap`

Now that we have the fields, let's write the `toMap` method. Let's refresh
ourselves with what this method should look like by taking a look at the manual
implementation from earlier:

```scala
def toMap(p: Person) = Map(
  "name" -> p.name,
  "age" -> p.age)
```

The implementation is just one statement! It's just a call to `Map.apply` with
"stuff" in it. Let's break down what that "stuff" includes:

1. the name of the field as a `String`
2. a call to the `->` method to create the tuple
3. a member access to the underlying field

What we need, then, is an AST that represents this. What better way to generate that AST than to use quasiquotes?

```scala
val toMapParams = fields.map { field =>
  val name = field.name
  val mapKey: String = name.decoded
  q"$mapKey -> t.$name"
}
```

That's all we need! The `mapKey` variable is annotated with its type `String` to
illustrate the fact that `String`s have a built-in `Liftable` implementation that
allows quasiquotes to convert it into the appropriate AST without us doing so
explicitly (the AST would be `Literal(Constant(mapKey))`).

There are probably two more things in here that stand out: what does it mean to
decode the name? And what's this t variable that hasn't been defined anywhere?
(Or has it…?)

[According to the
docs](http://www.scala-lang.org/files/archive/api/2.11.0-M5/index.html#scala.reflect.api.Names%24NameApi@decoded:String),
decoding the name "replaces all occurrences of `$op_names` in this name by
corresponding operator symbols". We want this because in the case a parameter
has a name like content-type, we want the map to have the key `content-type` and
not `content$minustype`.

The `t` variable is a bit more tricky. We must remember that all we're
constructing here is an AST. It is merely some small portion of code. With no
context, this `t` variable makes no sense, but if we put it in some context where
some variable `t` is defined, then it does make sense. If you look back at the
original definition of the `toMap` method we used in the macro, you'll see that
the name of the variable passed into the `toMap` method is, in fact, named `t`.
This is the `t` that we're referring to.

Combining all this together, we can advance our macro implementation to include
the `toMap` method:

```scala
c.Expr[Mappable[T]] { q"""
  new Mappable[$tpe] {
    def toMap(t: $tpe) = Map(..$toMapParams)
    def fromMap(map: Map[String, Any]) = ???
  }
""" }
```

The `toMap` method is implemented as described before. The `t` variable now has
sufficient context to give the code meaning. We use `..$toMapParams` to indicate
that we are passing a `List[T]`. There is a `...` variant for `List[List[T]]`
(e.g., parameter lists for methods) which are shown [on the quasiquotes doc
page](http://docs.scala-lang.org/overviews/macros/quasiquotes.html), but I
haven't had a chance to try them out.

If you want, you can comment out the `fromMap` method from the `Mappable` trait and
the macro implementation to give `toMap` a try:

```scala
def mapify[T: Mappable](t: T) =
  implicitly[Mappable[T]].toMap(t)

case class Item(name: String, price: Double)
val map = mapify(Item("lunch", 15.5))
println(map("name")) // "lunch"
println(map("price")) // 15.5
```

Cool, huh?

## writing `fromMap`

The `fromMap` method can be written in an analogous way. Let's take a look at
what we need:

```scala
def fromMap(map: Map[String, Any]) = Person(
  map("name").asInstanceOf[String]
  map("age").asInstanceOf[Int])
```

There are two things we need here that we didn't need for the implementation of
`toMap`: the companion object for the `apply` method, and the type of each
parameter for the cast. We can get both from the `tpe` variable:

```scala
val companion = tpe.typeSymbol.companionSymbol
def returnType(name: Name) = tpe.declaration(name).typeSignature
```

Using these and the same list of fields we had from the `toMap` implementation,
we can generate the `fromMap` implementation:

```scala
val fromMapParams = fields.map { field =>
  val name = field.name
  val decoded = name.decoded
  val returnType = tpe.declaration(name).typeSignature
  q"map($decoded).asInstanceOf[$returnType]"
}

c.Expr[Mappable[T]] { q"""
  new Mappable[$tpe] {
    def toMap(t: $tpe) = Map(..$toMapParams)
    def fromMap(map: Map[String, Any]) = $companion(..$fromMapParams)
  }
""" }
```

Remember that decoded is a `String` that gets lifted into an AST by the
quasiquotes. `map` will be the name of the variable that gets passed to the
`fromMap` method. The factory for the case class is the `apply` method of the
companion object, which we can call by doing a function application directly on
the companion object's symbol, just like in standard Scala.

It's important to note that the order of the parameters that get fed into the
`apply` method is important. This is why in the beginning we chose to retrieve
the list of parameters from the primary constructor. By doing so, we've
guaranteed ourselves that the order will indeed be correct.

And that's it! You can try it out like this:

```scala
def materialize[T: Mappable](map: Map[String, Any]) =
  implicitly[Mappable[T]].fromMap(map)

case class Item(name: String, price: Double)
val item = materialize[Item](Map("name" -> "dinner", "price" -> 25.8))
println(item.name) // "dinner"
println(item.price) // 25.8
```

## wrapping it up

This is the complete implementation of the macro. You can also find it in the
`complete-example` branch of my [macro template
repo](https://github.com/echojc/scala-macro-template). I've taken the liberty
to simplify the code where possible to make it short and concise.

```scala
import scala.reflect.macros.Context

trait Mappable[T] {
  def toMap(t: T): Map[String, Any]
  def fromMap(map: Map[String, Any]): T
}

object Mappable {
  implicit def materializeMappable[T]: Mappable[T] =
    macro materializeMappableImpl[T]

  def materializeMappableImpl[T: c.WeakTypeTag](c: Context): c.Expr[Mappable[T]] = {
    import c.universe._
    val tpe = weakTypeOf[T]
    val companion = tpe.typeSymbol.companionSymbol

    val fields = tpe.declarations.collectFirst {
      case m: MethodSymbol if m.isPrimaryConstructor ⇒ m
    }.get.paramss.head

    val (toMapParams, fromMapParams) = fields.map { field ⇒
      val name = field.name
      val decoded = name.decoded
      val returnType = tpe.declaration(name).typeSignature

      (q"$decoded → t.$name", q"map($decoded).asInstanceOf[$returnType]")
    }.unzip

    c.Expr[Mappable[T]] { q"""
      new Mappable[$tpe] {
        def toMap(t: $tpe): Map[String, Any] = Map(..$toMapParams)
        def fromMap(map: Map[String, Any]): $tpe = $companion(..$fromMapParams)
      }
    """ }
  }
}
```

I hope this introduction to Scala macros has been helpful. I'm no expert in
them and most of what I've done here was the result of [scouring the Scala
docs](http://www.scala-lang.org/files/archive/api/2.11.0-M5/index.html#scala.reflect.api.Universe)
and a lot of googling. Comments and suggestions are most welcome!
