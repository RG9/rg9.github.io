---
title: Better raise Error Prone's "EqualsIncompatibleType" check to ERROR
categories:
  - Code tips
tags:
  - errorprone
  - java
---

Let's consider the following example:

```java
var list = List.of(1);
var first1 = list.stream().findFirst();
var first2 = list.getFirst();
if (Objects.equals(first1, first2)) { // compiles, just warning in Intellij
	// unrechable
}
```

It's hard to spot when using `var` instead of explicit type.
In project, I'm working I found a piece of such bug in prod and test code.

To force fixing such error, we could simply raise Error Prone's [EqualsIncompatibleType](https://errorprone.info/bugpattern/EqualsIncompatibleType) bug pattern to `ERROR`.

> Below instruction is from ChatGTP
{: .prompt-info }

To set the `EqualsIncompatibleType` bug pattern to error in Error Prone, 
update your Maven or Gradle configuration to include the following compiler argument:

```
-Xep:EqualsIncompatibleType:ERROR
```

For Maven, add it to the `compilerArgs` in your `pom.xml`:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <compilerArgs>
      <arg>-Xep:EqualsIncompatibleType:ERROR</arg>
    </compilerArgs>
  </configuration>
</plugin>
```

For Gradle, add it to your `build.gradle`:

```groovy
tasks.withType(JavaCompile) {
    options.compilerArgs += ['-Xep:EqualsIncompatibleType:ERROR']
}
```

This will treat all `EqualsIncompatibleType` findings as errors during compilation.
