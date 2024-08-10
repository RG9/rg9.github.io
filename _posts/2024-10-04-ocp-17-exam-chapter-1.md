---
title: OCP 17 Exam - chapter 1 notes (syntax, types)
categories: [ Book notes]
tags: [ ocp17, java ]
---

- source file must have at most one public type
- **import** that specifies class name takes precedence over import with `*` if there are two classes with the same name
  - e.g. `java.util.Date` vs. `java.sql.*`, which also provides `Date` class
- **types** - `boolean` size is not specified, depends on JVM
    - byte | 8 bit | -128 | 127
    - short | 16 bit | -32,768 | 32,767
    - int | 32 bit | -2,147,483,648 | 2,147,483,647
    - long | 64 bit | -2^63 | 2^63 - 1
    - float | 32 bit | n/a
    - double | 64 bit | n/a
    - char | 16 bit | 0 | 65,535
- **string text block** - if you put ending `"""` in new line, it will result in blank line
    - `\` - joins lines - line break is ignored
    - `\s` - add extra two spaces at the end of line
- **underscore** in number literals can be used only inside, we cannot start or end number with `_` or use near `.`
    - correct: `1___1_1.1___1_1`
- **variable name** must start with letter, `_`, or currency symbol like `$`, `€`, `¥`
- **declaration of multiple variables** in the same line is only possible for the same type, except `var`!
    - e.g. `int v1 = 1, v2, v3 = 3;`
- you can **declare array** in two ways: `int[] arrayName` or `int arrayName[]`
- `var var = "var"` is allowed because `var` is not reserved word, just reserved type!
- curly braces could be used inside class or method and they define new scope
    - when used in class it is called
- "identifying blocks and variable scope is **second nature of the exam**" ;)
- **object vs. references** - "all references are the same size, but object can have different size"

> it might be tricky but `var name = (String)null;` is OK - compiles and doesn't throw any exception!
{: .prompt-info }


# Fundamentals exam notes from practise attempt:

- **cast**: if we assign smaller type size to larger, e.g. `int` to `long` or `byte` to `short` then we don't need cast.
    - `short` to `char` and vice-versa (both 16-bit) also needs a cast
- "**functional interface** must have exactly one abstract method and may have other default or static methods"
- "**break** w/o a label can occur only in a switch, while, do, or for statement"
- "**interfaces** allow multiple implementation **inheritance** through default methods" - Java doesn't allow to extend
  from multiple classes
- `trim` is method of `String`, not `StringBuilder`

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
