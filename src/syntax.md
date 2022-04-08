# Syntax

Ah syntax: the most visible part of any language proposal.

For each of the proposed [features](features.md), we need a clear, unambigous syntax.

Bikeshed here!

## Variadic generics and functions

### Homogenous variadic function arguments

`..T` syntax:

```rust
fn sum<T>(summands: ..T)
```

This is taken directly from the [original draft RFC](https://github.com/rust-lang/rfcs/issues/376).

### Heterogenous variadic function arguments

We need to be able to distinguish between the case where the argument types are guaranteed to be identical, and where they can differ.

`..*T` syntax:

```rust
fn pretty_print<*T: Display>(items: ..*T)
```

Inspired by the use of `*` in regexes as a wildcard.
Too confusing: `*` is already used to mean dereference and pointer, and `*T` is a valid type.

`..?T` syntax:

```rust
fn pretty_print<?T: Display>(items: ..?T)
```

Indicates some sense of uncertainty about the type's identity.
`?` is occasionally used as part of traits to remove a `Sized` bound.

### Variadic tuples

## Tuple item trait bound

## Tuple concatenation
