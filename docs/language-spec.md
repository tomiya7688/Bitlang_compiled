# Bitlang compiled Language Specification

Status: Draft

Bitlang compiled is a C-like low-level language positioned between `Bitlang preprocessed` and backend representations such as C.

The baseline rule is simple: when ordinary C syntax and semantics can be reused without conflicting with Bitlang compiled requirements, they are reused directly.

This document records the C-compatible surface and the Bitlang-specific rules that must survive lowering.

A foundational exception is the numeric type system: Bitlang compiled inherits Bitlang's canonical numeric type model, including integer and floating-point types, instead of redefining numeric semantics around C primitive types. See [`numeric-types.md`](numeric-types.md).

Bitlang-specific ownership, lifetime, property, reference, class/module, and functional lowering rules are defined in [`semantic-lowering.md`](semantic-lowering.md). Remaining non-C decisions are tracked in [`open-decisions.md`](open-decisions.md).

## 1. Translation unit

A Bitlang compiled source file is a translation unit containing declarations and definitions in a C-like form.

Statements are terminated by `;` unless the grammar of a compound construct provides its own block termination.

Blocks use braces:

```c
{
    statement;
    statement;
}
```

Bitlang source preprocessing has already completed before this stage. Bitlang preprocessor functions are not part of Bitlang compiled semantics.

## 2. Comments

C-style comments are accepted.

```c
// line comment

/* block comment */
```

## 3. Identifiers

Identifiers use the ordinary C-like lexical form: letters, digits, and `_`, with the first character not being a digit.

```c
value
player_hp
_temp2
```

Semantic identifier identity remains case-insensitive as inherited from Bitlang Preprocessed. `_` remains significant.

Therefore capitalization alone must not create a distinct Bitlang compiled symbol. Compiler-generated output should use one deterministic canonical spelling, and C backend name mangling must avoid collisions when emitting into C's case-sensitive identifier namespace.

String and character contents remain case-sensitive and are not subject to identifier normalization.

Compiler-generated identifiers are not required to be pleasant for humans to read. They may encode module/class ownership, scope, type, source information, or other data required for deterministic symbol identity.

## 4. Variables

Variable declarations use a C-like declaration shape, but canonical Bitlang types are retained.

```c
Int10x32 value;
Int10x32 count = 0;
Float10x32 ratio = 1.0;
```

Multiple declarations may use the same general C form where doing so does not introduce ambiguity.

```c
Int10x32 a, b, c;
```

Compiler output may choose to emit one declaration per variable for simpler analysis and rewriting.

### 4.1 Canonical numeric types

Numeric type representation is inherited from Bitlang rather than redefined by Bitlang compiled.

For numeric types carrying both radix and bit width, the canonical form is:

```text
<TypeName><Radix>x<BitWidth>
```

Examples:

```text
Int10x32
Uint10x32
Int2x32
Int16x64
Float10x32
Float10x64
```

The bit width and radix are explicit type information. Signedness is explicit for integer families. Bitlang compiled must not replace this information with target-dependent C types such as plain `int`, `long`, `float`, or `double`.

Source shorthand has already been resolved before compiler-generated Bitlang compiled is produced. Therefore source-level shorthand such as `int` is represented by its canonical Bitlang type, normally `Int10x32`, before this stage.

Different radix types are distinct. No C integer promotions, usual arithmetic conversions, or ordinary implicit numeric conversions are inherited. Widening, narrowing, signedness changes, radix changes, and integer/floating-point conversions must be explicit.

Bitlang's distinction between cast and pulse remains valid in Bitlang compiled. The compiled representation preserves the already-resolved conversion operation rather than asking a C backend to infer one.

Numeric overflow behavior is inherited from Bitlang and is an error by default unless an explicit operation specifies another policy.

Floating-point types follow the same inheritance rule as integer types. Their canonical Bitlang type and already-defined semantics are preserved through Bitlang compiled; `float`, `double`, and related C types are backend representation choices only.

## 5. Assignment

C-style assignment syntax is used.

```c
value = 10;
value += 2;
value -= 2;
value *= 2;
value /= 2;
value %= 2;
```

Compound assignment may be normalized into explicit assignment and operation form.

Compiler-generated compiled output is permitted to prefer the already-expanded form inherited from Bitlang Preprocessed rather than reintroducing source-level sugar.

## 6. Arithmetic and comparison operators

The following C-style operators are part of the baseline surface where the operand types support them:

```text
+  -  *  /  %
== != < <= > >=
```

Unary `+` and `-` use C-like syntax.

Bitwise and logical operators are also written in the C form:

```text
& | ^ ~
&& || !
<< >>
```

Operand types must already satisfy Bitlang's explicit compatibility rules. C's implicit integer promotions and target-dependent conversion rules do not apply.

Bitlang compiled must not depend on C's unspecified operand evaluation order. When side effects make order observable, lowering must sequence them explicitly using statements and compiler-generated temporaries before C emission. See [`semantic-lowering.md`](semantic-lowering.md).

Shift edge cases remain a separate Bitlang semantic decision and must not inherit C undefined or implementation-defined behavior accidentally.

## 7. Increment and decrement

C-style increment and decrement syntax may be accepted by a Bitlang compiled parser where defined for the type:

```c
i++;
++i;
i--;
--i;
```

However, Bitlang source and canonical preprocessing do not rely on increment/decrement value-expression semantics. Compiler-generated compiled output may normalize mutation into ordinary explicit arithmetic assignment and need not emit these forms.

## 8. Functions

Functions use a C-like declaration and definition form while retaining canonical Bitlang types.

```c
Int10x32 add(Int10x32 a, Int10x32 b) {
    return a + b;
}
```

A function without a return value uses `void`.

```c
void reset(void) {
}
```

Function prototypes use the same form:

```c
Int10x32 add(Int10x32 a, Int10x32 b);
```

High-level methods are lowered into ordinary functions with explicit receiver/object representation where needed.

```c
void Player_damage(struct Player* self, Int10x32 amount);
```

Captured functions/closures must use an explicit environment representation before backend emission. See [`semantic-lowering.md`](semantic-lowering.md).

## 9. Return

C-style return syntax is used.

```c
return value;
```

or:

```c
return;
```

for a `void` function.

## 10. Conditional branching

C-style `if`, `else if`, and `else` syntax is used.

```c
if (value > 0) {
    positive();
} else if (value < 0) {
    negative();
} else {
    zero();
}
```

## 11. switch

C-style `switch`, `case`, `default`, and `break` syntax is used where the controlling type is valid.

```c
switch (value) {
case 0:
    zero();
    break;
case 1:
    one();
    break;
default:
    other();
    break;
}
```

C-compatible fallthrough behavior may be reused unless a later Bitlang-specific restriction requires explicit fallthrough annotation or diagnostics.

## 12. Loops

### while

```c
while (condition) {
    work();
}
```

### do-while

```c
do {
    work();
} while (condition);
```

### for

```c
for (Int10x32 i = 0; i < count; i++) {
    work(i);
}
```

The compiler may normalize loop forms internally when performing optimization.

## 13. break and continue

C-style loop control is used.

```c
break;
continue;
```

## 14. struct

C-style structures are retained as a first-class low-level facility.

```c
struct Player {
    Int10x32 hp;
    Int10x32 mp;
};
```

Member access uses the C form:

```c
player.hp
player_ptr->hp
```

Bitlang high-level classes may be lowered into one or more `struct` definitions plus ordinary functions.

Nested struct values do not need to be flattened field-by-field. Struct preservation is an acceptable default lowering. Full flattening is an optimization only when layout, aliasing, and observable behavior are preserved.

Physical struct layout and alignment remain separate decisions when they are observable or cross an ABI boundary.

## 15. enum

C-style enumeration syntax is part of the baseline surface.

```c
enum State {
    STATE_IDLE,
    STATE_RUNNING,
    STATE_STOPPED
};
```

The deterministic underlying Bitlang numeric representation remains to be defined separately when enum range, storage, or ABI is observable.

## 16. Arrays

C-like fixed-size array syntax is used.

```c
Int10x32 values[16];
```

Indexing uses square brackets:

```c
values[index]
```

Multi-dimensional C-like syntax may be represented directly:

```c
Int10x32 matrix[4][4];
```

The out-of-bounds failure policy is Bitlang-specific and must not be inherited accidentally from unchecked C indexing.

## 17. Pointers and references

C-like pointer-shaped low-level representation may be used where appropriate.

Bitlang `Ptr<T>` and `Ref<T>` remain semantically distinct until their guarantees have been discharged by lowering.

`Ptr<T>` is the raw low-level pointer form. Its resolved semantics may permit null, reassignment, pointer arithmetic, and raw-address operations.

`Ref<T>` is the safe reference form. It is non-null, cannot participate in pointer arithmetic, cannot be rebound after binding, and must not outlive its referent.

A C backend may eventually represent both with pointer-shaped storage, but it must not reintroduce an operation that violates the already-validated `Ref<T>` guarantees.

The exact textual spelling used for Ptr versus Ref inside canonical compiled output remains a separate syntax decision.

Raw-pointer invalid-access/provenance behavior remains a separate semantic decision.

## 18. Function pointers

Function pointer representation may use C-compatible syntax where required for direct C translation.

```c
Int10x32 (*operation)(Int10x32, Int10x32);
```

Captured closures require an explicit environment in addition to a callable target and are not represented as a bare C function pointer unless no capture state is required.

## 19. typedef

A C-compatible `typedef` form may be used for low-level aliases.

```c
typedef Uint10x32 CounterType;
```

Aliases must not erase the resolved semantic type information required for validation or backend lowering.

## 20. const

`const` uses C-like placement and syntax where appropriate, but its Bitlang semantics are stronger than C `const`.

```c
const Int10x32 value = 10;
```

Bitlang `Const` means the semantic value remains fixed after initialization. For aggregates or objects, mutation through another alias is not permitted merely because the C backend type system could express such an alias.

C `const` may be emitted as part of backend representation, but the compiler must already have enforced the stronger Bitlang invariant. See [`semantic-lowering.md`](semantic-lowering.md).

## 21. static and storage-related declarations

When `static` represents Bitlang static retention, it means storage/lifetime retention and does not implicitly mean C internal linkage.

```c
static Int10x32 counter;
```

Linkage/export visibility is a separate semantic concern and must be lowered independently. A C backend must not hide a symbol solely because its Bitlang value has static lifetime.

## 22. Explicit casts

C-style explicit cast syntax is used as the baseline representation for a resolved cast operation:

```c
Int10x32 value = (Int10x32)ratio;
```

The legality and failure behavior of a cast follow Bitlang's explicit conversion rules rather than C's permissive conversion model.

No ordinary implicit conversion is performed merely to make an operation type-compatible.

Representation-changing pulse operations remain semantically distinct from casts even if a backend eventually implements both through generated conversion code.

## 23. sizeof

A C-compatible `sizeof` form may be retained syntactically:

```c
sizeof(Int10x32)
sizeof(value)
```

However, Bitlang semantic bit width may differ from backend storage/carrier width. Therefore the exact meaning of `sizeof` for types such as `Int10x24`, and its relationship to backend storage size and alignment, remains a Bitlang-specific decision. See [`open-decisions.md`](open-decisions.md).

## 24. C-compatible lowering principle

A valid Bitlang compiled construct should, where possible, translate to straightforward C without reconstructing lost high-level semantics.

For example, a high-level class method:

```text
player.damage(amount)
```

may be represented in Bitlang compiled approximately as:

```c
struct Player {
    Int10x32 hp;
};

void Player_damage(struct Player* self, Int10x32 amount) {
    self->hp -= amount;
}
```

The C backend then maps canonical Bitlang types and validated properties to C representations that preserve their semantics.

A defined Bitlang operation must not be translated into C in a way whose correctness depends on C undefined behavior. When direct C is insufficient, the backend must use checks, helpers, carrier representations, explicit control flow, or another defined lowering strategy.

The exact compiler-generated identifier names are an implementation concern and need not be optimized for human readability.

## 25. Property and state lowering

Bitlang Preprocessed contains the final resolved semantic property state. Bitlang compiled does not need to preserve every property name textually after its constraints have been validated and lowered.

Important inherited rules include:

- read, write, and reassignment capability remain independent during validation;
- ownership, borrow state, lifetime, copy/move capability, move state, release policy/capability/state, initialization, nullability, optionality, and const state must not be guessed from C defaults;
- `Auto_release` ownership must become explicit release behavior on applicable exit paths unless ownership was transferred or release is otherwise no longer required;
- `Moved`, `Released`, and `Uninitialized` states must not survive as ordinary valid reads/releases in emitted code;
- absence and null are separate, so `Optional nullable T` must preserve distinct absent, present-null, and present-non-null states where reachable;
- analysis-only metadata may be removed once the required low-level behavior and safety constraints have been established.

Detailed rules are in [`semantic-lowering.md`](semantic-lowering.md) and [`borrow-state-lowering.md`](borrow-state-lowering.md).

## 26. High-level construct elimination

Bitlang compiled is procedural and low-level. The C backend must not be responsible for reconstructing high-level language meaning.

Before backend emission:

- class methods become ordinary functions with explicit receiver representation;
- dynamic dispatch becomes explicit call targets/tables where required;
- closures become an explicit callable target plus explicit environment representation;
- pattern matching becomes ordinary control flow;
- sum/variant-like values receive an explicit low-level representation;
- unresolved source-level generic/currying/preprocessor syntax does not remain.

Module/class ownership needed for symbol identity is encoded into generated names or explicit symbol metadata.

## 27. Remaining Bitlang-specific decisions

The remaining decisions that cannot simply inherit C behavior are tracked in [`open-decisions.md`](open-decisions.md). The major unresolved areas are:

- semantic `sizeof` versus backend storage size and alignment,
- array bounds failure behavior,
- common runtime failure/trap model for dynamic semantic errors,
- shift edge cases,
- raw-pointer invalid-access, provenance, and unsafe-operation boundaries,
- observable struct layout/alignment and explicit packed layout,
- deterministic enum underlying representation,
- external C/native ABI and symbol contract,
- Bitlang string/character low-level representation,
- static/module initialization and destruction order,
- concurrency/atomic memory model,
- exact canonical Ptr/Ref textual representation,
- exact-layout / bit-field facility for protocols, hardware, and ABI-specific layouts.

Until one of these areas is explicitly defined, similarity to C syntax does not imply that C undefined or implementation-defined behavior becomes Bitlang semantics.
