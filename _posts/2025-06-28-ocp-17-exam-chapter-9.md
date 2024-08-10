---
title: OCP 17 Exam - chapter 9 notes
categories: [ Book notes ]
tags: [ ocp17, java ]
---

- be aware of removing or adding elements to an immutable list (`List.of`) as it will throw `UnsupportedOperationException` (review Q.2)

- how to properly define a collection:

```java
    // HashSet<Number> hs = new HashSet<Integer>(); incompatible types: HashSet<Integer> cannot be converted to HashSet<Number>
    HashSet<? extends Number> hs = new HashSet<Integer>();
    Map<String, ? extends Number> hm = new HashMap<String, Integer>();
    // HashSet<? super Number> hs = new HashSet<Integer>(); error: incompatible types: HashSet<Integer> cannot be converted to HashSet<? super Number>
    HashSet<? super Number> hs2 = new HashSet<Number>();
    HashSet<? super ClassCastException> set = new HashSet<Exception>();
    
    // List<> list = new ArrayList<String>(); // error: illegal start of type
    // List<Object> values = new HashSet<Object>(); // incompatible types: HashSet<Object> cannot be converted to List<Object>
    //  List<Object> objects = new ArrayList<? extends Object>(); // : error: unexpected type <? extends Object>
    List<Object> objects = new ArrayList(); // warn:  uses unchecked or unsafe operation
    List<String> obj1 = new ArrayList(); 
    List obj2 = new ArrayList<String>();
    var obj3 = new ArrayList();
```
- collection parametrized type (inside `<>`) on the left of assignment must be equal or wildcard must be used

> exam my trick us with valid generic type, but a collection type is wrong, e.g. assigning `HashSet` to `List` (review Q.11)
{: .prompt-info }


- what is valid method override with a collection as parameter and return type (review Q.7):

```java
  static class Foo {
    List<String> bar(List<String> a) {   System.out.println("Foo"); return null; }
  }
  
  static class BarOverride extends Foo {
    @Override
     List<String> bar(List<String> a) {   System.out.println("BarOverride"); return null; } // ok
    // ArrayList<String> bar(List<String> a) {  System.out.println("BarOverride"); return null; } // ok
    // List<? extends String> bar(List<String> a) {  System.out.println("BarOverride"); return null; } // DOES NOT COMPILE
    // Object bar(List<String> a) {   System.out.println("BarOverride"); return null; } // DOES NOT COMPILE
    // List bar(List<String> a) {   System.out.println("BarOverride"); return null; } // ok
    // --
    // List<String> bar(ArrayList<String> a) {   System.out.println("BarOverride"); return null; } // DOES NOT COMPILE
    // List<String> bar(List<? extends String> a) {   System.out.println("BarOverride"); return null; } // DOES NO COMPILE
    // List<String> bar(List<?> a) {   System.out.println("BarOverride"); return null; } // DOES NOT COMPILE
    // List<String> bar(List a) {   System.out.println("BarOverride"); return null; } // ok
  }
```

- be aware of reverser order by comparing second parameter of `Comparator#compareTo` to first (review Q.8)

```java
    String[] values = { "A", "1", "a" };
    Arrays.sort(values);
    System.out.println(Arrays.toString(values)); // prints [1, A, a]
    
    Arrays.sort(values, new Comparator<String>() {
      public int compare(String a, String b) {
        return b.compareTo(a); // reverse
      }
    });
    // or 
    // Arrays.sort(values, (a, b) -> b.compareTo(a));
    // Arrays.sort(values, Comparator.reverseOrder());
    System.out.println("reverse: " + Arrays.toString(values)); // prints [a, A, 1]
    
    Arrays.sort(values, new Comparator<String>() {
      public int compare(String a, String b) {
        return a.compareTo(b);
      }
    });
    // or 
    // Arrays.sort(values, Comparator.naturalOrder());
    System.out.println(Arrays.toString(values)); // prints [1, A, a]
```

> exam my trick by using class that implements both `Comparable` and `Comparator`, but in different order, so read code carefully! (review Q.12)
{: .prompt-info }

```java
    static record BothComparableAndComparator(int i) implements Comparable<BothComparableAndComparator>, Comparator<BothComparableAndComparator> {
      public int compareTo(BothComparableAndComparator that){ return this.i - that.i; } // MUST BE PUBLIC
      public int compare(BothComparableAndComparator b1, BothComparableAndComparator b2) { return b2.compareTo(b1); } // reverse
    }

    var treeset = new TreeSet<BothComparableAndComparator>(new BothComparableAndComparator(123));
    treeset.add(new BothComparableAndComparator(455));
    treeset.add(new BothComparableAndComparator(300));
    System.out.println(treeset); // prints 455, 300
```

> be aware that `List#remove` (review Q.15)
{: .prompt-info }

- `Collection` interface methods

```java
    Collection<Integer> collection = new ArrayList<>();
    collection.add(1);
    System.out.println(collection);
    collection.addAll(List.of(2, 3, 4, 5, 6, 7, 8, 9, 10));
    System.out.println(collection);
    // --
    System.out.println("contains 2: "+ collection.contains(2));
    System.out.println("contains 22: "+ collection.contains(22));
    System.out.println("contains 1, 2: "+ collection.containsAll(List.of(1, 2)));
    System.out.println("contains 0, 1, 2: "+ collection.containsAll(List.of(0, 1, 2)));
    // --
    collection.remove(3);
    System.out.println(collection);
    collection.removeAll(List.of(1, 2));
    System.out.println(collection);
    collection.retainAll(List.of(4, 5, 6)); // remove all elements that are not int the collection
    System.out.println(collection);
    collection.removeIf(integer -> integer == 4);
    System.out.println(collection);
    collection.clear();
    System.out.println(collection);
```

- `Queue` specific methods

```java
    Queue<Integer> queue = new LinkedList<>();
    System.out.println("add: " + queue.add(1));
    System.out.println(queue);
    System.out.println("offer: " + queue.offer(2)); // in contrary to add(..) doesn't throw IllegalStateException when queue reached its capacity
    System.out.println(queue);
    // --
    System.out.println(queue.peek()); // head of queue
    System.out.println(queue.element()); // in contrary to peek() doesn't throw exception when queue is empty
    // --
    System.out.println("remove:  " + queue.remove()); // removes head
    System.out.println(queue);
    System.out.println(queue.peek());
    System.out.println(queue.element());
    System.out.println("poll: " + queue.poll()); // in contrary to remove() doesn't throw exception when queue is empty
    System.out.println(queue);
    System.out.println(queue.peek());
    // System.out.println(queue.element()); // throws NoSuchElementException
    System.out.println("poll: " + queue.poll());
    // System.out.println("remove:  " + queue.remove()); // throws NoSuchElementException
```

- `Map` methods (review Q.16)

```java
    System.out.println("MAP METHODS");
Map<Integer, String> map = new HashMap<>();
    System.out.println("put 1a: prev value: " + map.put(1, "1a")); // prints null as prev value
	System.out.println("put 2a, prev value: " + map.put(1, "1b")); // prints 1a as prev value
	map.putAll(Map.of(1, "1c", 2, "2"));
	map.putIfAbsent(1, "1d");
    map.putIfAbsent(3, "3");
    System.out.println(map); // prints {1=1c, 2=2, 3=3}
// --
// map.merge(1, null, (v1, v2) -> null); NPE
    map.merge(1, "", (v1, v2) -> null); // removes key 1
	System.out.println(map); // prints {2=2, 3=3}
    map.merge(2, "2m", (v1, v2) -> v1); // no change - keeps old value
	System.out.println(map); // prints {2=2, 3=3}
    map.merge(2, "2m", (v1, v2) -> v2);
	System.out.println(map); // prints {2=2m, 3=3}
    map.put(2, null);
    System.out.println(map); // {2=null, 3=3}
    map.merge(2, "2m", (v1, v2) -> v1); // merge function is not called if existing value is NULL !!!
	System.out.println(map); // prints {2=2m, 3=3}
// ...
```
> note: if attempting to merge value that is currently `null`, then merge function is not called (review Q.19)
{: .prompt-info }

- there is a nice way to define immutable Map in single call:
```java
Map.ofEntries(
   Map.entry("k1", "v1"),
   Map.entry("k2", "v2"))
```

# Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/Chapter9.java>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
