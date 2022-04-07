# Implementation proposals

There are *many* proposals, which may be more or less related, about how we could implement variadics in Rust.

Any accepted proposal:

- **must** be able to meet all of the **essential** [features](../features.md)
- **should** be able to meet all of the **important** features
- it **would be nice** if they were able to meet all of the nice-to-have features

Proposals that only implement variadic functions, variadic tuples or variadic generics are welcome:
we can always mix and match solutions.

If you have additions to or small optional changes to an existing proposal, please [make a PR](https://github.com/alice-i-cecile/rust-variadics-background) amending it, rather than creating an entirely new proposal.

Like always, proposals should be:

- clear
- minimally scoped
- extensible
- reversible

Because variadics is [tentatively planned for the Rust 2024 edition](https://blog.rust-lang.org/inside-rust/2022/04/04/lang-roadmap-2024.html),
[backwards-incompatible changes](https://doc.rust-lang.org/edition-guide/editions/index.html) can be part of implementation proposals if necessary.
