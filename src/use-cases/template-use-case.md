# Use case: NAME

In one sentence, what do users want to do?

```rust
let example = demonstrates_functionality();

assert!(example.is_concise());
```

## Features needed

### Variadic functions

- [ ] E: Homogenous variadic functions
- [ ] I: Variadic function arguments implement `IntoIterator`
- [ ] I: Heterogenous variadic functions

### Variadic tuples

- [ ] E: Tuple trait
- [ ] E: Tuple item trait bounds
- [ ] E: Tuple value iteration
- [ ] I: Explicit variadic tuples
- [ ] U: Argument packing and unpacking
- [ ] U: Tuple manipulation
- [ ] U: `HIterator` and `IntoHIterator` traits
- [ ] C: Tuple type iteration

### Variadic generics

- [ ] E: Unbounded variadic generics
- [ ] E: Variadic generic trait bounds

## Detailed explanation

What do users want to do?

Are there any particularly challenging edge cases?
Would users be willing to accept limitations that may make implementation simpler?
How does this use case require the specific features outlined above?

## Workarounds

Each workaround should have its own subsection.
In real code bases that exist today, how do users currently work around the lack of this feature?

Compare snippets of code before and after.
Why would this feature make things better?

If you can find them, link to real-world uses of these workarounds.
