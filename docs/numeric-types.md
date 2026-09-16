# Bitlang compiled Numeric Types

Status: Draft / inherited core rule

Bitlang compiled does **not** define a separate numeric type system.

Integer and floating-point numeric semantics are inherited from Bitlang's canonical type system. C compatibility applies to syntax and low-level program structure; it does not replace Bitlang numeric types with C primitive types.

This is a foundational exception to the general rule that C-compatible syntax and semantics should be reused where possible.

## Canonical type-name representation

For numeric types that carry both a radix/base and a bit width, the canonical form is:

```text
<TypeName><Radix>x<BitWidth>
```

Examples:

```text
Int10x32
Uint10x32
Int2x8
Int16x64
Float10x32
Float10x64
```

The radix and bit width are part of the type itself. They must not be reconstructed from a target C implementation's `int`, `long`, `float`, `double`, pointer size, or ABI defaults.

## Relationship to Bitlang source and preprocessed

Source-level shorthand, defaults, inferred types, and explicitly configured conversion automation are resolved before Bitlang compiled is produced.

For example, Bitlang source may contain:

```text
int value
```

The canonical Bitlang representation is:

```text
Int10x32 value
```

Bitlang compiled retains the canonical type rather than converting it back into an ambiguous C primitive type.

Consequently compiler-generated Bitlang compiled should use forms such as:

```c
Int10x32 value;
Uint10x64 size;
Float10x32 ratio;
```

rather than using C primitive spellings as the semantic type:

```c
int value;
unsigned long size;
float ratio;
```

The declaration grammar remains C-like; the numeric type system remains Bitlang's.

## Signed and unsigned integers

`Int` is signed.

`Uint` is unsigned.

Signedness is explicit in the canonical type and is not inferred from target ABI defaults.

## Floating-point types

Floating-point types follow the same Bitlang canonical representation rule as integers when radix and bit width are meaningful:

```text
<TypeName><Radix>x<BitWidth>
```

The radix and bit width are therefore explicit type information for floating-point values as well.

If the radix is omitted at Bitlang source level, Bitlang's default radix is resolved before canonical Bitlang compiled output is produced.

Bitlang compiled does not redefine floating-point types according to C's `float`, `double`, or `long double`. Those are possible backend representations only.

Any floating-point behavior already defined by the Bitlang type system is inherited unchanged by Bitlang compiled. Backend-specific implementation details remain the responsibility of lowering.

## Radix is semantic

Radix is part of the type, not only literal or display formatting.

For example:

```text
Int2x32
Int8x32
Int10x32
Int16x32
```

are distinct canonical types.

The same principle applies to other numeric families that carry radix information.

Values of different radix types are not directly compatible operands merely because they have the same bit width and numeric value.

A representation conversion must already be explicit, normally through Bitlang's pulse semantics, before incompatible radix types are combined.

Bitlang compiled preserves this distinction until lowering has produced an equivalent backend representation.

## No C numeric promotions

Bitlang compiled does not inherit C's integer promotions, usual arithmetic conversions, or implicit numeric conversions.

It must not silently:

- widen or narrow a numeric value,
- change signedness,
- change radix,
- choose a target-dependent primitive width,
- convert between integer and floating-point families,
- convert operands merely to make an expression compile.

Required conversions must already be explicit in the compiled representation.

## Cast and pulse

Bitlang's distinction between cast and pulse remains applicable.

A cast changes the semantic type of a value. A pulse changes representation while preserving the source value, including radix-oriented representation changes.

Bitlang compiled must preserve the already-resolved operation rather than replacing it with a C implicit conversion.

## Overflow

Numeric overflow is an error by default, following Bitlang semantics.

Bitlang compiled must not silently adopt C signed-overflow behavior, unsigned wraparound, or backend-specific floating-point behavior merely because C is a backend target.

If a distinct operation explicitly requests wrapping or another overflow policy, that operation must remain explicit through lowering.

## C backend lowering

The C backend is responsible for translating canonical Bitlang compiled numeric types into C representations that preserve the required width and semantics.

A backend may use fixed-width C integer types, C floating-point types, generated helper types, runtime checks, helper operations, or another defined representation as required.

The backend mapping is not allowed to redefine the source type according to whatever width or semantics a C implementation happens to assign to its primitive types.

Where radix, overflow, precision, or other Bitlang semantics require more than a C primitive type can express directly, the backend must preserve those semantics through the lowering strategy.

## Core rule

Bitlang compiled is C-like in syntax and low-level structure, but **numeric types and numeric semantics remain Bitlang-native**.

This applies to both integer and floating-point numeric types and is a foundational language rule rather than an optional backend convention.
