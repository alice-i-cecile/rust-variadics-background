# Proposal: NAME

In a single sentence, what does this proposal allow and how does it do so?

## Areas covered

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
- [ ] U: Variadic tuple destructuring
- [ ] C: Tuple type iteration

### Variadic generics

- [ ] E: Unbounded variadic generics
- [ ] E: Variadic generic trait bounds

## Detailed explanation

Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

## Strengths

What unique benefits does this approach have?

## Drawbacks

Why should we not do this?

## Rationale

### Why didn't you do this obvious thing?

Because it actually wouldn't work.

### Why did you do X?

For subtle reasons, this enables us to...

## Prior art

Discuss prior art, both the good and the bad, in relation to this proposal.

Does another programming language use this approach? If so, link the appropriate [page on other languages](./../variadics-in-other-langs/language-comparisons.md) here.

Are there any helpful papers, blog posts, forum threads or other background reading about this approach?

## Unresolved questions

### Questions to answer before an RFC

### Questions to answer during an RFC
