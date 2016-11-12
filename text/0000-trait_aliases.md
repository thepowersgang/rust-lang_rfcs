- Feature Name: `trait_aliases`
- Start Date: 2016-10-09
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Introduce the syntax `trait Alias<Arg> = OtherTrait<Foo, Arg>;` that provides an alias for a trait in the same way as `type` does.

# Motivation
[motivation]: #motivation

It is often desirable to give an existing trait a shorter name, for clarity or to aid in refactoring. The existing method of this
(creating a new trait and implementing it for all `T: OtherTrait`) is overly wordy, and doesn't quite cover all cases.

# Detailed design
[design]: #detailed-design

Introduce the following syntax for defining a trait alias in item position

```
[pub] trait Name[<Generics...>] = Trait<Args...>;
```

This item acts in the same way as existing type aliases do - no validity checking is done on the passed bounds set, and the alias
naively replaces the existing trait name with the righthand side (substituting parameters as required).

Unlike type aliases, this would be valid as a bound to a generic parameter (or to another trait) as well as being a valid type
(becoming a trait object).

# Drawbacks
[drawbacks]: #drawbacks

- Additional complexity to type resolution

# Alternatives
[alternatives]: #alternatives

- The existing workaround exists and covers most usecases
 - It however doesn't work correctly when associated type bounds are involved.

# Unresolved questions
[unresolved]: #unresolved-questions

- Should this syntax support multiple traits under the same alias
 - Doing so restricts the alias' use as trait object
 - Could be restricted to only support a single non-auto trait
