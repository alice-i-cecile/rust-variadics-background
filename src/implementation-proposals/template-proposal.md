# Proposal: NAME

In a single sentence, what does this proposal allow and how does it do so?

## Features provided

Variadic tuples:

- [ ] E: Tuple trait
- [ ] E: Homogenous variadic tuples
- [ ] E: Heterogenous variadic tuples
- [ ] E: Tuple item trait bounds
- [ ] E: Tuple value iteration
- [ ] I: Tuple type manipulation
- [ ] U: Variadic tuple destructuring
  
Variadic functions:

- [ ] E: Homogenous variadic functions
- [ ] E: Heterogenous variadic functions
- [ ] U: Argument packing and unpacking
- [ ] C: Homogenous variadic function arguments implement `IntoIterator`
- [ ] C: Flexible variadic function argument position

Variadic generics:

- [ ] E: Minimum variadic generics
- [ ] C: Flexible variadic generic argument position
- [ ] C: Variadic generic type iteration

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

### Questions to answer during implementation
