---
title: OCP 17 Exam - chapter 3 notes (switch, loops, pattern matching, flow scoping)
categories: [ Book notes ]
tags: [ ocp17, java ]
---

- "watch indentation and braces", e.g. in the following example count is always incremented

```
if (count < 2)
	System.out.println("Good day");
    count++;
```

- "**pattern matching** was introduced to reduce boilerplate code, ... like a lot of other new enhancements in Java"
- **reassigning pattern variables** is possible (can be prevented with `final`)

```
if (count instanceof final Integer i)
    i = 5; // DOES NOT COMPILE
```

- pattern variable must be **subtype**

```
Double d = 2.0;
if(d instance of Double)
if(d instance of Double data) // DOES NOT COMPILE, but compiles in JAVA 21
```

- **flow scoping** - "scope is determined by compiler, not hierarchy like class or local scoping"

```
private static flowScopingShowcase(Number num) {
    if(!(number instance Integer integer))
        return
    System.out.println("it's integer " + integer.intValue());
}
```

- **switch statement**
    - `case constantExpresion: { System.out.print("value"); }`
    - **break** is optional. When `case` is matched and there will be no `break;`, then all
      subsequent `case`s will be evaluated as well.
    - **default** can be placed anywhere - doesn't have to be at the end
    - we can combine `case` in two ways: `case 1: case 2:` or `case 1,2:` (since Java 14)
    - supported types: `int`, `byte`, `short`, `char`, `String`, `enum` and `var` (if resolved to mentioned types)
    - if `case` statement is defined locally, then must be `final`
    -
- **switch expressions**
    - `case constantExpresion -> { yield "value"; }`
    - assignment to variable is optional
    - `default` is only required if assigned to variable and not all `cases` were covered
    - if assigned to variable it must end with `;`
    - block with `}` cannot end with `;` (for switch statement it can!)
- **loops**
  - we can use `var` in for-each loop: `for(var name : names)`
  - statements after `break;` DOES NOT COMPILE
  - `break LABEL;` will go to expression stating with `LABEL: `
  - **tip on exam** is to SKIP too complex loops, because I won't get extra points for solving this!

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter3.java>


----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
