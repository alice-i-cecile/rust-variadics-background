# Use case: NAME

Implement a trait for all tuples, often depeneding on some properties of that tuple.

```rust
impl <T: Tuple<?Item: Hash>> Hash for T {}
```

## Features desired

Variadic tuples:

- [x] Heterogenous variadic tuples
- [x] Tuple item trait bounds
- [ ] Tuple value iteration
- [x] Tuple trait
- [ ] Tuple type manipulation
- [x] Homogenous variadic tuples
- [ ] Variadic tuple destructuring
  
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

Users have a trait that has a natural implementation for tuples.
Within the standard library, this might include:

- `Clone`
- `Copy`
- `Debug`
- `Hash`
- `PartialEq`
- `Add`

and many more.

In almost all cases, users only want to implement this trait if all of the types in the tuple obey some trait bound.
Commonly, but not always, the trait they would like to implement is the same as the one used by the trait bound.

```rust
impl <T: Tuple<?Item: Hash>> Hash for T {}
```

In all cases, a blanket impl is desired, regardless of the size of the tuple.

That said, there are a few cases where users may want to implement traits for all tuples, without a [tuple item trait bound](../features.md#tuple-item-trait-bounds).
In particular, this is useful for [trait extension methods](http://xion.io/post/code/rust-extension-traits.html), especially those that attempt to implement various helper methods on tuple types such as [tuple value iteration](../features.md#tuple-value-iteration)).

In some cases (e.g the `IntoIterator` trait), users may want a guarantee that the tuple is homogenous: it contains only one type of a item.

```rust
impl <T: Tuple + HomogenousTuple> IntoIterator<Item=T> for T {}
```

In other case, users want a guarantee that the types are distinct, allowing them to be stored efficiently:

```rust
impl <B: Tuple<?Item: Component> + UniqueTuple> Bundle for B {}
```

## Workarounds

### Macros

See the discussion on the [`all_tuples!` macro](heterogenous-lists.md#alltuples-macro).
