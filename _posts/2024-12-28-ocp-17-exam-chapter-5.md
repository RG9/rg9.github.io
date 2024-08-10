---
title: OCP 17 Exam - chapter 5 notes (methods)
categories:
  - Book notes
tags:
  - ocp17
  - java
---

* **access modifiers** (`public`, `protected`, ...) and **optional modifiers** (`final`, `static`) can be in any order, but must be before **return type**
```
void public method() { } // DOES NOT COMPILE: invalid method declaration; return type required
```

- variable is **effectively final** if not assigned second time (doesn't change), but doesn't have to be marked `final`.
 
* **varargs** rules
```java
    static class VarargsPlayground {
        public static void method(int i, int... num) {
           System.out.println("i: "+i +", varargs len: "+ num.length);
        }
        public static void method(Integer i, int... num) { // OK, because Java tries to find "most specific method" 
           System.out.println("Integer: "+i +", varargs len: "+ num.length);
        }
        private static void method(int... num) { // COMPILES, but doesn't make sense because we cannot really use varargs (we have to pass array explicitly)
           System.out.println("varargs len: "+ num.length);
        }
       // static void method(int[] num) { } // DOES NO COMPILE: cannot declare both method(int[]) and method(int...)
       //  static void method(int... i, int... num) { } // DOES NOT COMPILE: varargs parameter must be the last parameter
       // static void method(int... i, int num) { } // DOES NOT COMPILE: varargs parameter must be the last parameter
    }

      //  VarargsPlayground.method(1); // DOES NOT COMPILE: reference to method is ambiguous
      // VarargsPlayground.method(null); // java.lang.NullPointerException: Cannot read the array length because "<parameter1>" is null
       VarargsPlayground.method();
      // VarargsPlayground.method(1, 2); // DOES NOT COMPILE: reference to method is ambiguous
       VarargsPlayground.method(new int[1]);
       VarargsPlayground.method(1, new int[1]);
       VarargsPlayground.method(Integer.valueOf(1), new int[2]);
```


* be aware that we cannot call instance (non-static) method inside `static` method - we need object reference

* **static imports**

```java
// import static java.util.Arrays;       // DOES NOT COMPILE - we shoud use .* (error: static import only from classes and interfaces)
import static java.util.Arrays.asList; // OK, but we should use "asList" instead "Arrays.asList(1)"
// static import java.util.Arrays.*;     // DOES NOT COMPILE because wrong order of modifiers
import static java.util.Arrays.*; // OK, "Arrays.asList" would be redundant
```

* Java is a **“pass-by-value”** language. Java makes a copy of primitive/reference and method receives this copy. In case of passing object to method call, we will have two references (original variable and method parameter) pointing to the same object in memory.
* alternative approach is "pass-by-reference", e.g. Perl

 
* **autoboxing and unboxing rules** 
	- be aware that the same rule applies when passing values as method parameter 
		- thus cannot invoke `method(int i)` with `method(1L)` - DOES NOT COMPILE because of "possible lossy conversion"
```java
       System.out.println("autoboxing");
       var a0 = 1;
       int a1 = 1;
       byte a2 = 1;
       short a3 = 1;
       long a4 = 1;
       float a5 = 1;
       double a6 = 1;
       char a7 = 1;
       boolean a8 = true;
       Integer b1 = 1;
       Byte b2 = 1;
       Short b3 =1;
      // Long b4 = 1; // DOES NOT COMPILE: incompatible types: int cannot be converted to Long
      // Float b5 = 1; // DOES NOT COMPILE: incompatible types: int cannot be converted to Float
     //  Double b6 = 1; // DOES NOT COMPILE: incompatible types: int cannot be converted to Double
       Long b4 = 1L;
       Float b5 = 1.0f;
       Double b6 = 1.0;
       Character b7 = 1;
       Boolean b8 = true;
       System.out.println("unboxing");
       a1 = b1; 
       a1 = b2;
       a1 = b3;
      // a1 = b4; // DOES NOT COMPILE; incompatible types: Long cannot be converted to int
      // a1 = (int) b4; DOES NOT COMPILE = error: incompatible types: Long cannot be converted to int
      // a1 = a4; // DOES NOT COMPILE: incompatible types: possible lossy conversion from long to int
      a1 = (int) a4;
      // a1 = b5; // DOES NOT COMPILE; incompatible types: Float cannot be converted to int
      // a1 = a5; // DOES NOT COMPILE:  incompatible types: possible lossy conversion from float to int
      a1 = (int) a5;
       a1 = b7;
      //  a1 = b8; // DOES NOT COMPILE: incompatible types: Boolean cannot be converted to int
       // a2 = b1; // DOES NOT COMPILE: incompatible types: Integer cannot be converted to byte
       a2 = b2;
       // a3 = b1; // DOES NOT COMPILE: incompatible types: Integer cannot be converted to short
       a3 = b2;
       a3 = b3;
       a5 = b3;
       a6 = b3;
```
 

* **Overloaded methods** are methods with the same **name** but a different parameter list. 

* "Java calls the most specific method it can find", so we can overload with **"autoboxed type"**
```
public class Foo {
   public void bar(int i) {}
   public void bar(Integer i) {}
}
```

* **method to call order**, e.g. `method(1)` - 
	- exact type - `void method(int i)` 
	- larger primitive  - `void method(long i)`
	- autoboxed type - `void method(Integer i)`
	- varargs - `void method(int... i)` 


## Exam notes
 * labels are not allowed for methods
```java
 zzz: void method () { } // DOES NOT COMPILE
```

* watch out for intentional typos like `...int` in varargs (should be `int...`)
```java
public void method(String[] string, …int nums) {}
```

* is package access more lenient than `protected`?
	* No, because `protected` allows to be accessed from the same package plus subclass access.
* if we call static method on `null` instance reference then there is no `NullPointerException`. Java translates instance reference `foo` to class reference `Foo`.
```java
    static class Foo {
      static int bar = 5;
    }
    ...
    Foo foo = null;
    System.out.println(foo.bar); // no NPE because static field is used
```
* pay attention because there could be 2 compilations errors to the same variable but in different lines and it still counts!!
* "If a variable is `static final`, it must be set exactly once, and it must be in the declaration line or in a static initialization block."
* instance initialization block have access to both instance ans `static` variables
```
    static class InstanceInitializers {
      int i;
      static long l;
      {
        i = 1;
        l = 1;
      }
      static {
        // i = 2; // DOES NOT COMPILE error: non-static variable i cannot be referenced from a static contex
        l = 2;
      }
    }
```

> actually we should avoid initializers, because they make code more complicated
{: .prompt-tip } 

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter5.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
