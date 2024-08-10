---
title: "JUnit's @CsvSource.quoteCharacter"
categories: [Code tips]
tags: [junit, java]
---


`@CsvSource` used with **text blocks** is really awesome! Similarly to [validation files](https://github.com/cronn/validation-file-assertions), it can be used to create easily readable tests with readily visible inputs and outputs placed side by side.

For example:
```java
	@ParameterizedTest
	@CsvSource(delimiterString = "->", textBlock = """
		ABC -> abc
		Abc -> abc
		""")
	void toLowercase(String input, String expected) {
		assertThat(input.toLowerCase())
			.isEqualTo(expected);
	}
```


What recently confused me was the following error:
```
org.junit.jupiter.api.extension.ParameterResolutionException:
No ParameterResolver registered for parameter [java.lang.String arg1] in method [void MyTest.test(java.lang.String,java.lang.String)].
```

It came out that it was caused by a parameter value starting with a single quote `'`, that wasn't closed by another `'` at the end of value. In my case the culprit was `'S-GRAVENHAGE`, which is Belgian street name.

The solution is to set parameter `quoteCharacter` to double quote `"`, provided we are using *text block*. This way we can test empty string with `""`.

Example:
```java
	@ParameterizedTest
	@CsvSource(quoteCharacter = '\"', delimiterString = "->", textBlock = """
		'S-GRAVENHAGE -> 's-gravenhage
		"" -> ""
		""")
	void toLowercase(String input, String expected) {
		assertThat(input.toLowerCase())
			.isEqualTo(expected);
	}
```

I hope it was helpful, cheers!