# Use case: Heterogenous lists

Create collections of arbitrary legnth with mixed types (which typically share at least one trait of interest).

```rust
// This type implements `Tuple`, as it is a tuple
// And each item in this type also implements `Display`
let hlist: impl Tuple<impl Debug> = (3, 3.14, "pie");
for item in hlist {
    dbg!(item);
}
```

## Features desired

Variadic tuples:

- [x] Heterogenous variadic tuples
- [x] Tuple item trait bounds
- [x] Tuple value iteration
- [x] Tuple trait
- [x] Tuple type manipulation
- [ ] Homogenous variadic tuples
- [x] Variadic tuple destructuring
  
Variadic functions:

- [ ] Homogenous variadic functions
- [ ] Heterogenous variadic functions
- [ ] Argument packing and unpacking
- [ ] Homogenous variadic arguments implement `IntoIterator`
- [ ] Flexible variadic argument position

Variadic generics:

- [ ] Minimum variadic generics
- [ ] Variadic generic type iteration
- [ ] Flexible variadic generic position

## Detailed explanation

Heterogenous lists are a broadly useful primitive, although they can be rather abstract.
In particular, heterogenous lists are a foundational tool of modern Rust [ECS] Entity-Component-System engines like [`hecs`](https://docs.rs/hecs/latest/hecs/), [`bevy`](https://github.com/bevyengine/bevy) and [`edict`](https://github.com/zakarumych/edict), allowing users to insert and access data in an efficient type-safe way.
This style of approach was described in the paper [*Strongly Typed Collections of Heterogenous Data*](http://homepages.cwi.nl/~ralf/HList/), where the authors applied this approach to classical (rather than in-memory) databases.

The simplest use case occurs when attempting to insert multiple components (typed units of data) into an entity (very roughly analogous to a database row):

```rust
// A World is a heterogenous, strongly typed collection of entity-component data
impl World {
    // Entity is a unique identifier
    // Components are strongly and statically typed, and each must implement the `Component` trait
    fn insert_component<C: Component>(&mut self, entity: Entity, component: C) {}
}
```

This is straightforward, but what if we wanted to insert mutliple components at once?

```rust
impl World {
    // The Box<dyn Component> approach is easy
    // But the performance overhead is unacceptable for this application
    fn insert_components(&mut self, entity: Entity, components: Vec<Box dyn Component>) {}
}
```

In a world with variadics, we could write this method as:

```rust
impl World {
    // Using variadic tuples
    fn insert_components(&mut self, entity: Entity, components: impl Tuple<impl Component>) {}

    // Using variadic functions
    fn insert_components<C: Component>(&mut self, entity: Entity, components: ..*C>) {}

    // Using heterogenous IntoHIterator sugar
    fn insert_components(&mut self, entity: Entity, components: impl IntoHIterator<impl Component>>) {}
}
```

Having a built-in trait for all tuples (the hypothetical `Tuple` trait) would make it much easier to reason about and work with heterogenous lists, as well as provide a home for convenience methods.
Users may want to:

- iterate over the values of a variadic tuple
- iterate over the types of a variadic tuple
  - ensure that the types are unique
- push and pop items onto a variadic tuple
- filter the items of a variadic tuple
- create variadic tuples [from a partial list of values](https://docs.rs/frunk/latest/frunk/hlist/trait.LiftFrom.html)
  - this is closely related to [default arguments](https://internals.rust-lang.org/t/named-default-arguments-a-review-proposal-and-macro-implementation/8396)

At a type level, the most fundamental addition is the ability to specify trait bounds for the tuple items.
For example, you may want to enforce that for your `PrettyPrintable` trait, each type used in your variadic tuple must also implement the `Display` trait.

Critically, this is not the same thing as a trait bound on `PrettyPrintable`!
`PrettyPrintable: Tuple + Display` implies that the type itself implements `Display`.
Instead, we care about the fact that the items themself implement the trait.
We cannot use a (classical) associated type here either, because associated types must have a single fixed type.
Instead, we need a new concept: a **tuple item trait bound**.

## Workarounds

## `frunk`

The `frunk` crate provides an [`Hlist` trait](https://docs.rs/frunk/latest/frunk/hlist/trait.HList.html) for exactly this purpose.

Under the hood, it is implemented using an [extensive collection of macros](https://github.com/lloydmeta/frunk/blob/2b16d2cabdf0ce6ef40f92562ba5265f33eb72fa/core/src/macros.rs#L30), and has many useful methods.

It has several limitations though:

- there is no way to specify **tuple item trait bounds**
- the types must be specified in advance as type hints
- the documentation is very academic, making it hard for users to see why it might be useful to them
- there is significant boilerplate involved with the construction of `HList` types
- the implementation involves [extremely advanced macros](https://github.com/lloydmeta/frunk/blob/2b16d2cabdf0ce6ef40f92562ba5265f33eb72fa/core/src/macros.rs#L228)

### `all_tuples!` macro

With the ~mildly cursed~ power of macros, we can emulate variadic tuples today!
Suppose we're trying to implement a simple `sum` function, but want to use an API that looks like variadic tuples for the Aesthetic.

```rust
fn sum<T: Add, I: Summable<T>>(addends: I) -> T{
    addends.into_iter().reduce(|(a, b)|{a + b})
}

trait Summable {
    type T: Add;

    fn into_iter(self) -> impl Iterator<Item=T>
}

struct SumIter<T: Add> {
    storage: Vec<T>,
    index: usize,
}

impl Iterator for SumIter {
   // You know how iterator adaptor structs work
}

assert_eq!(sum((1,2,3,4)), 10);
```

We can make this magic work by implenenting the `Summable` trait for tuple types:

```rust
// We can't just implement `IntoIterator` for `(T, T)` because of orphan rules
impl <T: Add> Summable for (T, T) {
    fn into_iter(self) -> SumIter<T> {
        SumIter{
            storage: vec![self.0, self.1],
            index: 0,
        }
    }
}
```

Use a macro to implement this for some large range of tuple sizes, and voila: knock-off homogenous variadic tuples!
This has some drawbacks though:

- we cannot implement the `IntoIterator` trait directly (even for homogenous tuples), because of orphan rules
- there is a *lot* of added complexity and boilerplate
- confusing [doc spam](https://docs.rs/bevy/0.6/bevy/ecs/bundle/trait.Bundle.html#impl-Bundle-for-(C0%2C%20C1))
- [terrible error messages](https://github.com/bevyengine/bevy/issues/1519)
- painful increases to compile times and executable sizes
- we can only implement this trait up to a fixed N

Of course, this is the easy case: we can actually take this approach and mimic trait-bound heterogenous variadic tuples!

```rust
// By splitting the generics apart, we can get heterogenous types
impl <T: Display, U: Display> PrettyPrintable for (T, U) {}
```

In practice, this requires a macro-generating macro and some *creative* use of the type system, but, as shown by [bevy's `all_tuples!` macro](https://github.com/bevyengine/bevy/blob/032b0f4bac9d9d7ea9820b774d4a9124ae46e33b/crates/bevy_ecs/macros/src/lib.rs#L49) it can be done.

Unsurprisingly, all of the problems listed above are amplified when working with heterogenous fake-variadics.

### `for` trait

TODO: inspect how this is done in [`legion`](https://github.com/amethyst/legion/blob/093744ab559cc55edde8b45537793d08b7cd97cd/src/internals/systems/system.rs#L138)
