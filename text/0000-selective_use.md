- Feature Name: `selective_use`
- Start Date: 2018-09-09
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Support importing only specific items with `use` statements (instead of all items of the smae name) by annotating the `use` with the desired item type.

```rust
use macro core::panic;
use fn core::mem::zeroed;
```

# Motivation
[motivation]: #motivation

With the introduction of using `use` to import macros, it is now far more likely to have a `use` statement import multiple items (e.g. a macro and a function) when only one was desired (as macros and functions use the same name format). In the case of importing from the standard library, this can cause problems as one of the items may be unstable (e.g. the core::panic module, which has the same path as the panic macro).
This feature will allow cherry-picking items to work around this accidental import, and also provide extra clarity to the reader when the type of the import is important and non-obvious (e.g. when importing traits just to bring methods into scope).


# Detailed design
[design]: #detailed-design

Between `use` and the path, an item type reserved word can be inserted (one of: `macro`, `mod`, `type`, `trait`, `struct`, `union`, `enum`, `fn`, `const`, `static`) which applies to the entire statement. This annoation is optional, and uses the existing behavior of importing all matching items if omitted.

When the import is resolved, only items with the correct type are imported (with an error if none are matched).


## Contextual Keywords
The `union` contextual keyword does not require special handling in this case, as use paths do not require a leading `::`.

<!-- This is the bulk of the RFC. Explain the design in enough detail for somebody familiar
with the language to understand, and for somebody familiar with the compiler to implement.
This should get into specifics and corner-cases, and include examples of how the feature is used. -->

# Drawbacks
[drawbacks]: #drawbacks

- Additional complexity around name resolution (both in code and education)

# Alternatives
[alternatives]: #alternatives

- No change (the motivation is just to solve a papercut)
- Use an attribute instead of new syntax

# Unresolved questions
[unresolved]: #unresolved-questions

- Should annotations be allowed inside the braced portion of use statements?


