---
title: OCP 17 Exam - chapter 4 notes (String, equals)
categories: [ Book notes ]
tags: [ ocp17, java ]
---

- be aware that exam want to trick us on concatenation of `String` and numbers, e.g. prints `66`

```
      int five = 5;
      String six = "6";
      System.out.println(1 + five + six);
```

- `String#substring` accepts numbers/indexes from `0` to length of String, e.g.

```
var text = "abcdef";
System.out.println(text.substring(0, 6)); // prints abcdef
System.out.println(text.substring(6, 6)); // prints ""
// System.out.println(text.substring(0, 7)); // throws java.lang.StringIndexOutOfBoundsException
```

- `equals` and `hashcode` is not required on exam, but be aware that if `equals` is implemented then both should be
  consistent to avoid collisions in "hash" based collections
- `String#contains` is just convenience method instead `string.indexOf(otherString) != -1`
- `String#trim` removes only ` \t\n\r`, but `String#strip` removes also Unicode whitespace, e.g. `\u2000`
- `String#ident` ... TODO

> Blog post not finished
{: .prompt-warning }

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter4.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
