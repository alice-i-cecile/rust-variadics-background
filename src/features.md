# Feature Prioritization

Based on the [use cases](use-cases/use-cases.md) examined, how can we split out and prioritize the features of variadics?

At this stage, features are assessed on the basis of their utility:

- **Essential \[E\]:** The *bare* minimum feature set for someone to truthfully say "Rust has variadics now!".
- **Important: \[I\]** Features that enable fundamentally new and valuable uses of variadics.
- **Useful \[U\]:** Features that may make using variadics easier, but don't unlock fundamentally new functionality.
- **Questionable \[Q\]:** Features that may improve usability in some ways, but degrade it in others.

Note that this is independent of the difficulty of actually implementing such a feature.
This feature list is designed to be compared across languages, and so features that may be literally impossible in Rust may be included.

Difficulty, utility and path-dependent effects (e.g. feature A can be implemented in terms of feature B) should be combined later, when deciding how to proceed.

## Feature list summary

This feature list is used throughout this document, to enable consistent comparisons and at-a-glance understanding.

Variadic tuples:

- E: Heterogenous variadic tuples
- E: Tuple item trait bounds
- E: Tuple value iteration
- I: Tuple trait
- I: Tuple type manipulation
- U: Homogenous variadic tuples
- U: Variadic tuple destructuring
  
Variadic functions:

- U: Homogenous variadic functions
- U: Heterogenous variadic functions
- U: Argument packing and unpacking
- Q: Homogenous variadic arguments implement `IntoIterator`
- Q: Flexible variadic argument position

Variadic generics:

- U: Minimum variadic generics
- U: Variadic generic type iteration
- Q: Flexible variadic generic position

## Variadic tuples

### Heterogenous variadic tuples

**Utility:** Essential

**Minimal example:** `(..?T)`

Tuples of arbitrary length can be represented, and they can store different item types.
The last element of any tuple can be heterogenous variadic: `(bool, bool, ..?T)`.

Requires [homogenous variadic tuples](features.md#homogenous-variadic-tuples).
Much more useful with [tuple item trait bounds](features.md#tuple-item-trait-bounds).

A `UniqueTuple` subtrait of `Tuple` can be defined if we have a [tuple trait](features.md#tuple-trait).
The type of each element of a tuple with this trait is guaranteed to be unique.
This is useful expressive functionality, especially for ECS applications, where this is a fundamental constraint in several places.

### Tuple item trait bounds

**Utility:** Essential

**Minimal example:** `trait Bundle: Tuple<?Item: Component>`

Each type contained within the tuple must implement the provided trait(s).

Note that this is very different from `trait Bundle: Tuple + Component`, which means that every trait that implements the `Bundle` trait must implement `Component`.
This cannot be represented as a standard associated type bound (`trait Bundle: Tuple<Item: Component>`), as associated types must represent exactly one type.

This is essential to non-trivial use cases of variadic tuples.

### Tuple value iteration

**Utility:** Essential

**Minimal example:** `for i in (1,2,3)`

The values of arbitrary tuple types can be iterated over.

Almost certainly requires a [tuple trait](features.md#tuple-trait).

This is trivial for tuples that are guaranteed to be homogenous, but requires new technology for heterogenous tuples.
If we have a [tuple item trait bound](features.md#tuple-item-trait-bounds), we can treat the item type as a `dyn Trait`.
For concrete tuples (e.g. `(3, 3.14, "pi")`), we could attempt to infer whether or not tuple item trait bounds are valid
based on which trait methods are used.

### Tuple trait

**Utility:** Important

**Minimal example:** `trait HList: Tuple`

A `Tuple` trait is implemented by all tuples.

This allows us to represent tuples of arbitrary length existentially.
Useful methods can be added to it later, see [tuple value iteration](features.md#tuple-value-iteration) and [tuple type manipulation](features.md#tuple-type-manipulation).

The [`frunk` crate's methods on `HCons`](https://docs.rs/frunk/latest/frunk/hlist/struct.HCons.html) contain a nice collection of methods that may be nice to implement.

### Tuple type manipulation

**Utility:** Important

**Minimal example:** `(1,).append(2, 3) == (1, 2, 3)`

Tuple types can be converted into other tuple types.
The critical operations are:

- push
- pop
- append
- prepend

This was explored in [RFC Draft #376](https://github.com/rust-lang/rfcs/issues/376), which proposed using it as a building block for an implementation.

The principal challenge here is memory layout:

> In general, in current Rust, a tuple of the form (A, B, C) does not contain a tuple of the form (B, C) anywhere in its representation. This means if you have an (A, B, C) you can't take a reference to the "sub-tuple" (B, C) because it doesn't exist

From [@canndrew](https://github.com/rust-lang/rfcs/issues/376#issuecomment-213692855).

### Homogenous variadic tuples

**Utility:** Useful

**Minimal example:** `(..i32)`

Tuples of arbitrary length can be represented, but only if they store the same type.
The last element of any tuple can be homogenous variadic: `(bool, bool, ..String)`.

A `HomogenousTuple` subtrait of `Tuple` can be defined if we have a [tuple trait](features.md#tuple-trait).
This trait would enable:

- infallible conversions to and from arrays
- blanket impls of `IntoIterator` and `FromIterator`

### Variadic tuple destructuring

**Utility:** Useful

**Minimal example:** `let (head, tail): (i32, (i32, i32, i32)) = (4, 3, 2, 1)`

Allow tuples to be split via destructuring.

Requires [tuple type manipulation](features.md#tuple-type-manipulation) to work, as we must be able to remove elements and construct new tuple types.

This was explored in [RFC Draft #376](https://github.com/rust-lang/rfcs/issues/376).

## Variadic functions

### Homogenous variadic functions

**Utility:** Useful

**Minimal example:** `fn sum(items: ..i32) -> i32`

Variadic function arguments can absorb any number of arguments, as long as they are the same type.

This is the core feature of variadic functions, and introduces basic function overloading to Rust.
It can be emulated with variadic tuples (e.g. `fn sum(items: (..i32)`), but this requires an extra set of parentheses.

### Heterogenous variadic functions

**Utility:** Useful

**Minimal example:** `fn type_ids<?T: Any>(args: ..?T) -> Vec<TypeId>`

Variadic function arguments can absorb any number of arguments whose type is not necessarily the same, each of which must obey any trait bounds of that type.

This feature requires trait bounds to be useful, analogous to [tuple item trait bounds](features.md#tuple-item-trait-bounds).

### Argument packing and unpacking

**Utility:** Useful

**Minimal example:**

```rust
fn outer_function(args: ..i32){
    // Packing variadic arguments into a variadic tuple
    let unpacked_args: (..i32) = args;
    for mut arg in unpacked_args {
        if arg < 0.1 {
            arg = 0;
        }
    }
    // Unpacking a variadic tuple into a variadic function argument 
    inner_function(unpacked_args)
}

fn inner_function(args: ..impl Add) {}
```

Variadic function arguments can be extracted as a variadic tuple.
Variadic tuples can be passed in place of a variadic function argument.

This is most commonly used when passing variadic arguments down a function call stack, modifying or logging them as you go.
This could also be supported in a more general form, to allow unpacking a tuple into multiple function arguments.

Related to [variadic tuple destructuring](features.md#variadic-tuple-destructuring).

Becomes more useful with [tuple type manipulation](features.md#tuple-type-manipulation), as it allows you to add and remove variadic arguments.

### Homogenous variadic arguments implement `IntoIterator`

**Utility:** Questionable

**Minimal example:**

```rust
fn sum<T: Add, I: IntoIterator<Item = T>>(summands: I) -> T{
    summands.iter().reduce(|a, b| a + b)
}

let six = sum(1, 2, 3);
```

Implicitly allows the use of variadic function arguments in place of an `IntoIterator` type.

This would allow library authors to support variadic function arguments without creating dedicated APIs.

However, this syntactic sugar introduces implicit function overloading, and can be quite confusing.

TODO: add a pathological example.

Note that with variadic tuples and [tuple value iteration](features.md#tuple-value-iteration), end users can pass in a variadic tuple into an `impl IntoIterator` argument, acheiving all of the practical value of this feature at the cost of an additional set of parantheses.

This feature would be more useful (and more confusing!) with [flexible variadic function argument positions](features.md#flexible-variadic-function-argument-position), as we could have several iterator function argument types.

### Flexible variadic argument position

**Utility:** Questionable

**Minimal example:** `fn compound_homogenous_variadic_function(strings: ..String, ints: ..i32, flag: bool)`

Allow variadic function arguments in positions other than the last position, and allow more than one variadic function argument per type.

Critically, the types of adjacent variadic function argument must be clearly distinguishable.
This is simple in the non-generic homogenous case: they must be different types.
If we add generics, we must restrict the set of possible monomorphizations, to forbid outcomes where the types are the same.

This is extremely complex in the case of heterogenous variadic function arguments.
See [flexible variadic generic argument position](features.md#c-flexible-variadic-generic-argument-position) for discussion.

## Variadic generics

### Minimum variadic generics

**Utility:** Essential

**Minimal example:** `struct Union<?T>(PhantomData<?T>)`

Variadic generic type parameters can absorb any number of type arguments.
Trait bounds can be added, and apply to every type seperately:

```rust
// This stores a variadic tuple internally
struct PrettyPrintable<?T: Display>((..?T));
```

The variadic argument must come last, unless [flexible variadic generic argument position](features.md#c-flexible-variadic-generic-argument-position) is implemented.

### Flexible variadic generic position

**Utility:** Questionable

**Minimal example:**

```rust
struct SortedBySize<?S: Sized, ?U: !Sized>{
    sized_tuple: (..?S),
    unsized_tuple: (..?U),
}
```

Allow variadic generic type parameters in positions other than the last position, and allow more than one variadic generic argument per type.

### Variadic generic type iteration

**Utility:** Questionable

**Minimal example:**

```rust
fn assert_unique_types<?T: Any>(){
   let set = HashSet::new();
   for T in ?T {
        let type_id = TypeId::of::<T>();
        assert!(!set.contains(type_id));
        set.insert(type_id);
   }
}
```

Variadic generic types can be iterated over within function bodies.

This would open up a large range of possible applications.

However, it would involve quite a bit of magic: types are not representable as values in Rust.
