---
title: OCP 17 Exam - chapter 10 notes
categories: [ Book notes ]
tags: [ ocp17, java ]
---

## Streams fundamentals
- "Streams are evaluated lazily, which means **intermediate operations** are not executed until a terminal operation is invoked."
```java
    var ints = new ArrayList<Integer>();
    ints.add(1); 
    var intsStream = ints.stream();
    ints.add(2);
    System.out.println(intsStream.count()); // prints 2 !!
```
- "Streams are single-use; stream must have at most one terminal operation." Exception will be thrown when terminal operation is called again on stream reference: (review Q.3)
```java
    var stream = Stream.iterate(0, i -> i+1).limit(10);
    var m1 = stream.noneMatch(i -> i < 0);
    // var m2 = stream.anyMatch(i -> i > 5); // throws java.lang.IllegalStateException: stream has already been operated upon or closed
    var m2 = true;
    System.out.println(m1 + " | " + m2);
```

> Note: `java.util.Collection.parallelStream` can be used to speedup processing of large stream with time-consuming operations.
{: .prompt-info }

## Infinite streams

- **infinite streams** can be created with `Stream#iterate` or `Stream#generate`:

```java
    Stream.iterate(1, i -> i + 1).limit(10)
        .forEach(i -> System.out.print(i +",")); // prints 1,2,3,4,5,6,7,8,9,10,
    System.out.println("");   
    
    Stream.iterate(1, i -> i <= 10, i -> i + 1)
        .forEach(i -> System.out.print(i +",")); // prints 1,2,3,4,5,6,7,8,9,10,   
    System.out.println("");    
    
    Stream.generate(() -> new Random().nextInt(10)).limit(10)
        .forEach(i -> System.out.print(i +",")); // prints random numbers 
    System.out.println("");  
```

> infinite stream w/o stop (e.g. `limit(10)`) causes program hang! (review Q.2)
{: .prompt-warning }

> Note: only `findFirst` and `findAny` terminate infinite stream. Methods that sometimes terminates are: `allMatch`, `anyMatch`, `noneMatch`.
{: .prompt-info }

## Reduce operations

- `Stream#reduce` - if you don't specify identity parameter, then return type is `Optional`, see method signatures:

```java
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
```

- `Stream#collect` is called **mutable reduction** as we can accumulate an intermediate result, see method signature:

```java
<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);
```

- parameters of `Stream#collect`:
    - `supplier` is used to initialize collection
    - `accumulator` adds elements to collection
    - `combiner` combines to collection - useful for parallel stream

## Primitive streams

- all ways to create primitive stream:

```java
    var intStreamEmpty = IntStream.empty();
    var intStreamOfValues = IntStream.of(1, 2, 3);
    var intStreamOfSingleValue = IntStream.of(1);
    var intStreamRange = IntStream.range(0, 10);
    var intStreamRangeClosed = IntStream.rangeClosed(0, 10);
    var intStreamIterateInfinite = IntStream.iterate(1, i -> i + 1);
    var intStreamGenerateInfinite = IntStream.generate(new Random()::nextInt);
    var intStreamUnboxed = Stream.of(1, 2, 3).mapToInt(Integer::intValue);
```

- remember that return type of `average` is `OptionalDouble`, whereas `IntSummaryStatistics#getDouble` returns `double` with `0` when empty stream (review Q.8):

```java
    OptionalInt optionalInt = IntStream.empty().findFirst();
int sum = IntStream.empty().sum();
OptionalDouble optionalDouble = IntStream.empty().average();
    if(optionalDouble.isPresent()) {
	System.out.println(optionalDouble.getAsDouble());
	}
	System.out.println(IntStream.empty().summaryStatistics().getAverage()); // prints 0.0

	// prints IntSummaryStatistics{count=3, sum=6, min=1, average=2.000000, max=3}
	System.out.println(intStreamOfValues.summaryStatistics());
```

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter10.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
