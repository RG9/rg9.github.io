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
- class mentioned in `permits` must extend sealed class (review Q.30)

```java
    sealed class Sealed permits SubClass { }
    final class SubClass { } // DOES NOT COMPILE: subclass Chapter7_SealedClasses.SubClass must extend sealed class
```

> `permits` is not required for classes in the same file! (review Q.30)
{: .prompt-tip }

> sealed class can be indirectly extended when extending `non-sealed` subclass (review Q.14)
{: .prompt-tip }


## Records 
- record cannot declare static field with **the same name** as instance field
- 
```java
record SameName(String name) { static int name = 0; } // DOES NOT COMPILE: record component name is already defined in record SameName
```

- we can define compact constructor in order to validate parameters w/o need to repeating all parameters in signature
```java
    record CompactConstructor(int i, String s) { 
      CompactConstructor {  
       i=2; s="b"; // can modify implicit constructor params
        // this.i = 3; // but not record field - DOES NOT COMPILE: error: cannot assign a value to final variable i
      }
    }
```

- overloaded constructor must first call default one (review Q.21) 
```java
    record CustomConstructor(int i, String s) { 
      CustomConstructor() {  
        this(23, "custom"); // must invoke default all args constructor, otherwise DOES NOT COMPILE: error: constructor is not canonical, so its first statement must invoke another constructor of class CustomConstructor
      }
    }
```

## Nested classes

> Types:
> - Inner class: A non-static type defined at the member level of a class
> - Static nested class: A static type defined at the member level of a class
> - Local class: A class defined within a method body
> - Anonymous class: A special case of a local class that does not have a name

> exam may trick us by extending `final` class, even using anonymous class! (review Q.15)
{: .prompt-tip }

> be aware that static nested class doesn't have access to non-static members of parent class! Might be tricky! (review Q.16)
{: .prompt-tip }

## Polymorphism

> "Polymorphism is property of object to have many forms"

> encapsulation allows both getters and setters as the goal is to only restrict direct access to variable (review Q.12)
{: .prompt-tip }

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter7/Chapter7_SealedClasses.java>

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter7/Chapter7_Records.java>

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter7/Chapter7_Polymorphism.java>


----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
