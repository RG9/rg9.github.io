---
title: OCP 17 Exam - chapter 7 notes - part 1 (interfaces and enums)
categories: [ Book notes ]
tags: [ ocp17, java ]
---

## Interfaces
- `abstract` is implicit in interface - automatically added by compiler

```java
     public abstract interface AbstractIsImplicitInInterface { // abstract is implicit (added by compiler)
	        public static final String MY_CONSTANT = "ABC"; // "public static final" is implicit
        public abstract void myMethod(); // public and abstract is implicit
     }
```

- thus interface cannot be `final` as `abstract` class

```java
final interface FinalInterface {  } // DOES NOT COMPILE: illegal combination of modifiers: interface and final
```

- and like in `abstract` class, all `abstract` methods must be implemented by subtype
- we can implement two interface that have methods with the same name based on rules of overriding from Chapter 6 (see [[2025-01-18-ocp-17-exam-chapter-6]])
* `interface` allows  only `private` or `public static` or `public default` method with body (`public` is implicit)

```java
     interface InterfaceAllowedVisibilityModifiers {
       private static void privateStaticMethodWithBody() { }
       // protected static void staticMethodWithBody() { } DOES NOT COMPILE:  error: modifier protected not allowed here
       static void staticMethodWithBody() { } // implicitly "public"

       private void methodWithBody(){}
       // protected void methodWithBody(){} DOES NOT COMPILE: interface abstract methods cannot have body
       // void methodWithBody(){} // DOES NOT COMPILE: interface abstract methods cannot have body
       default void defaultMethodWithBody(){}
     }
     
```

*  tricky difference between `interface` and `abstact class` is that **subtype** of `interface` must always provide `public` modifier for implemented method (review Q.7)

```java
     static abstract class SomeAbstractClass {
        abstract void methodToImplement();
     }
     
     interface SomeInterface {
        void methodToImplement();
     }
    // ...
      new SomeAbstractClass() {
         void methodToImplement() { } 
      };
      new SomeInterface() {
         // void methodToImplement() { } // DOES NOT COMPILE: attempting to assign weaker access privileges; was public
         public void methodToImplement() { }
      };
```

> exam may trick us by using interface with `extends` instead `implements` keyword (review Q.6)
{: .prompt-tip }

- class inherits from **two or more interfaces with default methods with the same signature**
	- subclass must implement/override `default` method, otherwise DOES NOT COMPILE, because compiler doesn't know which default to use
	- when we want to use/call `default` method implementation from Interface, we should use `InterfaceName.super.methodName();` syntax. (review Q.23)
  
 ```java
      interface SameDefaultSignature1 {
        default int getInt() { return 1; }
     }
     
     interface SameDefaultSignature2 {
        default int getInt() { return 2; }
     }
     
     // DOES NOT COMPILE:   class MustOverrideDefault inherits unrelated defaults for getInt() from types SameDefaultSignature1 and SameDefaultSignature2
     // static class MustOverrideDefault implements SameDefaultSignature1, SameDefaultSignature2 { } 

     static class MustOverrideDefault implements SameDefaultSignature1, SameDefaultSignature2 {
        public int getInt() {
          return SameDefaultSignature1.super.getInt() + SameDefaultSignature2.super.getInt(); // prints 3
        }
     }
```

## Enums
* "fixed set of constants"
* advantage over `int` or `String` constants is that providing invalid value of **enum** would fail compilation (type-safe checking)
* "simple enum"

```java
     enum Seasons {
      WINTER, SPRING, SUMMER, AUTUMN
      , // allowed, but optional
      // , // another one DOES NOT COMPILE
      ; // optional
     }
```

> remember that Enum must have private constructor (or w/o access modifier because is implicitly private) (review Q.26)
{: .prompt-tip }
```java
     enum EnumWithPublicConstructor {
	  ONE(1), TWO("two");
      // public EnumWithPublicConstructor(int val){ } // compilation error: modifier public not allowed here
        EnumWithPublicConstructor(int val){ System.out.println(val); }  // constructor is implicitly private
        private EnumWithPublicConstructor(Object val){ System.out.println(val);  }
      }
```

* Enum cannot be extended by another enum
* Enum's `valueOf(String)`

```java
       System.out.println(Seasons.valueOf("SPRING")); // prints SPRING
       // System.out.println(Seasons.valueOf("Spring")); // IllegalArgumentException: No enum constant Chapter7.Seasons.Spring
```
* must be used in switch without type prefix

```java
       var season = Seasons.WINTER;
       switch (season) {
          // case Seasons.WINTER -> { System.out.println("winter!"); } // DOES NOT COMPILE:  an enum switch case label must be the unqualified name of an enumeration constant
          case WINTER -> { System.out.println("winter!"); } 
          default -> { System.out.println("other season"); }
       }
```

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter7/Chapter7_Interfaces.java>

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter7/Chapter7_Enums.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
