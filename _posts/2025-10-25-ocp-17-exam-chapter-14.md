---
title: OCP 17 Exam - chapter 14 notes - I/O
categories: [ Book notes ]
tags: [ ocp17, java ]
---

## Streams

- byte streams are called `*InputStream`/`*OutputStream`, where character streams are `*Reader`/`*Writer`

- exceptions are `PrintWriter` and `PrintStream`

- exam my trick us by wrapping `Writer/Reader` into `Input/OutputStream` or wrapping `InputStream` into `OutputStream`

- be aware that not all readers support `mark(int)`, so program might for some readers throw `IOException`

# IO

- it might be tricky that `File` has constructor with parent and child, e.g. `new File(new File("/root"), "/user/.bashrc")` (Q.15)

# NIO.2

- memorize creating `Path`: `Path.of`, `Paths.get`, `FileSystem.getPath` (Q.10)

- be aware that `Path#toRealPath` might throw `NoSuchFileException` (Q.23)

- `Files.readAllLines` returns `List`, while `Files.lines` returns `Stream` (Q.8)

- be aware that most of `Files` methods throws `IOException`, so questions might hide compilation error when `IOException` is not handled (Q.4)

- both methods `Files.walk` and `Files.find` returns `Stream<Path>`, but for `Files.find` we can provide path and attribute filter as method parameter (Q.5)

- to create directory we should use `Files.createDirectory` (to avoid exception when no parent, use `Files.createDirectories`)

# System console

- `System.console` might return `null` if not available (Q.3)


## Playground code

<https://github.com/RG9/rg-playground-ocp17/tree/main/Chapter14/src/test/java/pl/rg9/demo>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
