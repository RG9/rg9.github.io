---
title: OCP 17 Exam - chapter 12 notes - modules
categories: [ Book notes ]
tags: [ ocp17, java ]
---

> Note:
Java Platform Module System was introduced in JDK 9, see:  https://openjdk.org/projects/jigsaw/spec/
In java 25 modules would be available through imports like regular classes, see: https://www.baeldung.com/java-25-features#2-module-import-declarations-jep-511--preview.
{: .prompt-tip }


## Running and packaging simple module

Folder structure of `app`:
```
- app
  - src/pkg
    - App.java // prints hello world
  - module-info.java // only module app { }
```

Run below commands in parent directory <https://github.com/RG9/rg-playground-ocp17/blob/main/java-modules-demo>.

Compile to `app/out`:
```shell
$ javac --module-path mods -d app/out app/src/pkg/*.java app/module-info.java
```

> Note: --module-path is not needed now as "app" module do not have any dependencies, but good to know that parameter
{: .prompt-info }

Package into jar:
```shell
$ jar -cvf mods/app.jar -C app/out  .
added manifest
added module-info: module-info.class
adding: pkg/(in = 0) (out= 0)(stored 0%)
adding: pkg/App.class(in = 414) (out= 289)(deflated 30%)
```

Run:
```shell
$ java -p mods -m app/pkg.App
Hello app!
```

> `-p` is abbreviation of `--module-path`
> `-m` is abbreviation of `--module`
{: .prompt-info }

> Be aware of format of `--module` param, which should be separated by slash: `-m module/class`
{: .prompt-warning }

> Side note:
> Java module system is different than one in Intellij! https://www.jetbrains.com/help/idea/creating-and-managing-modules.html#modules-idea-java
> Here you can read more about support for Java modules in Intellij: https://blog.jetbrains.com/idea/2017/03/support-for-java-9-modules-in-intellij-idea-2017-1/
> Instead of creating a project, you should just "Open" parent directory.
{: .prompt-tip }

## Defining service

TBD

## Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/java-modules-demo>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
