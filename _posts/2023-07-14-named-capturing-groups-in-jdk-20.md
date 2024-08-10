---
title: "Elevate your Regex: Named Capturing Groups in Java's JDK 20 API"
canonical_url: 'https://blog.cronn.de/en/java/2023/07/14/named-capturing-groups-in-jdk-20.html'
categories: [Code tips]
tags: [java, regex]
---

> **_NOTE:_** Originally posted on https://blog.cronn.de/en/java/2023/07/14/named-capturing-groups-in-jdk-20.html

## TL;DR
```
Since JDK 11 there haven't been any significant changes to Regex API. 
In JDK 20 a few new methods were introduced, aimed at facilitating the use of named capturing groups. 
We can now access the match of a group by its name, not just by its index.
```

Numerous tutorials exist on how to write Regexp patterns in general, yet few of them focus on Java API.
This article presents how to deal with capturing groups from a practical standpoint and how we can enhance our existing code with the improvements in JDK 20.

## How did we use capturing groups prior to JDK 20?

### Single capturing group

Let's say we want to extract IDs from a text. For simplicity's sake, let's assume the IDs are sequences of digits, and we can match them with a simple regex: `\d+`. Additionally, each ID is typically preceded by its name in the format `Id: {id}`, e.g. `Id: 123`.

The conventional approach to match text in such a case involves using a capturing group. This requires including the matching pattern in parentheses, like `(\d+)`:
The regex may look like this:
```perl
(?i)\bId[: ]*?(\d+)
```
provided:
* `(?i)` - case insensitive flag
* `\b` - word boundary, i.e. look for a complete word, not part of one
* `Id` - match the letters "Id"
* `[: ]*?` - match colon or space zero or more time, but as few times as necessary (non-greedy)
* `(\d+)` - match and capture one or more digits

Let's have a look at a full example:
```java
@ParameterizedTest
@CsvSource(textBlock = """
    Id: 123 | 123 | # happy path
    Id: 1 | 1 | # shorter id
    Id: . | | # missing id
    Id: unknown | | # text instead of number
    Id  123 | 123 | # space instead of ':'
    ID: 123 | 123 | # upper case
    Id:123 | 123 | #missing separating space
    OtherId: 123 |  |
    """, delimiterString = "|")
void matchId(String input, String expectedId) {
    var pattern = Pattern.compile("(?i)\\bId[: ]*?(\\d+)");

    var matcher = pattern.matcher(input);
    // using "java.util.regex.MatchResult.group(int)" to extract capture
    var id = matcher.results().findFirst().map(m -> m.group(1)).orElse(null);

    assertThat(id).isEqualTo(expectedId);
}
```

This approach seems a bit shaky. What does group `1` mean? Does `(?i)` count as a group? (It's actually the `CASE_INSENSITIVE` flag). Why don't groups start from `0`? What if we have more groups?

*Side note:*
_There's a tricky way to avoid a single capturing group by using a look-behind:_
```java
var pattern = Pattern.compile("(?i)(?<=\\bId[: ]{0,10})\\d+");
var matcher = pattern.matcher(input);
var id = matcher.results().findFirst().map(MatchResult::group).orElse(null);
```
_I used `{0,10}` instead of `*` because the look-behind calculates the length of the potential match in order to step back and check it. This method can be slow and has its limitations._

### Multiple capturing groups

Assume the text might contain different types of IDs from various institutions (Tax ID, Court ID, Stats ID).
In this case, we need to introduce another group to capture the type of ID.
It is a good practice to name multiple groups for better readability, like `(?<groupName>.*)`.
Let's look at an example:
```java
@Test
void matchDifferentTypesOfIds() {
    String text = """
        Some text. Tax Id: 123. Some text.
        Some text, Court Id: 456, Stats Id: 789.
        """;
    var pattern = Pattern.compile("(?i)(?<type>\\w+ *Id)[: ]*?(?<id>\\d+)");

    var matcher = pattern.matcher(text);
    // using "java.util.regex.MatchResult.group(int)" to extract capture
    List<IdEntry> idEntries = matcher.results()
        .map(m -> new IdEntry(m.group(1), m.group(2)))
        .toList();

    assertThat(idEntries)
        .extracting(IdEntry::type, IdEntry::id)
        .containsExactly(tuple("Tax Id", "123"),
            tuple("Court Id", "456"),
            tuple("Stats Id", "789"));
}
```

Unfortunately, prior to JDK 20, named capturing groups were not fully supported. We could only reference groups by their index.

## JDK 20 new features

Since JDK 11, there haven't been any significant changes to `java.util.regex`.
JDK 20 doesn't deviate much either (though it is a feature release).
You can see [the comparison between JDK 11 and JDK 20](https://javaalmanac.io/jdk/20/apidiff/11).
Besides the enhancement of named capturing groups,
there is just one cosmetic change introducing methods `java.util.regex.Matcher.hasMatch` and `java.util.regex.MatchResult.hasMatch`.

### Extended support for named capturing groups

Issue: [https://bugs.openjdk.org/browse/JDK-8292872](https://bugs.openjdk.org/browse/JDK-8292872).

This feature introduces the following new API methods:
```
java.util.regex.MatchResult
+ String group(String name)
+ int start(String name)
+ int end(String name)
+ Map<String,Integer> namedGroups()

java.util.regex.Pattern
+ Map<String,Integer> namedGroups()
```

Let's revisit the previous example. In JDK 20, we can write it like this:
```java
// using "java.util.regex.MatchResult.group(String)" to extract capture
var idEntries = matcher.results()
    .map(m -> new IdEntry(m.group("type"), m.group("id")))
    .toList();
```

Using the `start` and `end` methods of `MatchResult`, we can even find the position in the text of group matches:
```java
assertThat(matcher.results())
   .extracting(m -> m.group("type"),
        m -> m.group("id"), m -> m.start("type"), m -> m.end("id"))
   .containsExactly(tuple("Tax Id", "123", 11, 22),
      tuple("Court Id", "456", 46, 59),
      tuple("Stats Id", "789", 61, 74));
```

Lastly, the new methods include one that returns a map of group names and their corresponding indices:
```java
assertThat(pattern.namedGroups())
   .isEqualTo(matcher.namedGroups());
assertThat(pattern.namedGroups())
   .asString()
   .isEqualTo("{id=2, type=1}");
```


## Conclusion

JDK 20 finally brings full support for named capturing groups, allowing not just their definition, but also operations on them. Now, developers can reference groups by name rather than by their numerical index. It improves readability by making code more intuitive. Also enhances maintainability, because code is less prone to errors. A big thank you to the Java community and the developers behind this useful enhancement!
