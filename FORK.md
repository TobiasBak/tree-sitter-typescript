# Fork notes

Branch `parse-fixes` is upstream tag `v0.23.2` plus grammar fixes for valid TypeScript that the released parsers reject or misread. `master` mirrors upstream. Consumers pin a revision of `parse-fixes`.

## Fixes

- An import type query as a type argument of a call, `f<typeof import("./x")>()`, failed to parse or was read as a comparison chain (upstream issue 367).
- `await f<T>(x)` was read as `(await f)<T>(x)`. It is now an `await_expression` around the generic call.
- A tagged template with type arguments, ``styled.div<{ a: string }>`...` ``, failed to parse (upstream issue 341).
- `in` and `out` variance annotations on type parameters failed to parse.
- A bare `&` in a JSX attribute string, as in `href="?a=1&b=2"`, failed to parse.

Generic calls and `new` with type arguments now resolve their ambiguity against comparisons through dynamic precedence instead of the tie-break between equally ranked trees.

## Known gap

A comparison whose right operand is `typeof` applied to a call or member, `a < typeof f()`, fails to parse, as it does upstream.

## Regenerate

With tree-sitter CLI 0.26.9, after changing `common/define-grammar.js`:

```sh
pnpm install --ignore-scripts
(cd typescript && tree-sitter generate --abi 14)
(cd tsx && tree-sitter generate --abi 14)
tree-sitter test
(cd tsx && tree-sitter test)
```

ABI 14 matches the parsers upstream ships. Regenerating the unmodified `v0.23.2` grammar this way yields the same trees as the released parsers on 4,834 TypeScript and TSX files from 19 repositories; the fixes changed trees only in files that contain the constructs above.
