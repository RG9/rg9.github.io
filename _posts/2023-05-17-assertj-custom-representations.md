---
title: "AssertJ custom representation of asserted object"
categories: [Code tips]
tags: [assertj, java, DX]
---

I really enjoy asserting collections with AssertJ.
Usually it's safer and simpler to use `containsExactlyInAnyOrder` rather than asserting individual elements (`collection.get(0)`) - even if there is only one element, because it may change in the future.
The challenge starts when asserting collection containing complex objects, because constructing an entire object as expected element can be tedious.
To extract only certain properties, we can use `.extracting(Foo::field1, Foo::field2)`.

```java
        var players = List.of(
            new Player("Michael Jordan", new Team("Bulls")),
            new Player("Kobe Bryant", new Team("Lakers")));

        assertThat(players)
            .extracting(Player::name, player -> player.team().name())
            .containsExactly(
                tuple("Michael Jordan", "Bulls"),
                tuple("Kobe Bryant", "Lakers"));
```

However, I tend to concatenate properties to string because I wasn't fond of working with tuples:
```java
        assertThat(players)
            .extracting(player -> player.name() + " | " + player.team().name())
            .containsExactly("Michael Jordan | Bulls",
                "Kobe Bryant | Lakers");
```
The reason is that, by default, AssertJ provides a generic "tuple" representation in the "actual" section.
When copied, this has to be manually adapted to Java code, which can be inconvenient.

For example:
```
Expecting actual:
  [("Michael Jordan", "Bulls"),
    ("Kobe Bryant" "Lakers")]
```

What I want is an "easy-to-copy" representation of the asserted object:
```
Expecting actual:
  [tuple("Michael Jordan", "Bulls"),
    tuple("Kobe Bryant" "Lakers")]
```

Fortunately, there's an easy way to globally fix this in 3 simple steps.

1. Define a custom representation:
```java
class CustomAssertJRepresentation extends StandardRepresentation {

    static final CustomAssertJRepresentation INSTANCE = new CustomAssertJRepresentation();

    @Override
    protected String toStringOf(Tuple tuple) {
        return "tuple" + super.toStringOf(tuple);
    }
}
```

2. Then add it to the global configuration:
```java
public class CustomAssertJConfiguration extends Configuration {

    @Override
    public Representation representation() {
        return CustomAssertJRepresentation.INSTANCE;
    }

    @Override
    public String describe() {
        return "CustomAssertJConfiguration applied";
    }
}
```

3. Lastly, register the global config in this file:
`/src/test/resources/META-INF/services/org.assertj.core.configuration.Configuration`
which will contain:
```
my.package.CustomAssertJConfiguration
```

Refer to the official documentation for more information: https://assertj.github.io/doc/#assertj-core-representation