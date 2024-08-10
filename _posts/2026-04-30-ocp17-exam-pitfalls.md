---
categories:
- Projects
devto_id: 4120526
tags:
- ocp17
title: What Solving Hundreds of OCP Questions Taught Me
---

## 🧠 1. Think Like the Compiler, Not Like a Developer

As developers, we naturally focus on business logic.

The compiler doesn't.

It doesn't care that your algorithm is correct if:

- a variable is declared twice,
    
- a local variable isn't initialized,
    
- you're calling an instance method from a static context,
    
- an abstract class is instantiated,
    
- an overridden method throws a broader checked exception.
    

Some examples that caught me during preparation:

```java
int value = 1;
int value = 2;   // duplicate local variable
```

```java
public static void main(String[] args) {
    print();      // print() isn't static
}
```

```java
abstract class Animal {}
new Animal();     // doesn't compile
```

One habit helped me a lot:

> **Before trying to predict the output, always ask: "Does it compile?"**

---

## 🚫 2. Compilation Always Comes Before Runtime

This sounds obvious.

Yet I repeatedly made the same mistake.

I'd spend a minute calculating the output only to discover that the code didn't even compile.

Sometimes I'd immediately think:

> "This throws an exception."

Only to notice later that there was already a compilation error a few lines above.

The OCP exam loves hiding compilation errors inside otherwise perfectly valid code.

---

## 👀 3. Read Every Single Word

Some of my mistakes had nothing to do with Java.

I simply didn't read the question carefully enough.

Examples include:

- Choose **two** answers.
    
- Which statement is **NOT** true?
    
- Which methods can be removed so the code **compiles and runs**?
    
- Which answer is **always** correct?
    

I also remember selecting a perfectly valid answer...

...only to notice afterwards that the question asked for **"Compiles and runs"**, not **"Produces the same output."**

One word changed everything.

---

## 🎭 4. The Exam Loves "Almost Correct" Code

This is probably one of Oracle's favorite tricks.

Everything looks correct except for one tiny detail.

For example:

```java
interface A {
    void test() { }     // missing default
}
```

```java
record Person(String name) {
    int age;            // instance fields are not allowed
}
```

```java
@Override
void read() throws Exception { } // broader checked exception
```

The code looks almost perfect.

That's exactly why it's easy to miss.

---

## 🔍 5. Don't Trust Familiar APIs

Many APIs on the exam look familiar.

That's the trap.

Examples I remember getting wrong:

- confusing `peek()` with `poll()`,
    
- using a class that doesn't actually exist in `java.time`,
    
- forgetting that `Stream.min()` returns `Optional<T>`, while `IntStream.min()` returns `OptionalInt`,
    
- mixing up `Period` and `Duration`.
    

Whenever I wasn't 100% sure an API existed, I stopped guessing.

---

## ⚠️ 6. Pay Attention to Words Like "Always", "Must", and "Never"

These words completely change the meaning of a sentence.

For example:

- constructors **must** start with `this()` or `super()`? (only if they're explicitly called)
    
- `default` is **always** required? (not necessarily)
    
- a method **must never** throw an unchecked exception when overriding? (false)
    

One word often decides whether the answer is correct.

---

## ❌ 7. Eliminate Wrong Answers First

One of the biggest improvements in my exam strategy was learning to eliminate answers before searching for the correct one.

Sometimes I couldn't immediately identify the right option.

But I could confidently reject others because:

- the code doesn't compile,
    
- the API doesn't exist,
    
- the modifier isn't legal,
    
- the exception cannot occur.
    

That was often enough to find the correct answer.

---

## 📚 8. Small Language Rules Matter More Than You Think

The OCP exam rewards precision.

Some examples that repeatedly appeared during my preparation:

- `Duration` vs `Period`
    
- `Month.MARCH` vs `3`
    
- `List.of()` returns an immutable list
    
- `PriorityQueue` always removes the smallest element
    
- `switch` case labels must be compile-time constants
    
- `try-with-resources` always closes resources before entering `catch`
    

None of these rules are difficult.

The challenge is remembering them all under time pressure.

---

## 😴 9. Fatigue Creates More Mistakes Than Lack of Knowledge

This was one of the biggest surprises.

After solving many questions in a row, my error rate increased—not because the questions became harder, but because I became less careful.

Typical mistakes included:

- missing **Choose three answers**,
    
- stopping halfway through the code,
    
- assuming indentation affected execution,
    
- overlooking a single `final` keyword,
    
- **forgetting to check every answer because I thought I had already found the correct one**.
    

Sometimes taking a five-minute break was more valuable than solving another twenty questions.

---

## 💡 10. Every Question Exists to Test One Rule

Eventually, I stopped asking:

> **"What's the correct answer?"**

Instead, I started asking:

> **"What rule is Oracle trying to test?"**

That small shift completely changed the way I studied.

Instead of memorizing questions, I focused on understanding the language rules behind them.

Once I understood the rule, similar questions became much easier.

---

## 🚀 Final Thoughts

Preparing for the OCP exam taught me something unexpected.

The hardest part wasn't learning more Java.

It was learning to slow down.

To **read carefully**.

To think like the compiler before thinking like a developer.

Ironically, the more experience you have, the easier it is to make assumptions because your brain automatically fills in missing details.

If I could give one piece of advice to anyone preparing for the OCP exam, it would be this:

> **Slow down. Read every line. Think like the compiler. Only then think like a programmer.**