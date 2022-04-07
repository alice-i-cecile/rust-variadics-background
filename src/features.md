# Feature Prioritization

Based on the [use cases](use-cases/use-cases.md) examined, how can we split out and prioritize the features of variadics?

This is purely a measure of *importance*, and should be weighed against difficulty (and path-dependent effects) when deciding how to proceed in practice.
There are four levels of importance:

- **Essential \[E\]:** The *bare* minimum feature set for someone to truthfully say "Rust has variadics now!".
- **Important: \[I\]** Features that will actively frustrate users of variadics if they're missing.
- **Useful \[U\]:** Features that everyone can agree would be nice to have, but that users can live without for now.
- **Controversial \[C\]:** Features that could be cool, but may come with serious unavoidable drawbacks, encourage questionable patterns or be only tangentially related.

## Variadic functions

- E: **Homogenous variadic functions**
  - `fn sum(item: ..i32)`
  - any number of arguments can be provided to a function
  - all of the arguments have the same type
  - only one variadic argument
  - the variadic argument comes last
- I: **Variadic function arguments implement `IntoIterator`**
  - allows use of `sum(1, 2, 3)` when the signature is `fn sum(impl IntoIterator<Item = impl Add>)`
  - important to ensure APIs are flexible and elegant
- I: **Heterogenous variadic functions**
  - `fn type_ids(args: ..impl Any)`
  - the type of each argument in the variadic list of function arguments can differ
  - requires variadic generics, as we must store the list of types
  - completely useless without variadic generic trait bounds: we can't even use methods from `Any` due to its trait bounds

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
  - Provide more interesting manipulations of tuples
  - These would be automatically implemented methods on the `Tuple` trait and typically return `impl Tuple`
  - See [`frunk`'s methods on `HCons`](https://docs.rs/frunk/latest/frunk/hlist/struct.HCons.html) for more ideas

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
- E: **Variadic generic type iteration**
  - `for T in Types{...}`
  - essential for practical uses of variadic generics
