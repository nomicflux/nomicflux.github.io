---
title: X-Functors
---

I'm largely writing this post to get my own ideas straight about the different types of \\.*functors\\ (`Functors`, `Bifunctors`, and `Profunctors`, to start), and if this helps anyone else, great!

I've seen similar attempts before, but they rely more on things like the functor laws to give reasons why we need `Bifunctor`s, etc.  I think there are far more basic, structural reasons why - not that law-abiding functors are not important!  So the following is based on the big-pictures reasons for these varieties of abstract nonsense.

### Functors

Plain 'ole `Functor`s are relatively simple - they're anything you can `map` (or `fmap`) over.  For example, you could have a `List`:

```haskell
sampleList :: [Int]
sampleList = [0,1,2]
```

All of the values in that `List` are normal `Int` values, so why shouldn't we be able to use normal functions that take an `Int` and return something else?  For example, we could just add 1 to a normal `Int`, so we should be able to add 1 to a `List` of `Int`s:

```haskell
-- [1,2,3]
easyAs :: [Int]
easyAs = fmap (+ 1) sampleList
```

Or perhaps we have a potentially `null` value, which we'll represent with a `Maybe`.  This is nice for double-checking to make sure that we don't accidentally let a `null` through. But if there is a value, we should be able to use normal functions just fine, and if there is not, we should be able to stick with our `Nothing`, no matter what the content.  `Functor`s are here again to help:

```haskell
-- Just 6
addThree :: Maybe Int
addThree  = fmap (+ 3) (Just 3)

-- Nothing
dontAddThree :: Maybe Int
dontAddThree = fmap (+ 3) Nothing
```

What happens if we take datatypes with multiple type parameters, though?  For example, `Either`:

```haskell
-- Right "3"
showThree :: Either Int String
showThree = fmap show (Right 3)

-- Left 3
noShowThree :: Either Int String
noShowThree = fmap show (Left 3)
```

We can `map` over a `Right`, but not over a `Left`.  Similarly for a `Tuple`:

```haskell
-- (1, "2")
mapSecond :: (Int, String)
mapSecond = fmap show (1,2)
```

This seems somewhat arbitrary - wouldn't it be nice to have an `Either` or a `Tuple` with no bias between the two arguments?  But the two type parameters for an `Either` or a `Tuple` don't have to be the same.  If we have `Either a b` and we want to map a function `b -> d` over it, this can only work on the second parameter.  We would have to supply a second function `a -> c` in order to make sure that we could work with the other alternative.  `fmap` only lets us supply a single function.

### Bifunctors

That's where `Bifunctor`s come in:

```
-- ("1", 2)
mapJustFirst = first show (1,2)
-- (1, "2")
mapJustSecond = second show (1,2)
-- ("1", "2")
mapBoth = bimap show show (1,2)

-- Right "2"
mapRight = second show (Right 2)
-- Right 2
dontMapLeft = first show (Right 2)
-- Left "1"
mapLeft = first show (Left 1)
-- Right "2"
mapEither = bimap show show (Right 2)
```

With a `Bifunctor`, we can treat two separate paths equally, since we can now provide options for both cases.

If you are coming from the AJAX world, or used to doing anything with Promises or other asynchronous sources of error, you may see a familiar pattern here: you supply one callback in case of success, and another in case of error.  When you could have two separate "output" values, you can provide a way to transform either to what you need.  That's a `Bifunctor`.

Let's take another example where we have a two-sided structure: Functions.  We've got something of the form `a -> b`, and we want to change both sides.  We know that `a ->` (or `(->) a`) is a `Functor`:

```haskell
mapFunction :: Int -> String
mapFunction = fmap show (+ 1)   -- Same as `show . (+ 1)`

-- "4"
useTheFunction :: String
useTheFunction = mapFunction 3
```

This is just the same as function composition.  We take a function `a -> b`, we map some change to the output `b -> c`, and we get `a -> c`.

Now let's try to change the input: If we have `a -> b` and a function `a -> c`, we should get `c -> b`, right?

```haskell
bifunctorIt :: String -> Int
bifunctorIt = first show (+ 1)  -- Error!
```

Do you see the problem?  After changing an `Int` into `String` through `show`, we no longer have a way of doing addition!  Of course, we could probably parse the `String`, handle the `Maybe` or the `Either`  or some other value resulting from that, provide a default value in case of failure, then add 1 and get back an `Int`.  But that's just for those specific types - we don't have a way in general of converting from some arbitrary type to an `Int`, and no default value to use in all cases (for example, if we're adding, we'd want a default of 0, but if we're multiplying, we'd want a default of 1).

### Profunctors

But that doesn't mean we can't do anything.  The main problem with trying to turn a function into a `Bifunctor` was that it would let us perform an arbitrary conversion `a -> c`, then the `Bifunctor` machinery would have to figure out what to do to handle that - and that's beyond the scope of your average typeclass.  What if we reversed that process, though: supply all of the conversion machinery, at which point Haskell can do what it does best and simply check that the types align?

In other words, instead of taking a function `a -> b` and supplying a map `a -> c`, we instead supplied `c -> a`?  We tell Haskell how to get from `c` to `a`, at which point we already know how to get from `a` to `b`, and then Haskell can take us from `c` to `b` in the future:

```haskell
mapToInt :: String -> Int
mapToInt str = case reads str of
  [] -> 0
  (x, _):_ -> x

profunctorIt :: String -> Int
profunctorIt = (+ 1) . mapToInt
```

Again, we can just use function composition - this time, we start with the other function, then use our original one.  We can even bookend our function:

```haskell
justStrings :: String -> String
justStrings = show . (+ 1) . mapToInt
```

So we take a function which just works on `Int`s, parse a `String` into an `Int` to use that function, then `show` the resulting `Int` to get back to a `String`.  This is (as you might be able to tell from the header of the section) an example of a `Profunctor`.

Unlike a `Bifunctor`, which is dealing with two separate sorts of outputs (either both at once, as in a `Tuple` or perhaps a record, or with mutually exclusive possibilities, such as with an `Either` or a Promise callback), a `Profunctor` deals with something which has an input and an output.  We need to transform things *into* the input and *out of* the output; for a function `a -> b`, we need things like `_ -> a` and `b -> _`.  These give us the `lmap` and the `rmap`, respectively, for a `Profunctor`, while a `dimap` applies both at once.  But the specific names don't matter as much as the concepts - you can always go and lookup details in the documentation later.

If you want get technical, you can say that a `Profunctor` combines a `Covariant` functor (that's the one which changes the output, which moves out of a given type; `b -> c` changes `b` into `c` in the same direction as the arrow) and a `Contravariant` functor (changing the input, moving into a given type; `c -> a` changes `a` into `c` against the arrow).  And with this information, you can use `Contravariant`s by themselves for a bonus freebie functor!  `Covariant` functors are just the normal `Functor`s we know and love.

### Review

The core ideas behind all of these types of `Functor`s is that we can reuse functions we already have in order to adapt the `Functor` structure.

1. `Functor`s proper just let us change one aspect of themselves - the value inside of a `List` or a `Maybe`, or the second type parameter for an `Either` or a `Tuple`
2. `Bifunctor`s let us change multiple outputs - both sides of an `Either` or a `Tuple`, or in providing different callbacks for different situations
3. `Profunctor`s let us change an input and an output, like with a function `(->)`
