---
title: OCP 17 Exam - chapter 13 notes - concurrency
categories: [ Book notes ]
tags: [ ocp17, java ]
---

## Executors

- be aware whether `Executors.newSingleThreadExecutor` is used, because running on single thread automatically synchronizes executions, so result will be always consistent (Q.22)

- `awaitTermination(1, TimeUnit.SECONDS)` returns `false` instead of throwing timeout when tasks execution didn't finish (Q.12)

- be aware of methods of `ScheduledExecutorService` - only 1 accepts `Callable` (Q.4):
    - `ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit)`
    - `ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)`
    - `ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`
    - `ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`

- `scheduleAtFixedRate` will always wait for task to complete if overdue, but then starts new one immediately.
  Thus fixed rate is not really strict that task must be executed at given time.

- be aware of difference between `execute` and `submit`. First returns `void`, second returns `Future`. (Q.20)

## Reentrant Lock

- we could `lock` many times w/o exception (`holdCount` will increase), but calling `unlock` when not locked, will throw
  `IllegalMonitorStateException`

- also because `lock` could be called multiple times, the `unlock` should be the same number of times as `lock` to release lock (Q.17)

- `tryLock(long timeout, TimeUnit unit)` works that waits specified amount of time for lock, but returns `false` instead
  throwing timeout exception

## Runnable vs Callable

- `Callable` could throw check Exception, but `Runnable` does not (Q.3)
    - quote from javadoc of `Callable`:
      `A Runnable, however, does not return a result and cannot throw a checked exception.`

## Threads

- be aware of `Thread#run` vs `Thread#start`. First "runs" always in the invoking thread, while "start" starts new
  thread so result would be unpredictable. (Q.8)

- `Thread#interrupt` might be ignored by running tread or even lock. 
`InterruptedException` is checked exception, so must be explicitly declared when calling `Thread#sleep` or `java.util.concurrent.locks.ReentrantLock.lockInterruptibly`. (Q.16)

- **race condition** is when two threads wants to modify the same resource at the same time and result would be invalid
- **live lock** is when to threads are blocked forever but appear to be active

## Parallel stream

- `findFirst` will always return first element, no matter parallel stream is used (Q.21)

## Playground code

<https://github.com/RG9/rg-playground-ocp17/tree/main/Chapter13/src/test/java/pl/rg9/demo>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
