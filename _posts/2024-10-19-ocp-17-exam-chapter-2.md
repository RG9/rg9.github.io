---
title: OCP 17 Exam - chapter 2 notes (numeric promotions, operators)
categories: [ Book notes ]
tags: [ ocp17, java ]
---

- **unary** = 1 param, **binary** = 2 params, **ternary** = 3 params
- **numeric promotions rules**:
    - always to larger data type, e.g. `short` -> `int`
    - integral type is promoted to floating point if used together,
        - e.g. `var z = (double) 1.0 + (short) 1` - result is `double`
    - smaller types `byte`, `short`, `char` are promoted to `int` if used in arithmetic ops (`+`, `*`, etc.),
        - e.g. `short s = 1 + (short)(1 * 2)` - doesn't compile, because `1+` implicitly casts to `int`
- `float y = 2.1` doesn't compile, because `f` is required!
- `*=` automatically casts to smaller type,
    - e.g.`long l = 1; int i = 2; i=i*l` - doesn't compile, but `long l = 1; int i = 2; i*=l` does.
- **assignment also returns value**, examples:
    - `long coyote = (wolf = 3)` assigns `3` to `wolf` and `coyote`
    - `if(healthy = true)`, assigns `true` to `healthy` and satisfies `if` statement
- **equality operators** can be used only for the same type,
    - e.g. `true == 3` or `"3" == 3` doesn't compile
- `instanceof` is also limited to the same type or superclass or interited type
    - e.g. `Number num = 3; boolean b = num instanceof String;` - doesn't compile, but
      `b = num instanceof java.util.concurrent.atomic.AtomicInteger;` does
- `null instanceof String` always returns `false`
- **XOR** truth table:

```
XOR true ^ true = **false**
XOR false ^ false = false
XOR true ^ false = true
XOR false ^ true = true
```

- `&` always evaluates both operands, where `&&` evaluates right, only when left is true (similar rule applies to `|`
  and `||`)
    - e.g. `int i = 6; boolean b = (i >= 6) || (++i <= 7)` - after executing `i=6`, because left of `||` was true, so no
      need to evaluate right side (**unperformed side effect**)

# Playground code     
<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter2.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
