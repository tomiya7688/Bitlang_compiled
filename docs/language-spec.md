# Bitlang compiled Language Specification

Status: Draft

Bitlang compiled is a C-like low-level language positioned between `Bitlang preprocessed` and backend representations such as C.

The baseline rule is simple: when ordinary C syntax and semantics can be reused without conflicting with Bitlang compiled requirements, they are reused directly.

This document initially records the C-compatible surface. Bitlang compiled-specific restrictions and extensions are added explicitly as they are decided.

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

## 2. Comments

C-style comments are accepted.

```c
// line comment

/* block comment */
```

## 3. Identifiers

Identifiers use the ordinary C-like form: letters, digits, and `_`, with the first character not being a digit.

```c
value
player_hp
_temp2
```

Compiler-generated identifiers are not required to be pleasant for humans to read. They may encode ownership, scope, type, or source information as necessary.

The exact identifier normalization and case rules for Bitlang compiled remain a Bitlang compiled-specific specification item and are not inherited blindly from C.

## 4. Variables

C-style variable declaration syntax is used where applicable.

```c
int value;
int count = 0;
float ratio = 1.0;
```

Multiple declarations may use the same general C form where doing so does not introduce ambiguity.

```c
int a, b, c;
```

Compiler output may choose to emit one declaration per variable for simpler analysis and rewriting.

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

Compound assignment is syntactic sugar for a lower-level assignment operation and may be normalized by the compiler.

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

Exact type-conversion, overflow, shift, and evaluation rules are Bitlang compiled-specific semantics and will be defined explicitly rather than inheriting every C undefined or implementation-defined behavior.

## 7. Increment and decrement

C-style increment and decrement syntax is accepted where defined for the type:

```c
i++;
++i;
i--;
--i;
```

The compiler may normalize these into ordinary arithmetic assignments during lowering or optimization.

## 8. Functions

Functions use a C-like declaration and definition form.

```c
int add(int a, int b) {
    return a + b;
}
```

A function without a return value uses `void`.

```c
void reset(void) {
}
```

Function prototypes use the C form:

```c
int add(int a, int b);
```

High-level concepts such as methods are expected to be lowered before or while producing Bitlang compiled. For example, an instance method may become an ordinary function with an explicit object pointer.

```c
void Player_damage(struct Player* self, int amount);
```

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

Fallthrough behavior and any restrictions on it will be defined explicitly in the Bitlang compiled semantic rules.

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
for (int i = 0; i < count; i++) {
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
    int hp;
    int mp;
};
```

Member access uses the C form:

```c
player.hp
player_ptr->hp
```

Bitlang high-level classes may be lowered into one or more `struct` definitions plus ordinary functions.

## 15. enum

C-style enumeration syntax is part of the baseline surface.

```c
enum State {
    STATE_IDLE,
    STATE_RUNNING,
    STATE_STOPPED
};
```

The exact underlying representation and whether it must always be explicit will be defined separately.

## 16. Arrays

C-like fixed-size array syntax is used.

```c
int values[16];
```

Indexing uses square brackets:

```c
values[index]
```

Multi-dimensional C-like syntax may be represented directly:

```c
int matrix[4][4];
```

Bounds semantics are not assumed to be identical to C and will be specified explicitly.

## 17. Pointers

Pointer syntax follows C where pointers are allowed.

```c
int* ptr;
struct Player* player;
```

Address-of and dereference use the C forms:

```c
ptr = &value;
value = *ptr;
```

Pointer arithmetic, lifetime rules, invalid pointer behavior, aliasing, and other memory-safety semantics are intentionally not inherited wholesale from C. These are core Bitlang compiled-specific specification areas.

## 18. Function pointers

Function pointer representation may use C-compatible syntax where required for direct C translation.

```c
int (*operation)(int, int);
```

The precise allowed forms will be restricted as necessary to keep parsing, analysis, and backend translation deterministic.

## 19. typedef

A C-compatible `typedef` form may be used for low-level aliases.

```c
typedef unsigned int uint;
```

Bitlang compiled may later place stricter limits on aliases to ensure that the resolved underlying type remains easy to inspect.

## 20. const

`const` uses C-like placement and syntax where appropriate.

```c
const int value = 10;
const int* ptr;
int* const ptr2 = other;
```

Exact mutability semantics are defined by Bitlang compiled, not merely delegated to a C compiler.

## 21. static and storage-related declarations

C-style `static` syntax is available as a baseline representation for internal storage duration or linkage concepts.

```c
static int counter;
static void helper(void) {
}
```

The complete linkage model will be specified separately.

## 22. Explicit casts

C-style explicit cast syntax is used as the baseline:

```c
int value = (int)ratio;
```

Bitlang compiled is expected to be stricter than C regarding which casts are legal. High-level implicit conversions should normally already be resolved before this stage.

## 23. sizeof

A C-compatible `sizeof` form is retained where useful for low-level representation and C translation.

```c
sizeof(int)
sizeof(value)
```

Whether all `sizeof` expressions must be compile-time resolvable will be defined separately.

## 24. C-compatible lowering principle

A valid Bitlang compiled construct should, where possible, translate to straightforward C without reconstructing lost high-level semantics.

For example, a high-level class method:

```text
player.damage(amount)
```

may be represented in Bitlang compiled approximately as:

```c
struct Player {
    int hp;
};

void Player_damage(struct Player* self, int amount) {
    self->hp -= amount;
}
```

The exact compiler-generated identifier names are an implementation concern and need not be optimized for human readability.

## 25. Intentionally unresolved C differences

The following areas must be specified by Bitlang compiled itself and must not simply inherit C behavior by accident:

- primitive type sizes and signedness,
- integer overflow,
- floating-point guarantees,
- implicit conversions and promotions,
- expression evaluation order,
- pointer arithmetic,
- pointer lifetime and ownership,
- aliasing,
- null representation and null access,
- array bounds behavior,
- struct layout and alignment,
- enum representation,
- linkage and symbol visibility,
- undefined and implementation-defined behavior,
- volatile semantics,
- atomic operations and concurrency,
- preprocessor availability,
- unions,
- bit-fields,
- variable length arrays,
- `goto`,
- variadic functions,
- C ABI interoperability.

Until these areas are explicitly defined, similarity to C syntax does not imply identical C semantics.
