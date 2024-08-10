---
title: OCP 17 Exam - chapter 7 notes - part 2 (sealed and records)
categories: [ Book notes ]
tags: [ ocp17, java ]
---

## Sealed classes and interfaces

- sealed classes allow us to implement "enum" on class level, so it can be used in switch statement:

```java
      Sealed v = new Chapter7_SealedClasses().new MyClass();
      // note: use "java --enable-preview --source 17" to enable patterns in switch statements
      switch(v) {
        // case SubClass -> System.out.println("Subclass"); error: type pattern expected
        case SubClass s -> System.out.println("Subclass");
        default ->  System.out.println("default"); // without default, there is error: the switch statement does not cover all possible input values
        case MyClass m -> System.out.println("MyClass");
      }
```
- "sealed class must have subclasses", following is not permitted:

```java
    sealed class Sealed { }
```
- class that extend sealed class must have one of the following modifiers: `final`, `sealed`, `non-sealed`

```java
    sealed class Sealed permits SubClass { }
    class SubClass extends Sealed { } // error: sealed, non-sealed or final modifiers expected
```
- class mentioned in `permits` must extend sealed class

```java
    sealed class Sealed permits SubClass { }
    final class SubClass { } // DOES NOT COMPILE: subclass Chapter7_SealedClasses.SubClass must extend sealed class
```

> sealed class can be indirectly extended when extending `non-sealed` subclass (review Q.14)
{: .prompt-tip }


## Records 
- record cannot declare static field with **the same name** as instance field
- 
```java
record SameName(String name) { static int name = 0; } // DOES NOT COMPILE: record component name is already defined in record SameName
```

## Nested classes

> exam may trick us by extending `final` class, even using anonymous class! (review Q.15)
{: .prompt-tip }

## Polymorphism

> encapsulation allows both getters and setters as the goal is to only restrict direct access to variable (review Q.12)
{: .prompt-tip }

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter7/Chapter7_SealedClasses.java>
