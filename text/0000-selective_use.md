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

Until the introduction of importing macros with `use`, `use` statements pulled from two namespaces - which I'll call "types" (which includes traits and modules) and "values" (functions, constants, ...). These generally don't collide, as types and values/functions use different naming conventions, but when they do it's impossible to import only one item from a module where there are two with the same name . This problem has been exacerbated by adding the macro namespace to `use` imports, as now macros can also be imported along with a function and module of the same name.

## Primary Usecase: Disambiguation
If there is a stable item (e.g. a macro) in the standard library, and an unstable item (e.g. a module) of the same name in the same module, then a `use` statement intended to import the macro will trigger the stability lint and not compile.


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

To simplify both parsing and reading, `use_type` is only valid at the start of the `use` declaration.

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
  - This RFC mainly exists to solve what is currently a papercut, but with the introduction of `use`-ing macros it may become more of an issue.
- Use an attribute instead of introducing new syntax
  - Less obvious that it's changing the behavior, but permits much more versatility in symbol names.
  - `#[source_namespace(macro,union,...)]`
- Only allow disambiguation between macros and other items, by requiring macro imports to be suffixed by `!`
  - A viable option to avoid the collision between modules/functions and macros, but doesn't prevent the collision between modules and functions
  - `use core::panic!;`

# Unresolved questions
[unresolved]: #unresolved-questions

- Should annotations be allowed inside the braced portion of use statements?
- With unit/tuple structs, the name exists in two namespaces (both type and value). This may be nice to be shown/distinguished in the annotation.
- There are three namespaces (types/modules, values, and macros), should the annotation just specify which namespace is desired? If so, what identifier/keyword should be used for each?

