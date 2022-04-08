# Proposal: Variadic tuples are all you need

With sufficiently powerful variadic tuples, we can reduce both variadic functions and variadic generics to syntactic sugar.

## Features provided

### Variadic tuples

- [ ] E: Tuple trait
- [ ] E: Tuple item trait bounds
- [ ] E: Tuple value iteration
- [ ] I: Explicit variadic tuples
- [ ] U: Argument packing and unpacking
- [ ] U: Tuple manipulation
- [ ] U: `HIterator` and `IntoHIterator` traits
- [ ] U: Variadic tuple destructuring
- [ ] U: `HomogenousTuple` and `UniqueTuple` traits
- [ ] C: Tuple type iteration

### Variadic functions

- [x] E: Homogenous variadic functions
- [ ] I: Variadic function arguments implement `IntoIterator`
- [x] I: Heterogenous variadic functions

### Variadic generics

- [x] E: Unbounded variadic generics
- [x] E: Variadic generic trait bounds
  
## Detailed explanation

Suppose we have powerful variadic tuples already.
We must be able to:

1. Iterate over their values.
2. Ensure that each contained type obeys a trait bound.

For convenience, we also assume that we have a common `Tuple` trait that all tuples share.
We can create new existential types and trait bounds as follows:

```rust
type ExistentialTuple = impl Tuple<impl Bounds>;
trait TupleTrait: Tuple<impl Bounds> {}
```

Variadic tuples have two essential features:

1. They can contain any number of values, whose type does not need to match.
2. Their type signature can contain any number of distinct types.

As a result, we can mimic all of the essential features of variadic functions and generics by providing an existential tuple type:

```rust
fn almost_variadic<T: Tuple<impl Bounds>>(arg: T) {}
struct TypeUnion<T: Tuple>(PhantomData<T>);
```

Of course, this doesn't quite get us to our desired syntax: we must wrap our arguments and types in an extra set of `()`.

```rust
// We must write this
sum((1, 2, 3));
let type_union = TypeUnion::<(A, B, C)>::default();

// But would like to write this
sum(1,2,3);
let type_union = TypeUnion::<A, B, C>::default();
```

When a variadic function argument or a variadic generic is expected, all remaining arguments are parsed as part of that function argument or type parameter.
These are collected into a concrete tuple or tuple type, with the compiler simply substituting the corresponding existential tuple type.

## Strengths

1. Greatly simplified implementation: all of the complexity is restricted to variadic tuples.
2. Clear staged roll-out that delivers value incrementally:
   1. An unstable `Tuple` trait can be macro-implemented for small tuples today, allowing immediate and distributed iteration without worrying about the more complex challenges.
   2. Add tuple-item trait bounds.
   3. Extend `Tuple` trait to arbitrary length tuples.
   4. Add sugar and syntax for variadic functions and generics.
3. Working compiler-external prototype.
4. Variadic tuples in function arguments and type parameters allow the use of variadics in positions other than the last position.

## Drawbacks

1. Relies on the ability to implement powerful variadic tuples without the use of variadic generics or variadic functions.
2. May be less efficient than other approaches.

## Rationale

### Why can't we start with variadic functions or variadic generics?

In order to make variadics work, we need both "values with unlimited arity" and "types with unlimited arity".
Variadic functions have the former but not the latter, variadic generics have the latter but not the former.
Variadic tuples have both.

To expand on this, suppose we started with variadic functions.
How would we represent the `Fn` type of a variadic `sum` function?
We would begin with `Fn(i32) -> i32`, then `Fn(i32, i32) -> i32`.
But because there's no representation of "types with unlimited arity", we have no way of describing the appropriate trait.

### Why are "tuple item trait bounds" essential?

In most serious use cases, we need shared trait bounds across all types within the tuple.
This either allows us to ensure that they all meet some property required for effective storage,
or allows us to perform a common operation on each item.

Without a trait bound, we have no guarntee that such methods exist.

## Prior art

This flavor of design was favored by the [ancient draft RFC](https://github.com/rust-lang/rfcs/issues/376) by @eddyb.
In it, they came to the same basic conclusion: if we have variadic tuples, we can build variadic functions and variadic generics.
The rest of their proposal, to use a const-list approach to get to variadic tuples, deserves treatment in another draft proposal.

The details of this approach is directly inspired by the [`all_tuples!`](../use-cases/heterogenous-lists.md#alltuples-macro) strategy used in [`bevy`](https://github.com/bevyengine/bevy), which was in turn inherited from [`hecs`](https://github.com/Ralith/hecs), which was in turn inspired by [`legion](https://github.com/amethyst/legion).

Because Bevy is able to access some approximation of variadic tuples (critically, with trait bounds), it is able to approximate both variadic functions (e.g. [`insert_bundle`](https://docs.rs/bevy/0.6/bevy/ecs/system/struct.EntityCommands.html#method.insert_bundle)) as well as variadic generics (e.g [systems](https://github.com/bevyengine/bevy/blob/v0.6.1/examples/ecs/ecs_guide.rs) and [queries](https://docs.rs/bevy/0.6/bevy/ecs/system/struct.Query.html)).

## Unresolved questions

### Questions to answer before an RFC

- How, precisely, do we implement the required variadic tuples?
- Is it important to syntactically distinguish between heterogenous tuples and homogenous tuples?
- Is variadic function and/or variadic generic sugar worthwhile if it doesn't add any new functionality?

### Questions to answer during an RFC

### Questions to answer during implementation
