# A gentle introduction to variadics

What on earth does "variadic" even mean?
And why do all of the crab-obsessed programmers in my life keep making longing allusions to it, with a tinge of wistful sadness in their voice?

Let's start at the beginning.
**Variadic** is just a fancy word for "can take a variable number of arguments".
In the context of Rust, we are interested in **variadic functions** (like a hypothetical `product(factor: ..i32)` function), **variadic tuples** (`trait Bundle: Tuple + Component)` and **variadic generics** (like a hypothetical `HeterogenouslyTypedList<T>(..T)`).

Let's talk a little bit about how you might use each of those in practice.

## Variadic functions

So, suppose we want to write a function that sums a list of integers. This is pretty easy right?

```rust
fn sum(vec: Vec<i32>) -> i32 {
    let sum = 0;
    for element in vec {
        sum += element;
    }
}
```

But then your users start complaining! They *hate* having to collect their elements into a vector,
especially in areas of their code where they always have the same (small) number of elements to add together.

```rust
// They might start with an iterator
let vec = Vec::from_iter(1..5);
dbg!(sum(vec));

// Or they may just have a couple of arguments that they need to use
struct Position(i32, i32);

fn manhattan_distance_from_origin(position: Position) -> i32 {
  sum(vec![position.0, position.1])
}
```

We can help those beginning with an iterator (or who want to add other things) by using Rust's trait system and a sprinkle of functional programming:

```rust
fn sum<T: Add, I: IntoIterator<Item=T>>(items: I) -> T {
    items.into_iter().reduce(|(a, b)| a + b)
}
```

Much more general and ✨elegant✨, but our poor users who just want to be able to call `sum(1, 2, 3)` are left out in the cold.
In fact, because Rust doesn't have **function overloading**, there's *no way* to allow `sum(1, 2)` and `sum(1, 2, 3)` to work in the same namespace.
Instead, they must collect the values that they want to pass in into a collection that we can iterate over, and *then* they can use the same `IntoIterator`-powered API.

In a futuristic world with *variadic functions*, we could write:

```rust
// ..T means "any number of arguments of type T"
fn sum(items: ..i32) -> i32{
    let sum = 0;
    for element in items {
        sum += element;
    }
}

assert_eq!(sum(), 0);
assert_eq!(sum(1), 1);
assert_eq!(sum(1, 2), 3);
assert_eq!(sum(1,2,3), 6);
```

If this world was particularly utopian, we could automatically convert lists of arguments into an unnamed type that implements `IntoIterator`,
and our ✨elegant✨ code from above would Just Work when users try to pass it multiple arguments of the same type.

## Variadic tuples

Tuples are great: simple, heterogenously typed collections of elements with a fixed length.
But what if we wanted tuples of *arbitrary* length?

Your first reaction might be "that's what vectors are for!".
But in Rust, iterators must be **homogenously typed**: [we cannot have](https://beachape.com/blog/2016/10/23/rust-hlists-heterogenously-typed-list/) a vector that contains both `f32` and `i32`, even if all we want to do is print their values.

TODO: add compelling simple example here.

## Variadic generics

Variadic functions seem cool, but you remember that Rust really loves moving complex logic into the type system.
Rather than function arguments, what if you could do the same thing with *type arguments*?

For the sake of illustration, let's examine a somewhat ~cursed~ academic example: trying to check if a supplied argument belongs to a list of types provided at compile time.
Here's an example of how this function might look if we only wanted to check if it was a single type:

```rust
use std::any::Any;

fn are_you_my_type<T: Any>(arg: &dyn Any) -> bool {
   arg.is::<T>()
}
```

Alright, that was easy! Let's see *two* types:

```rust
use std::any::Any;

fn are_you_my_type<T: Any, U: Any>(arg: &dyn Any) -> bool {
   arg.is::<T>() || arg.is::<U>()
}
```

Trivial.
How about *any number* of types?

...

Now, your attempt might be to write a macro here: you can clearly see the pattern, so just add more generics!
But that won't work, because now your function doesn't work if fewer type arguments are provided.
You are cunning though, and use default type arguments to get around this, and your macro outputs a function that looks like this:

```rust
use std::any::Any;

fn are_you_my_type<T: Any = (), U: Any = (), V: Any = ()>(arg: &dyn Any) -> bool {
   arg.is::<T>() || arg.is::<U>() || arg.is::<V>()
}
```

This kind-of mostly works: you tell your macro to generate this function up to some ridiculous number of type parameters and if someone complains that they can't use your function for 273 types at once, you close the ticket as user error.

But then some *jerk* over in QA tells you that your approach is fundmentally broken: what if someone wants to check if the type is *actually* `()`?
In desperation, you turn to increasingly elaborate work-arounds: registering types manually using a builder pattern and checking if the `TypeId` matches those found in the list, experimenting with recursive type unions, providing a *macro* for your users to use...

Longing for the clean, obvious correctness of

```rust
use std::any::Any;

// ..Types in the type parameters means "any number of type arguments"
fn are_you_my_type<..Types: Any>(arg: &dyn Any) -> bool {
    let matches = false;
    // In the utopian fantasy of variadics,
    // we can iterate over lists of type parameters
    for T in Types {
        if arg.is::<T>{
            matches = true;
            break;
        }
   }
   matches
}
```

you join the other crabs, muttering furtively about "variadics" and "arity".

## Why do we need to tackle all of these things together?

Both of these features are cool, but if they're complex to implement, why can't we design and implement them seperately?
They're conceptually related, sure, but if we break up the work, can't we make this easier?

Ultimately, there are three reasons for this:

1. Syntax should be intuitively consistent betweeen variadic functions, variadic tuples and variadic generics.
2. Variadic functions, variadic generics and varidic tuples may rely on each other under the hood, depending on the [implementation strategy](implementation-proposals/proposals.md).
3. Many [features](features.md) rely on the interplay between variadic functions, tuples and generics to reach their full potential.

## Why don't we have variadics already?

If variadics are so cool, why doesn't Rust already have them?

1. The [Language Team](https://www.rust-lang.org/governance/teams/lang) has been busy with other super cool and important features.
   1. But it's now tentatively listed on the [roadmap for Rust 2024](https://blog.rust-lang.org/inside-rust/2022/04/04/lang-roadmap-2024.html#looking-forward-1)).
2. It is not clear which [features](features.md) are essential for variadics in practice.
3. We had not reached consensus on the ideal [syntax](syntax.md) for variadics.
4. There are (terrible) workarounds for a lot of the critical [use cases](use-cases/use-cases.md).
5. There are a lot of competing ideas for [how they could be implemented](implementation-proposals/proposals.md).
6. The way variadics are [implemented in other languages](variadics-in-other-langs/language-comparisons.md) sometimes leaves something to be desired.

## Glossary

As with many topics in programming language theory, a dense (but sometimes helpful) jargon has developed.
For those of us firmly rooted in practical applications of programming, here's an alphabetized translation guide:

- **Arity:** the number of arguments taken by a function.
  - The arity of `hello_world()` is zero, the arity of `add(a, b)` is two and the type-arity of `HashMap<i32, String>` is two.
- **Existential type:** a type that cannot be named, but we know some details about
  - `impl Add` is an existential type: we know that this type implements the `Add` trait, but nothing else about it
  - for an excellent discussion of the details of existential types, go read [Existential Types in Rust](https://varkor.github.io/blog/2018/07/03/existential-types-in-rust.html)
- **Function overloading:** the ability to use a single function name to mean several different (hopefully related) things based on which arguments it was passed.
- **Homogeneously typed:** all of the items in a collection are the same type.
  - The opposite of this is **hetergonenously typed**.
- **Variadic:** something that can take a varying number of arguemnts.
  - In other words, the arity is not fixed.

## Contributing

Are you a crab muttering about variadics? Come help!
This little book is hosted on GitHub at [`alice-i-cecile/rust-variadics-background`](https://github.com/alice-i-cecile/rust-variadics-background).

Whether you're confused by an explanation, would like to explain at length why I'm wrong, have a wild new idea for how we could make variadics a reality, or simply want to fix a typo, please come contribute!

Unsurprisingly, this book is an attempt to collect thoughts and solicit feedback in advance of a new variadics [RFC](https://github.com/rust-lang/rfcs).
Stay tuned!
