# Moder Syntax Conventions

This document outlines the syntactic rules and naming conventions for the Moder codebase. Adherence to these rules ensures consistency, prevents namespace collisions, and maintains compatibility with standard C libraries.

## 1. Identifier Casing & Naming

| Category | Convention | Example |
| :--- | :--- | :--- |
| **Types** (Structs, Typedefs) | `snake_case` + `_t` suffix | `allocator_t` |
| **Enums** | `PascalCase` | `SlabType`|
| ** Enum constants | 'SCREAMING_SNAKE_CASE' | `SLAB_SMALL`|
| **Variables / Members** | `snake_case` | `new_allocator`, `page_count` |
| **Unions** | `uni_` prefix + `snake_case` | `uni_slab_data` |
| **Global Variables** | `gbl_` prefix + `snake_case` | `gbl_allocator` |
| **Pointers** | `snake_case` + `_ptr` suffix | `page_ptr`, `buffer_ptr` |
| **Functions** | `snake_case` (Optionally `md` prefix) | `create_page`, `md_malloc` |
| **Macros & Defines** | `SCREAMING_SNAKE_CASE` | `MAX_BUFFER_SIZE` |
| **Labels** (`goto`) | `snake_case` | `cleanup_fail:` |

## 2. Special Contexts

### Constants
*   **Local (Stack):** Follow standard variable naming (`snake_case`).
*   **Global/Static:** Follow macro naming (`SCREAMING_SNAKE_CASE`).

### Function Parameters
*   Use `snake_case`. 
*   If a parameter name shadows another identifier in the same scope, use the `pr_` prefix (e.g., `pr_page`).
*   never use md_ prefix with local and helper functions that the user will not interact with.

### Reserved Names (Strict)
To avoid conflicts with the C Standard Library and compiler internals:
*   **Never** start an identifier with a double underscore (`__name`).
*   **Never** start an identifier with a single underscore followed by an uppercase letter (`_Name`).

## 3. General Best Practices
*   **Prefixing:** Use the `md` prefix for public-facing functions to prevent collisions when the `MD_GUARD_DISABLE` flag is not active.
*   **Namespacing:** Ensure function names and variable names do not collide.
*   **Labels:** While `goto` is discouraged, labels must be unique within their containing function.
*   **pointer start**: Write the pointer start with the identifier not the type (i.e. `int *arr` not `int* arr`).

## 2. statements spacing

this may seem a minor, but it's too important to follow these rules:

- in if statements, follow this pattern:

```cpp
if (condition) {
    // do stuff here.
} else if (condition_2) {
    // do other stuff here.
} else {
    // do yet other stuff here.
}
```

- in for loops follow this pattern:

```cpp
for (init; condition; loop) {
    // do soo fancy stuff here.
}
```

- in while loops, follow this pattern:

```cpp
while (condition) {
    // do some stuff here.
}
```

- in do while loops, follow this pattern:
```cpp
do {
    // coffee is goated.
} while (condition);
```

- in switch statements, follow this pattern:
```cpp
switch (expression) {
    case CONST_1:
        // do beautiful stuff here.
        break; // don't forget the break or your code will break!
    case CONST_2:
        // do ugly stuff here.
        break; //
    default:
        // do freshman stuff here.
        break;
}
```

- not a statement, but follow this pattern in function definition:
```cpp
type identifier(paremter_type_1 parameter_identifier_1, ...) {
    // YDSSH (yet do some stuff here).
}
```

you should follow these rules including spaces and new line locations (e.g. the space between the if statement and conditiion parantheses, the { is in the same line as the for statement, the identation rules, etc..) to ensure a consistent codebase.
also always use braces even in single-line stataments, this is important for clarity, for example:
```cpp
if (condition) foo();
```
is forbidden, rather write:
```cpp
if (condition) {
    foo();
}
```

## Safe gurads

when making a new header file, it's standard to add a safe guard at the start of the file, follow this pattern:
```cpp
#ifndef MODER_{FILE_NAME}_H
#define MODER_{FILE_NAME_H

// actual code here.

#endif
```

replace {FILE_NAME} with the actual header file name.
