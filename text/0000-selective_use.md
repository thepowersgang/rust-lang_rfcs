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

## Guide-level explanation
Sometimes there's either multiple items with the same name in a module (e.g. a macro and an associated helper module), or it's not obvious what the type of the imported item is. To provide disambiguation in this case, use statements can be annoted with the type of item that's being imported. For example: to import just the macro `panic` without also importing the `panic` module, you can write `use macro core::panic;`


## Reference-level explanation

### Grammar
Replace the existing
```
use_decl : vis ? "use" [ path "as" ident | path_glob ] ;
```

with

```
use_decl : vis ? "use" use_type [ path "as" ident | path_glob ] ;
use_type :
 | /* empty */
 | "macro"
 | "mod"
 | "type" | "trait" | "struct" | "enum" | "union"
 | "static" | "const" | "fn"
 ;
```
### Handling
When resolving use statements, if a use type is specified the resolution code will only consider items of that specified type. The lookup will fail (and error) if the specified item is found, but isn't of the correct type (even when doing a wildcard import).

### Parsing ambiguity
`union` is a contextual keyword, which means that it cannot be followed by a `::` without causing an ambiguity between importing from a module in the crate root called `union`, or importing a union. This is not a major issue, as leading `::`s on use paths are optional and rarely used.

# Drawbacks
[drawbacks]: #drawbacks

- Additional complexity around name resolution (both in code and in teaching)

# Alternatives
[alternatives]: #alternatives

- Do nothing
- Use an attribute instead of introducing new syntax
- Only allow disambiguation between macros and other items, by requiring macro imports to be suffixed by `!`
  - `use core::panic!;`

# Unresolved questions
[unresolved]: #unresolved-questions

- Should annotations be allowed inside the braced portion of use statements?
- With unit/tuple structs, the name exists in two namespaces (both type and value). This may be nice to be shown in the annotation.
- There are three namespaces (types/modules, values, and macros), should the annotation just specify which namespace is desired? If so, what identifier/keyword should be used for each?
- Should enum variants be similarly annotated?

