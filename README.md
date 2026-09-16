# Bitlang compiled

Bitlang compiled is the low-level, C-like language used after `Bitlang preprocessed` in the Bitlang toolchain.

Its primary goals are:

- provide a stable low-level representation for the Bitlang compiler,
- preserve a structure that can be translated to C mechanically,
- remain suitable for optimization and static analysis,
- remove high-level ambiguity before backend translation,
- keep the language itself simple enough to be hand-written when useful, without requiring compiler-generated code to be human-friendly.

## Position in the toolchain

```text
Bitlang source
    -> Bitlang preprocessor
Bitlang preprocessed
    -> static analysis / lowering
Bitlang compiled
    -> optimization / backend translation
C / other backend representations
```

## Design rule

Where a C language construct can be adopted without conflicting with Bitlang compiled's safety, determinism, or backend requirements, Bitlang compiled follows the C form directly.

Differences from C are specified explicitly rather than inventing alternative syntax unnecessarily.

See [`docs/language-spec.md`](docs/language-spec.md) for the current language specification.
