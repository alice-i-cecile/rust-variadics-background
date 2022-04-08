# Use case: Functions over iterators

Functions that accept a generic iterator are very common: it would be nice if we could pass in arbitraty numbers of arguments without having to collect them first.

```rust
fn sum<T: Add, I: IntoIterator<Item = T>>(items: I) -> T {
    ...
}

assert_eq!(sum(), 0));
assert_eq!(sum(1), 1));
assert_eq!(sum(1, 2), 3));
// This is a variadic tuple!
assert_eq!(sum((1, 2, 3)), 6);
// Ordinary iterators still work
assert_eq!(sum(1..=3), 6);
```

## Features desired

### Variadic tuples

- [ ] E: Tuple trait
- [x] E: Tuple item trait bounds
- [x] E: Tuple value iteration
- [ ] I: Explicit variadic tuples
- [ ] U: Argument packing and unpacking
- [ ] U: Tuple manipulation
- [x] U: `HIterator` and `IntoHIterator` traits
- [ ] U: Variadic tuple destructuring
- [ ] U: `HomogenousTuple` and `UniqueTuple` traits
- [ ] C: Tuple type iteration

### Variadic functions

- [x] E: Homogenous variadic functions
- [x] I: Variadic function arguments implement `IntoIterator`
- [x] I: Heterogenous variadic functions

### Variadic generics

- [ ] E: Unbounded variadic generics
- [ ] E: Variadic generic trait bounds

## Detailed explanation

Operations over iterators are extremely common, and `impl IntoIterator` goes a long way to providing flexible APIs.
However, there are two ergonomic extensions to this pattern that would be nice to support:

1. The users pass in a tuple.
   1. An iterator over the values would be constructable, requiring the **tuple value iteration** feature.
   2. If the tuple is homogenously typed, it implements the `IntoIterator` trait.
   3. Each item in the tuple may have to implement some trait(s), requiring the **tuple item trait bounds** feature.
2. The user passes in more than one argument.
   1. Via the **`IntoHIterator`** feature, these are implicitly converted into a variadic tuple.
   2. This tuple is then converted into an iterator, as above.

Within the broader use case, there's an important distinction between heterogenously and homogenously typed arguments.
The type of the item in the `IntoIterator` and `Iterator` traits must be fixed.
As a result, we can split this functionality into the relatively simple **homogenous variadic functions** feature and  the more complex **hetergonenous variadic functions** feature.

## Workarounds

### Collection into an iterable

Rather than providing multiple arguments or a tuple directly into the function, these are first collected into a iterable collection.

For example:

```rust
impl InputMap {
    fn insert_chord(&mut self, input_iter: impl IntoIterator<Item=impl Into<UserInput>>, action: Action) {
        let chord: Vec<UserInput> = input_iter.into_iter().collect();
        self.map.insert(action, chord);
    }
}

let input_map = InputMap::default();

// With homogenously typed items, this isn't too bad
input_map.insert_chord([KeyCode::LCtrl, KeyCode::A]);
// With heterogenously typed items, this gets gnarly
// We're forced to box our types, and the ergonomics suck
let heterogenous_list: [Box<dyn Into<UserInput>>; 2] = [Box::new(KeyCode::LCtrl), Box::new(MouseButton::Left)]; 
input_map.insert_chord(heterogenous_list);
```

If we had full variadic functions (including a **`IntoHIterator` trait**):

```rust
impl InputMap {
    fn insert_chord(&mut self, input_iter: impl IntoHIterator<impl Into<UserInput>>, action: Action) {}
}

let input_map = InputMap::default();
// We (hopefully) save an allocation here!
input_map.insert_chord(KeyCode::LCtrl, KeyCode::A);
// So ergonomic...
// And (hopefully) we can avoid using the heap!
input_map.insert_chord(KeyCode::LCtrl, MouseButton::Left);
```

This code was inspired by the method of the same name in [`leafwing_input_manager`](https://github.com/Leafwing-Studios/leafwing_input_manager/blob/de284e51dfc4f2bb611a75da3e41d1b79fb2cbb3/src/input_map.rs#L232).

### Const generic arrays

In performance critical situations, users may choose to use const generic arrays,
sacrificing API flexibility for performance:

```rust
// This allows us to avoid conversion overhead
// but we must still allocate
// and cannot pass in e.g. a Vec<usize>
impl <T> Vec<T>{
    fn get_many_mut<const N: usize>(&mut self, indexes: [usize; N]){}
}
```

If we had homogenous variadic functions, we could write this as:

```rust
impl <T> Vec<T>{
    // This may avoid an allocation (which would matter if we were using larger types)
    // and allows us to use `vec.get_many_mut(1, 2, 42)`
    fn get_many_mut<const N: usize>(&mut self, indexes: ..usize){}
}
```

If we had homogenous variadic functions and variadic function arguments implemented `IntoIterator`, we could write this as:

```rust
impl <T> Vec<T>{
    // If we're lucky, the compiler will optimize this to avoid allocation,
    // allowing us to have a nice flexible API *and* optimal run-time performance
    fn get_many_mut<const N: usize>(&mut self, indexes: impl IntoIterator<Item=usize>){}
}
```

This code was inspired by the [`Query::get_many_mut`](https://github.com/bevyengine/bevy/blob/b33dae31ec16915989496728b16160974bcc0fc7/crates/bevy_ecs/src/system/query.rs#L736) API in Bevy.

### `all_tuples!` macro

See the [corresponding section](heterogenous-lists.md#alltuples-macro) in the [Heterogenous lists](heterogenous-lists.md) use case.
