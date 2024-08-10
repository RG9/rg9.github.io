---
title: OCP 17 Exam - chapter 11 notes
categories: [ Book notes ]
tags: [ ocp17, java ]
---

## try-with-resources

- can be used only with classes that implement `AutoCloseable` (also `java.io.Closeable` which extends `AutoCloseable`)

- two or more resources can be defined in one `try`, separated by `;`

- implicit `finally` blocked, that closes resource, is executed before any programmer code (`catch` or explicit
  `finally`)

- suppressed exception is exception that was thrown when closing resource in `try-with-resources` while handing another
  exception from `try` block. Java treats first exception as primary and other as `java.lang.Throwable.getSuppressed`.

```java
  static record ThrowsOnClosing(String name) implements AutoCloseable {
    public void close() throws Exception { // throws is optional!!
      System.out.println(" - closing java.lang.AutoCloseable: " + name);
      throw new IllegalArgumentException("closing resource: " + name);
    }
  }
  
  static record ThrowsOnClosingClosable(String name) implements java.io.Closeable { // Closeable extends  AutoCloseable
    public void close() throws java.io.IOException { // throws is optional!!
      System.out.println(" - closing java.io.Closeable: " + name);
      throw new IllegalArgumentException("closing resource: " + name);
    }
  }
  
  static void tryWithResources()
          throws Exception // since AutoCloseable#close can throw Exception we must catch it or declared as throws; otherwise DOES NOT COMPILE
  {
    var res2 = new ThrowsOnClosing("res2"); // effectively final
    try(var res1 = new ThrowsOnClosing("res1");
        res2; // automatically close res2
        var res3 = new ThrowsOnClosingClosable("res3");
        ThrowsOnClosing res4 = null) { // null is allowed
      System.out.println(" - try block");
      throw new IllegalArgumentException("thrown inside try");
    } catch (IllegalArgumentException e) {
      System.out.println(" - catch block");
      throw new IllegalArgumentException("rethrowing " + e.getMessage(), e);
    } finally {
      System.out.println(" - finally block");
      // throw new IllegalArgumentException("throwing from here will hide any previous exception!!");
    }
  }
  
  // ---

  public static void main(String[] args) {
    try {
      tryWithResources();
    } catch (Exception e){
      System.out.println("Caught: " + e.getMessage());
      for (Throwable t: e.getCause().getSuppressed()) {
        System.out.println("Suppressed: "+t.getMessage());
      }
    }
  }
```

prints:
```
 - try block
 - closing java.io.Closeable: res3
 - closing java.lang.AutoCloseable: res2
 - closing java.lang.AutoCloseable: res1
 - catch block
 - finally block
Caught: rethrowing thrown inside try
Suppressed: closing resource: res3
Suppressed: closing resource: res2
Suppressed: closing resource: res1
```

> the order of exceptions in `catch` block is important -
> first more specific exceptions (e.g. IllegalArgumentException) then more general (e.g. RuntimeException). 
> Otherwise there will be compile error that catch block is unreachable (review Q.4)
{: .prompt-warning }


> try-with-resources cannot be declared w/o braces like `if`!! (review Q.13)
{: .prompt-warning }

## Formatting

- decimal format `#` digit is optional, `0` if digit is absent then puts `0' 
```java
System.out.println(new DecimalFormat("#,##.000").format(1234.5)); // prints 12,34.500
System.out.println(new DecimalFormat(".###").format(1234.5)); // prints 1234.5
// System.out.println(new DecimalFormat("#.#.#").format(1234.5)); // java.lang.IllegalArgumentException: Multiple decimal separators in pattern "#.#.#"
System.out.println(new DecimalFormat("000.###").format(1)); // prints 001
```

- date formatting example (review Q.16):
```java
    System.out.println(ZonedDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd 'at' HH:mm:ss z (Z)", Locale.ENGLISH)));
      // prints e.g. 2014-02-02 at 21:30:57 CEST (+0200)

```

- be aware that text inside pattern must be inside `'`:
```java
    // System.out.println(DateTimeFormatter.ofPattern("hh o'clock")); // java.lang.IllegalArgumentException: Unknown pattern letter: o
    System.out.println(DateTimeFormatter.ofPattern("hh 'o''clock'").format(LocalTime.now())); // 12 o'clock (use ' to escape ')
```

> be aware that you should not use date formatting for `LocalTime`, otherwise there will be RuntimeException!
{: .prompt-warning }

- number formatting examples:
```java
    double amount = 120_400.02;
    System.out.println(NumberFormat.getCompactNumberInstance().format(amount)); // 120K
    System.out.println(NumberFormat.getCompactNumberInstance(new Locale("en"), Style.SHORT).format(amount)); // 120K
    System.out.println(NumberFormat.getCompactNumberInstance(new Locale("en"), Style.LONG).format(amount)); // 120 thousand
    System.out.println(NumberFormat.getCurrencyInstance().format(amount)); // $120,400.02
    System.out.println(NumberFormat.getCurrencyInstance(new Locale("pl")).format(amount)); // 120 400,02 ¤
    System.out.println(NumberFormat.getCurrencyInstance(new Locale("pl", "PL")).format(amount)); // 120 400,02 zł
```

## Locale

- in Java 17.0.7 all following compiles and run w/o Exception (review Q.15):
```java
   System.out.println(new Locale("ab")); // prints ab
   System.out.println(new Locale("cd", "GGGG")); // prints cd_GGGG
   System.out.println(new Locale("CD")); // prints cd
   System.out.println(new Locale("CD", "country")); // prints cd_COUNTRY
```

> exam might trick us with using a variable name that is already used!
{: .prompt-warning }

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter11.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
