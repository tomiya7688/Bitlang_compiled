# Bitlang compiled Numeric Types

Status: Draft / inherited core rule

Bitlang compiled does **not** inherit C primitive integer widths or C's implementation-defined integer model.

This is an intentional exception to the general rule that C-compatible syntax and semantics should be reused where possible.

The numeric type model is inherited from Bitlang itself.

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
```

`Int10x32` means a signed integer with radix 10 and a width of 32 bits.

`Uint10x32` means an unsigned integer with radix 10 and a width of 32 bits.

The bit width is therefore part of the type itself and must not be reconstructed from a target C implementation's `int`, `long`, or pointer size.

## Relationship to Bitlang source and preprocessed

Source-level shorthand is resolved before Bitlang compiled is produced.

For example, Bitlang source may contain:

```text
int value
```

The canonical Bitlang representation is:

```text
Int10x32 value
```

Bitlang compiled should retain the canonical type rather than converting it back into an ambiguous C primitive type.

Consequently compiler-generated Bitlang compiled should prefer:

```c
Int10x32 value;
Uint10x64 size;
```

rather than:

```c
int value;
unsigned long size;
```

The declaration grammar remains C-like; the type system remains Bitlang's.

## Signedness

`Int` is signed.

`Uint` is unsigned.

Signedness is explicit in the canonical type and is not inferred from target ABI defaults.

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

Values of different radix types are not directly compatible operands merely because they have the same bit width and numeric value.

A representation conversion must already be explicit, normally through Bitlang's pulse semantics, before incompatible radix types are combined.

Bitlang compiled must preserve this distinction until lowering has produced an equivalent backend representation.

## No C integer promotions

Bitlang compiled does not inherit C's integer promotions or usual arithmetic conversions.

It must not silently:

- widen or narrow an integer,
- change signedness,
- change radix,
- choose a target-dependent `int` width,
- convert operands merely to make an expression compile.

Required conversions must be explicit in the compiled representation.

## Overflow

Numeric overflow is an error by default, following Bitlang semantics.

Bitlang compiled must not silently adopt C signed-overflow behavior or unsigned wraparound merely because C is a backend target.

If a distinct operation explicitly requests wrapping or another overflow policy, that operation must remain explicit through lowering.

## Floating-point types

Floating-point types use the same general Bitlang naming principle when radix and bit width are relevant:

```text
<TypeName><Radix>x<BitWidth>
```

If radix is omitted at source level, Bitlang's default is resolved before canonical compiled output is produced.

Exact floating-point representation and backend mapping remain separate specification work, but C's target-dependent primitive spelling must not erase the canonical Bitlang type information.

## C backend lowering

The C backend is responsible for translating canonical Bitlang compiled types into C representations that preserve the required width and semantics.

For example, a backend may lower a canonical 32-bit integer to a fixed-width C integer representation rather than plain `int`.

The backend mapping is not allowed to redefine the source type according to whatever width a C implementation happens to assign to `int`, `long`, or related primitive types.

Where radix or overflow semantics require more than a C primitive type can express directly, the backend must preserve those semantics through generated checks, helper operations, metadata, or another defined lowering strategy.

## Core rule

Bitlang compiled is C-like in syntax and low-level structure, but **numeric representation remains Bitlang-native**.

This is a foundational rule of the language rather than an optional backend convention.
