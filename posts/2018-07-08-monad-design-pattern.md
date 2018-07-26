---
title: Monads as Design Patterns
---

Functional abstractions like `Monad`s are often presented in the contexts of typeclasses, with attendent libraries and syntax to learn.

I think typeclasses are fantastic.  But I also think that this creates an additional hurdle.  So I'd like to focus on abstractions as design patterns.  We want to solve an issue; we have some goal in mind which we don't know how to reach yet, but the pattern gives us some relatively easy-to-assemble but not immediately straightforward methods to reach that goal.

With that in mind, today I'd like to cover `Monad`s, with a glance at `Applicative`s as well (later, I'll cover the similarities and differences between them).  In the next post, I'll present `Comonad`s as a similar design pattern.

So to be clear:

**Goal**: To be able to compose multiple items with the same added functionality
**Pattern**: Implement `flatMap`, `map`, and `unit` methods.

(You may notice that I'm taking a Scala approach here instead of the previous Haskell-based posts.  First, if you're learning Haskell, you have probably already been convinced of the use of abstractions such as `Monad`s.  Second, I'm working as a Scala dev now, so this is my life at the moment, for good or for ill.)

## The Reader Monad

To start, let's take the `Reader` monad.  `Reader[R,A]` is dependency injection - give a value of type `R`, and use it to calculate some `A`.  The following example doesn’t use any typeclass magic or anything, or actually any libraries at all; just the basic few functions required by a `Monad` so that we can compose steps.

```scala
class Reader[R,A](runReader: R => A) {
  // To create some syntactic sugar for using Reader instances && properly encapsulate
  // `runReader` above
  def apply(r: R): A = runReader(r)

  // To change the value of a Reader without changing how it uses its dependency R
  def map[B](f: A => B): Reader[R,B] = Reader((r: R) => f(runReader(r)))

  // To change the value of a Reader in a way that uses its dependency R
  def flatMap[B](f: A => Reader[R,B]): Reader[R,B] = Reader((r: R) => f(runReader(r))(r))
}

object Reader {
  // More syntactic sugar to not have to type `new` all the time
  def apply[R,A](f: R => A): Reader[R,A] = new Reader(f)

  // Take a value which doesn't use a dependency R and let it be treated as a Reader
  // which ignores the input
  def unit[R,A](a: A): Reader[R,A] = Reader((_r: R) => a)

  // Technically part of a `Traversable` pattern, but I leave it here to illustrate
  // something `Monad`s can do that `Applicative`s could not
  def sequence[R,A](values: Seq[Reader[R,A]]): Reader[R, Seq[A]] =
    values.foldLeft(Reader.unit[R, Seq[A]](Seq.empty[A])) { case (acc, next) =>
      acc.flatMap( currentList => next.map(currentList :+ _))
    }
}
```

And with that, let's create some sort of config which can be passed in as a dependency:

```scala
case class Config(name: Option[String], exclamations: Int)
```

Now that we have the setup, let's create some config-dependent values:

```scala
val greet: Reader[Config, String] = Reader(cfg => "hello " + cfg.name.getOrElse("world"))
val exclaim: Reader[Config, String] = Reader(cfg => "!" * cfg.exclamations)
```

We have two different ways to combine these before having to pass in any configuration - we can stack our `Reader` legos together to get more `Reader`s.  The first way uses the fact that we implemented `flatMap` and `map` to use Scala's `for` syntax:

```scala
val fored: Reader[Config, String] = for {
    greeting <- greet
    exclamation <- exclaim
  } yield (greeting + exclamation)
```

And the second way uses the `sequence` function we implemented to be able to take an entire sequence of config-dependent values (perhaps determined dynamically at runtime) and wrap them into a single config-dependent value:

```scala
val sequenced: Reader[Config, String] =
  Reader.sequence(List(greet, exclaim)).map(_.mkString)
```

Now let's run all three versions with configuration added:

```scala
@ sequenced(Config(Some("Scala"), 3))
res36: String = "hello Scala!!!"

@ fored(Config(Some("FP"), 5))
res37: String = "hello FP!!!!!"

@ lifted(Config(None, 10))
res47: String = "hello world!!!!!!!!!!"
```

## The State Monad

The `Reader` monad allows us to read in dependencies, but not to alter them.  It is essentially a wrapper around a function `R => A`.  If we want to update the value passed in, we'll have to make sure that we output the updated version.  This would give us a function such as: `S => (A, S)`.  We can reuse most of the above code:

```scala
class State[S,A](runState: S => (A, S)) {
  def apply(s: S): (A, S) = runState(s)

  def map[B](f: A => B): State[S,B] =
    State((s1: S) => {
      val (a, s2) = runState(s1)
      (f(a), s2)
    })

  def flatMap[B](f: A => State[S,B]): State[S,B] =
    State((s1: S) => {
      val (a, s2) = runState(s1)
      f(a)(s2)
    })

  // Helper function for dealing with States
  def modify(f: S => S): State[S, A] =
    State( s1 => {
      val (a, s2) = runState(s1)
      (a, f(s2))
    })
}

object State {
  def apply[S,A](f: S => (A, S)): State[S,A] = new State(f)

  def unit[S,A](a: A): State[S,A] = State((s: S) => (a, s))

  def sequence[S,A](values: Seq[State[S,A]]): State[S, Seq[A]] =
    values.foldLeft(State.unit[S, Seq[A]](Seq.empty[A])) { case (acc, next) =>
      acc.flatMap( currentList => next.map(currentList :+ _))
    }

  // Readers are just States which pass their arguments through unchanged
  def fromReader[R,A](reader: Reader[R,A]): State[R, A] =
    State(r => (reader(r), r))

  // Helper functions for dealing with States
  def get[S]: State[S, S] = State( s => (s, s))
  def put[S](newS: S): State[S, Unit] = State( _s => ((), newS))
}
```

It's no fun having State to modify if we don't modify it, so let's add some functions to do so:

```scala
val langs: List[String] = List("Scala", "Haskell", "Purescript")
def rotateLangs(cfg: Config): Config =
  cfg.copy(name = cfg.name.flatMap(currLang => langs lift (langs.indexOf(currLang) + 1) % langs.length))

def toneItUp(cfg: Config): Config = cfg.copy(exclamations = cfg.exclamations + 1)
```

We'll reuse our `Reader`s and have them modify the state while they read it:

```scala
val sGreet = State.fromReader(greet).modify(rotateLangs)
val sExclaim = State.fromReader(exclaim).modify(toneItDown)
```

Running it once gives us the same results as before, except it also returns an updated version of our `Config`:

```scala
val once = for {
   greeting <- sGreet
   exclamation <- sExclaim
} yield (greeting + exclamation)

@ once(Config(None, 5))
res33: (String, Config) = ("hello world!!!!!", Config(None, 6))
```

But we can also try running it multiple times in succession:

```scala
val thrice = State.sequence(List(sGreet, sExclaim, sGreet, sExclaim, sGreet, sExclaim))
                  .map(_.mkString)

@ thrice(Config(Some("Scala"), 1))
res47: (String, Config) = ("hello Scala!hello Haskell!!hello Purescript!!!", Config(Some("Scala"), 4))
```
