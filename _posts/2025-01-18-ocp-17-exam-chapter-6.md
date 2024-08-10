---
title: OCP 17 Exam - chapter 6 notes (inheritance, abstract classes, immutables)
categories: [ Book notes ]
tags: [ ocp17, java ]
---

> my protip: **always start reading code from `main` method**
{: .prompt-tip }

## Inheritance 

- synonyms of `subtype` when working with classes: `subclass`, `child class`
 
- synonyms of `supertype` when working with classes: `superclass`, `parent class`

- book tip: "pay attention to `final` classes. If other class extend it, then DOES NOT COMPILE"

- Java forbids multiple inheritance because it is hard to tell which method should be inherited when both superclasses
  have method with the same signature - "there is no natural ordering for parents"
    - "Multiple inheritance is the property of a class to have multiple direct superclasses" - correct but forbidden in
      Java

- be aware of tricky question "does all variables inherit `java.lang.Object`?" No, primitives not.

- From Chapter 1: "`.java` file can have at most one top-level class"

- `protected` or `private` top-level class DOES NOT COMPILE

- be aware of hiding of instance variables by method parameters:

```java
  class HiddenInstanceVariable {
     int num;
     void setNum(int num){
        num = num; // should be "this.num = num" to assign instance variable
     }
  }
  var hiddenInstanceVariable = new HiddenInstanceVariable();
  hiddenInstanceVariable.setNum(5);
  System.out.println(hiddenInstanceVariable.num); // prints 0 instead 5
```

- be aware of hiding inherited variable:

```java
    class HiddenInheritedVariable extends HiddenInstanceVariable {
       int num; // hidden, because same type and name as inherited - comment to unhide
       void setNum(int num){
          this.num = num;
       }
       void setSuperNum(int num){
          super.num = num;
       }
    }
```

- note: variable with the same name can be defined as instance variable and inside method !!
```java
    private int abc = 1;
    static void static_method() {
        int abc = 1;
    }
    void instance_method() {
        int abc = 1;
    }
    void instance_method(int abc) {
    }
```

- constructor name must be capitalized!! Easy to overlook on exam.

```java
    static class LowerCaseConstructorDoesNoCompile {
       // public lowerCaseConstructorDoesNoCompile() {} // DOES NOT COMPILE: error: invalid method declaration; return type required
        public LowerCaseConstructorDoesNoCompile() {} // ok
    }
```

> some of this might look obvious, but we should train our eye before exam ;)
{: .prompt-info }

- "compiler only inserts default constructor when no constructor is defined"

- compiler also inserts `super()` automatically if no explicit `this()` or `super()`

- no line is allowed before `this()` or `super()`

- be aware that cyclic constructor invocation is detected on compilation and DOES NOT COMPILE

```java
   static class ConstructorCicleDoesNotCompile {
      ConstructorCicleDoesNotCompile(){ 
          //this(5); // DOES NOT COMPILE error: recursive constructor invocation
      }
      ConstructorCicleDoesNotCompile(int num){
          this(); 
      }
   }
```

- be aware that if superclass defines constructor with arguments, then subclass must also define such constructor with
  `super(..)` call

- be aware of `final` instance variables on exam -> it can be only assigned once directly in declaration, via
  initializer or constructor


- **class initialization order**:
    - superclass `static` variables and `static` initializers (in order of declaration within class)
    - subclass `static` variables and `static` initializers (in order of declaration within class)
    - subclass constructor must call superclass constructor, so order is following:
        - superclass instance variables and instance initializers (in order of declaration within class)
        - superclass constructor body
        - subclass instance variables and instance initializers (in order of declaration within class)
        - subclass constructor body (be aware of default `super()` added by compiler)
      
```java
   static class InitializationOrderSuperClass {
      static { System.out.println("static superclass init"); }
      { System.out.println("superclass init"); }
      InitializationOrderSuperClass() {  System.out.println("superclass constructor"); }
   }
   
   static class InitializationOrderSubClass extends InitializationOrderSuperClass {
      static { System.out.println("static subclass init"); }
      { System.out.println("subclass init"); }
      InitializationOrderSubClass() {  System.out.println("subclass constructor"); }
   }

//prints   
//static superclass init
//static subclass init
//superclass init
//superclass constructor
//subclass init
//subclass constructor
```

- "**polymorphism** is the ability of an object to take on many different forms"

## Rules of overriding methods
- rules of **overriding methods** and **hiding static methods** and implementing `abstract` methods:
    - same name and parameter types (no covariant types here!)
    - visibility can be broader, e.g. super method has `protected`, so overridden can have `public`
    - declared thrown `Exception` should be the same or subtype (if no checked exception, then overridden can only
      declare unchecked)
    - return type must be same **or subtype** (**covariant return type**, so doesn't have to be exactly the same!!)

```java
    static class ClassWithMethodToOverride {
      protected Number get(Number num) throws java.io.IOException {
         return 1;
      }
    }
    
    static class ClassWithOverriddenMethod extends ClassWithMethodToOverride {
      @Override
      // public Integer get(Integer num) throws java.io.IOException { // DOES NOT COMPILE - signature must much - "covariant" types are not allowed
      // public Integer get(Number num) throws Exception { // DOES NOT COMPILE - exception should be the same or subtype
       public Integer get(Number num) throws java.net.SocketException { 
         return 2;
      }
    }
```

- be aware if there is the same signature, but with different, not covariant type - CODE DOES NOT COMPILE

```java
class Foo {
   void nothingToDo() { }
}
class Bar extends Foo {
  // int nothingToDo() { } // DOES NOT COMPILE: cannot override: return type int is not compatible with void
}
```

## Abstract classes

- it is possible to declare method `private` and `final` but not `private` and `abstract`

```java
    static abstract class AbstractClass { // note: abstract class does not have to declare "abstract" methods
      // private abstract void method(); // DOES NOT COMPILE:  error: illegal combination of modifiers: abstract and private
      // void abstract method(); // DOES NOT COMPILE
      abstract protected void  method_1(); // OK - abstract can be before
      protected abstract void method_2(); // OK - or after access modifier
      
      AbstractClass(){
        method_1(); // compiles
      }
    }
```

- when implementing `abstract` same rules applies as for method overriding

## Immutables

- class have to `final` or define `private` constructor to be immutable, otherwise it can be extended and mutable
  fields can be added !!
- creating immutable techniques:
    - defensive copy of collection provided in constructor or getter

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter6.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
