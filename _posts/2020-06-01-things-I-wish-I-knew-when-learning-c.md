---
layout: post
title: "Things I Wish I Knew When Learning C"
description: "Some interesting aspects of C that I wish I knew, such as The Clockwise/Spiral rule, Structure padding, Anonymous struct / union, X Macros and many others."
---

## The Clockwise/Spiral rule

C type declaration can be intimidating sometimes, such as:

```c
void (*signal(int, void (*fp)(int)))(int);
```

But with the Clockwise/Spiral rule one can easily read any type declaration.

See this wonderful explanation [here](http://c-faq.com/decl/spiral.anderson.html).

---

## Version detection

* `__STDC__` C90 defines this macro whose value is 1 indication that the compiler fully implements Standard C.
* `__STDC_VERSION__` C99 defines this macro whose value indicates which standard the compiler supports.

```c
#if __STDC_VERSION__ == 201710L
  /* it's C18 */
#elif __STDC_VERSION__ == 201112L
  /* it's C11 */
#elif __STDC_VERSION__ == 199901L
  /* it's C99 */
#elif __STDC__ == 1
  /* it's C90 */
#else
  /* it's not standard conforming */
#endif  
```

---

## Boolean

C90 had no build-in boolean type. C99 added a boolean type `_Bool` which now is a keyword. `bool` is just an alias for `_Bool`, and to use these types you must include `<stdbool.h>`.

---

## Exact-width integer

`<stdint.h>` defines many useful integer types. For example `intN_t` and `uintN_t`:

* `int8_t` is the signed integer that occupies **exactly** 8 bits.
* `uint32_t` is the unsigned integer that occupies **exactly** 32 bits;

However these exact-width integer types are optional, meaning they may be absent from the header.

---

## Anonymous struct / union

Anonymous struct are useful when you don't want to name your struct/union inside another struct/union. For example, when implementing a dynamically typed language, you need a type to represent all the values in your language, an easy way to do so is to use a enum and a anonymous union inside your struct.

```c
typedef struct {
  TypeEnum type;
  union {
    int i;
    double d;
    char *c;  
  };
} Value;

Value v;
// ...
if (v.type == TYPE_DOUBLE)
  do_thing_with_double(v.d);   // <=== here 
// else ...
```

---

## Structure padding

```c
typedef struct {
  char c;
  int i;
} foo;

printf("%d", sizeof(foo));
```

It'll print 8 instead of 5 because a padding of 3 bytes are added between c and i. It means that the ordering inside your structure will have an impact on the overall size. One must also remember not to use `memcmp` on nonflat structures.

---

## Compound literals

In C90, if you want a fixed value of a structure type, you have to create a named constant object.

```c
typedef struct {
  int a;
  int b;  
} foo;

foo const f = {1, 2};
```

In C99, you can use a compound literal to represent a fixed value of a structure type.

```
do_things_with_foo((foo){1, 2});

// even with arrays, and you don't have to specify the size of the array.
do_things_with_array((int[]){1, 2, 3, 4, 5}); 
```
 
---

## Designated initializer

```c
typedef union {
  int i;
  double d;
} foo;

foo f1 = { .i = 1 };
foo f2 = { .d = 3.2 };

// with arrays
int x[10] = { 0, 0, 0, 2, 0, 0, 0, 1};
int x[10] = { [3] = 2, [7] = 1 };
```

---

## Flexible array member

In C99, the last member of a structure can be an array with unspecified dimension.

```c
typedef struct {
  int a;
  char b[];   // <=== flexible array member
} foo;
```

The benefits are that you can now allocate the entire structure with only one call, the same with `free`.

```c
int size = 10;
foo f = malloc(sizeof(foo) + size * sizeof(char));
// ...
free(f);   // no need to free(f.b);
```

---

## intprt_t / uintptr_t

These two types are capable of holding pointers to objects (not pointers to functions). Use them when you want to do arithmetic operations on pointers other than `+` or `-`. Now you must wondering "why? why would I do that?", well for example, in the `Lua` languages implementation all values are represented using double precision number which is a technique call `NAN-tagging`. Here is the code for my toy languages implementation using `NAN-tagging`.

```c
typedef uint64_t Value;  // you can use double here, only uint64_t is more convenient.

typedef union {
  double num;
  Value bits;
} DoubleBits;

#define QNAN         ((uint64_t)0x7ffc000000000000)
#define SIGN_BIT     ((uint64_t)0x8000000000000000)
#define IS_NUM(v)    (((v) & QNAN) != QNAN)
#define NUM_VAL(n)   ((DoubleBits){ .num = n }.bits)
#define AS_NUM(v)    ((DoubleBits){ .bits = v }.num)
#define IS_OBJ(v)    (((v) & (QNAN | SIGN_BIT)) == (QNAN | SIGN_BIT))
#define OBJ_VAL(o)   ((Value)(uintptr_t)(o) | QNAN | SIGN_BIT)
#define AS_OBJ(v)    ((Obj*)(uintptr_t)((v) & ~(QNAN | SIGN_BIT)))

double a = 23;
printf("%d", AS_NUM(NUM_VAL(a)) == a); // prints 1;
// same with Obj.
```

---

## Struct inheritance

Since C guarantees that no padding will be added before the first member of a structure which means that the pointer points to the whole structure also points to the first member of the structure. One can write some OOP code using this technique.

```c
typedef struct {
  // ...
} Animal;

typedef struct {
  Animal base;
  // ...
} Dog;

typedef struct {
  Animal base;
  // ...
} Cat;

// now if you have a dog or cat, you can safely cast it to animal.
Animal *a = (Animal*)dog;
// And cast it back.
Dog *d = (Dog*)a;
```

---

## _Generic

It's like `switch`, but on types:

```c
#define acos(X) _Generic((X), \
    long double complex: cacosl, \
    double complex: cacos, \
    float complex: cacosf, \
    long double: acosl, \
    float: acosf, \
    default: acos \
    )(X)
```

However I haven't found an elegant solution for multiparameter functions.

---

## Labels as values

To write direct threaded code(computed goto), GUN labels as values extension is way to go. Especially useful for interpreter/compiler writers, you know what I'm saying right? TLDR: it's a faster version of switch statement.

```c
int *ip;
static void *dispatch_table[] = { &&add, &&subtract, ... };

void interpret() {  
  goto *dispatch_table[*ip++];
add:
  int b = pop();
  int a = pop();
  push(a + b);
  goto *dispatch_table[*ip++];
subtract:
  int b = pop();
  int a = pop(); 
  push(a - b);
  goto *dispatch_table[*ip++];
//...
}
```

---

## Statements as expressions

This is also a GUN extension which gives you the ability to evaluate statements as expressions.

```c
int a = ({int b = 3; b * 2;});  // a => 6
```

---

## Dynamic array

Normally you just define a structure with length, capacity and the array you want, but here is an implementation which doesn't need to define a new type. The idea is to store the length at raw[0] and capacity at raw[1], then use raw[2] as the dynamic array.

```c
#define ARRAY_NEW(arr) \
  do { \
    size_t *raw = malloc(2 * sizeof(size_t)); \
    raw[0] = 0; \
    raw[1] = 0; \
    arr = (void*)&raw[2]; \
  } while (false)

#define ARRAY_SIZE(arr) *((size_t*)(arr) - 2)

#define ARRAY_FREE(arr) \
  do { \
    size_t *raw = ((size_t*)(arr) - 2); \
    free(raw); \
    arr = NULL; \
  } while (false)

#define ARRAY_PUSH(arr, v) \
  do { \
    size_t *raw = ((size_t*)(arr) - 2); \
    if (raw[0] + 1 > raw[1]) { \
      raw[1] = (raw[1] == 0) ? 8 : raw[1] * 2; \
      raw = realloc(raw, 2 * sizeof(size_t) + raw[1] * sizeof(v)); \
      arr = (void*)&raw[2]; \
    } \
    arr[raw[0]] = (v); \
    raw[0] += 1; \
  } while (false)


int *c;
ARRAY_NEW(c);
ARRAY_PUSH(c, 1);
printf("%d\n", c[0]);  // print 1;
ARRAY_FREE(c);
```

---

## X macros

Sometimes you need to manage parallel list of data, for example in the above example with the interpreter, you have a list of bytecodes, and a jump table list(for computed goto), instead of hand written the jump table, X macros can generate them for you.

```c
// "opcode.h"
// this is the header file for all the bytecodes.
OPCODE(add)
OPCODE(subtract)
// ...
```

```c
int *ip;
static void *dispatch_table[] = {
  #define OPCODE(name) &&op_##name,
  #include "opcode.h"
  #undef OPCODE
};
void interpret() {
  goto *dispatch_table[*ip++];
op_add:
  // ...
op_subtract:
  //...
}
```