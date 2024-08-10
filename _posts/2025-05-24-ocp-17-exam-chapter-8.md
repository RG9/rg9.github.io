---
title: OCP 17 Exam - chapter 8 notes
categories: [ Book notes ]
tags: [ ocp17, java ]
---

- exam may trick us by asking about functional interface that doesn't exist, e.g. `IntegerSupplier` instead `IntSupplier` (review Q.9)

- or we can be tricked by lambda that doesn't compile (review Q.17)
```java
    // Predicate<String> p2 = s -> {s.length() < 5}; // error: not a statement
    // Predicate<String> p2 = s -> {return s.length() < 5}; // error: ';' expected
    Predicate<String> p2 = s -> {return s.length() < 5; };
        System.out.println(p2.test("")); // prints true 
    
    p2 = s -> s.length() < 5; // or shorter
    System.out.println(p2.test("")); // prints true
    // System.out.println((s -> s.length() < 5).test("")); //  error: lambda expression not expected here
    
    p2 = __ -> true; // we can return boolean directly
    p2 = (s) -> true; // we can use brackets
    p2 = (String s) -> true; // define type explicitly
    // p2 = String s -> true; // error: not a statement, but brackets are required
```

- be aware that exam may trick us by using lambda parameters of already defined variables (review Q.19)
```java
    String s = "";
    // Predicate<String> p3 = s -> s.length() < 5; // error: variable s is already defined in method main(String[])
    Predicate<String> p3 = string -> string.length() < 5; // correct
```

- inside lambdas we can use only variables that are `final` or **effectively final** (review Q.13)

```java
    for (int i = 0; i < 2; i++) {
      // Supplier<Integer> supplier = () -> i;  //  error: local variables referenced from a lambda expression must be final or effectively final
      int j = i; // if we can set "final" modifier then it's definitely effectively final
      Supplier<Integer> supplier = () -> j; 
      System.out.println(supplier.get());
    }
```

- lambdas cannot be assigned `var`, because compiler doesn't have enough information to guess type (review Q.11)

- `Function#compose` first executes function passed as parameter! (review Q.12)

```java
    UnaryOperator<Integer> a1 = i -> i * 3;
	UnaryOperator<Integer> a2 = i -> i + 2;
    System.out.println(a1.compose(a2).apply(2)); // should be 12, because first a2 will be called
```

- `@FunctionalInterface` annotation is optional and should be used to ensure users that in the future annotated interface won't break lambda interface rules, otherwise there will be compilation error

- Main lambda interface rule is that it should have exactly one abstract method (provided that abstract methods with the same signatures as `Object` method don't count)

```java
  @FunctionalInterface // produces compilation error when interface breaks functional interface rules
  interface MyFunctionalInterface {
     void print();
     // void print2(); // DOES NOT COMPILE MyFunctionalInterface is not a functional interface- multiple non-overriding abstract methods found in interface MyFunctionalInterface
     String toString(); // Object method doesn't count
     int hashCode(); // Object method doesn't count
     boolean equals(Object otherObject); // Object method doesn't count
     default int one() { return 1; } // default method doesn't count
     private int two() { return 2; } // private method doesn't count
  }
```

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter8.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
