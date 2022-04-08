# Feature Prioritization

Based on the [use cases](use-cases/use-cases.md) examined, how can we split out and prioritize the features of variadics?

This is purely a measure of *importance*, and should be weighed against difficulty (and path-dependent effects) when deciding how to proceed in practice.
There are four levels of importance:

- **Essential \[E\]:** The *bare* minimum feature set for someone to truthfully say "Rust has variadic X now!".
- **Important: \[I\]** Features that will actively frustrate users of variadics if they're missing.
- **Useful \[U\]:** Features that everyone can agree would be nice to have, but that users can live without for now.
- **Controversial \[C\]:** Features that could be cool, but may come with serious unavoidable drawbacks, encourage questionable patterns or be only tangentially related.

## Variadic tuples

- E: **Tuple trait**
  - `trait HList: Tuple`
  - Create a `Tuple` trait that is implemented by all tuples
  - This allows us to represent tuples of arbitary length existentially
- E: **Tuple item trait bounds**
  - `trait Bundle: Tuple<impl Component>`
  - each type within of the tuple must implement the given trait(s)
  - this is critically different than `trait Bundle: Tuple + Component`, which would mean that the tuple type itself must implement the `Component` trait
- E: **Tuple value iteration**
  - `(3, 3.14, "pie").values().map(|arg| println!("{arg:?}"));`
  - Cannot use the `Iterator` or `IntoIterator` trait because tuples are heterogenously typed
  - Use `HIterator` and `IntoHIterator` traits instead (if they exist)
  - Essential for working with heterogenously typed lists
  - Critical to some implementation strategies
- I: **Explicit variadic tuples**
  - `type HList = (..T)`
  - variadic tuples can be explicitly named
  - alternative syntax for `impl Tuple`
- U: **Argument packing and unpacking**
  - when provided a `my_fun(1,2,3)`, create a `(1,2,3)` tuple
  - pass the tuple `(1, 2, 3)` into `fn sum(item: ..i32)`
  - this is particularly useful for inspecting and modifying variadic arguments before passing them down call trees
- U: **Tuple manipulation**
  - `(1, 2, 3).append((4, 5, 6))`
  - `(1, 2, 3).pop()`
  - `(1, 2, 3).len()`
  - provide more interesting manipulations of tuples
  - may be a useful implementation building block, see [the ancient draft RFC](https://github.com/rust-lang/rfcs/issues/376) on variadics
  - these would be automatically implemented methods on the `Tuple` trait and typically return `impl Tuple`
  - see [`frunk`'s methods on `HCons`](https://docs.rs/frunk/latest/frunk/hlist/struct.HCons.html) for more ideas
- U: **`HIterator` and `IntoHIterator` traits**
  - allows use of `pretty_print("A", 2, 4)` when the signature is `fn pretty_print(impl IntoHIterator<Item impl Add>)`
  - heterogenously typed equivalents of `Iterator` and `IntoIterator`
  - come with blanket impls for `Iterator` and `IntoIterator` types
  - useful when creating flexible, ergonomic APIs
- U: **Variadic tuple destructuring**
  - `let (head, ..tail) = (1, 2, "foo", "bar");`
  - requires tuple manipulation
  - allows users to unpack tuples into smaller tuples
- U: **`HomogenousTuple` and `UniqueTuple` traits**
  - both traits are subtraits of `Tuple`
  - `HomogenousTuple` guarantees that all item types are identical
    - can be infallibly converted to and from arrays
    - `IntoIterator` and `FromIterator` can be blanket implemented
  - `HeterogenousTuple` guarantees that all item types are unique
    - very valuable in ECS applications
    - avoids repeated checks for uniqueness
- C: **Tuple type iteration**
  - enables `for T in Types{...}` for variadic generics
  - useful for runtime type reflection
  - useful to verify the uniqueness of types
  - types are not values in Rust, so this would involve a large degree of magic

## Variadic functions

- E: **Homogenous variadic functions**
  - `fn sum(item: ..i32)`
  - any number of arguments can be provided to a function
  - all of the arguments have the same type
  - only one variadic argument
  - the variadic argument comes last
- I: **Homogenous variadic function arguments implement `IntoIterator`**
  - allows use of `sum(1, 2, 3)` when the signature is `fn sum(impl IntoIterator<Item = impl Add>)`
  - important to ensure APIs are flexible and elegant
  - this should also be done for homogenous tuples
- I: **Heterogenous variadic functions**
  - `fn type_ids(args: ..impl Any)`
  - the type of each argument in the variadic list of function arguments can differ
  - requires variadic generics, as we must store the list of types
  - completely useless without variadic generic trait bounds: we can't even use methods from `Any` due to its trait bounds
- C: **Flexible variadic function argument position**
  - `fn compound_homogenous_variadic_function(strings: ..String, ints: ..i32, flag: bool)`
    - at compile time, we must be able to show that adjacent variadic function arguments are distinct
  - `fn compound_generic_variadic_function<A, B>(a: ..A, b: ..B)`
    - `A` can never be the same type as `B`
  - `fn compound_heterogenous_variadic_function<?S: Sized, ?U: !Sized>(flag: bool, sized_args: ?T, unsized_args: ?U)`
    - each provided argument must fit exactly one of the adjacent variadic generic types
    - requires flexible variadic generic argument positions
  - variadic function arguments can live in any position, improving ergonomics and expressivity
  - enables the use of more than one variadic function argument per function

## Variadic generics

- E: **Unbounded variadic generics**
  - `struct Union<..T>(PhantomData<..T>())`
  - any number of type arguments can be used
  - generics can be used in named structs, tuple structs, traits, type aliases and functions
  - the variadic type argument comes last
  - the type arguments do not have trait bounds
- E: **Variadic generic trait bounds**
  - `fn add_bundle<..T: Component>(component: T)`
  - each type in the variadic generic list must implement the supplied trait(s)
  - critical for more almost all use cases of variadic generics: you can't even use methods from `Any` without trait bounds
- C: **Flexible variadic generic argument position**
  - `struct Query<?Q: WorldQuery, ?F: WorldQuery + FilterFetch>`
    - any type provided must be able to be seperated into exactly one of the adjacent variadic generics
  - unbounded adjacent variadic generics are impossible
    - consider `type AmbiguousTupleUnion<?A, ?B>`
    - for any provided element, we have no way of knowing whether it belongs to `A` or `B`
  - variadic generics can live in any position, improving ergonomics and expressivity
  - enables the use of more than one variadic generic per type
  - adjacent variadics must have a guarantee that the types cannot co-exist
  - this feature becomes much more usable and useful with [negative impls](https://github.com/rust-lang/negative-impls-initiative)
  - may need [negative trait bounds](https://github.com/rust-lang/rust/issues/42721) to be useful enough: `struct Query<?Q: WorldQuery + !FilterFetch, ?F: WorldQuery + FilterFetch>`

## Impossible features

These features may seem appealing, but are fundamentally impossible within Rust.

- C: **Heterogenous `IntoIterator` and `Iterator` traits**
  - allows use of `pretty_print("A", 2, 4)` when the signature is `fn pretty_print(impl IntoIterator<Item impl Add>)`
  - would further increase API flexibility, with minimal changes to existing code bases
  - impossible because existing APIs rely on the fact that the item type is always the same
  - just create heterogenous iterator traits instead
