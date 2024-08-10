---
title: The hassle of transitive dependencies across sub-projects in Gradle
canonical_url: 'https://blog.cronn.de/en/gradle/java/2021/05/06/gradle-dependencies-gotchas-in-subprojects.html'
categories: [Code deep dives]
tags: [gradle, java]
---

> **_NOTE:_** Originally posted on https://blog.cronn.de/en/gradle/java/2021/05/06/gradle-dependencies-gotchas-in-subprojects.html

You have probably heard the term [dependency hell](https://blog.gradle.org/avoiding-dependency-hell-gradle-6).
Modern build tools, like Gradle, handle that problem very well, especially by providing automatic resolution of transitive dependencies.
By default, Gradle picks the highest version when many different versions of the same library are required.
However, it still has some flaws, and we will focus on one of them, observed in a multi-project in Gradle.

## What exactly is the problem?

First of all, we should clarify the terminology.
Dependencies can be divided into two categories: explicitly or implicitly declared.
The former is called a *“first-level”* dependency, the latter a *“transitive”* dependency.

Let's say in a multi-project we have two sub-projects: `project-a` and `project-b`.
Our `project-a` has a first-level dependency on library `lib-x` in version `1.2`, while `project-b` has a dependency on library `lib-y`, which also has a dependency on `lib-x` but here it is in a **lower** version - let's say `1.1`.
What's more `project-a` depends on `project-b`. It could be expressed as:

```
project-a
- project-b
- lib-x 1.2

project-b
- lib-y
  - lib-x 1.1
```

Running `project-a` will resolve the transitive dependency `lib-x` in the highest version `1.2` as expected.
However, running `project-b` (e.g. tests) will resolve `lib-x` in version `1.1`, so there will be an inconsistency between dependent projects.
A simple solution would be for `project-b` to have a **higher** version of `lib-x`, as this would result in both resolved versions being the same for both sub-projects. But this isn't always the case.

The dependency report for `project-a` with a **lower** version of `lib-x` in `project-b` would look like this:
```
+--- project-b
|    +--- lib-y
|    |	  +--- lib-x 1.1 -> 1.2
+--- lib-x 1.2
```

Whereas the report for `project-b` alone will resolve to a different version:
```
+--- lib-y
|    +--- lib-x 1.1 
```

*Sure, but actually it makes sense, doesn't it?*
Right, `project-b` knows nothing about the dependencies of `project-a`.
Indeed, this is how [resolution strategy](https://docs.gradle.org/current/userguide/dependency_resolution.html#sub:resolution-strategy) works - Gradle picks the highest version from all requested modules in the given context.

*So what's the problem then?*
In most projects, we treat sub-projects as a whole.
Therefore, if dependency resolution is not consistent across all sub-projects, it may result in unexpected behavior at runtime!

*Could this really happen?*
There are some examples of [binary incompatibility](https://stackoverflow.com/questions/43568116/is-there-a-way-to-stop-scala-2-12-breaking-the-jackson-object-mapper), but it's even worse when it comes to runtime differences because it's harder to spot.
Besides, according to Murphy's Law: “If anything can go wrong, it will” ;)


Now let's move on and consider a scenario when the inconsistency is between transitive dependencies.
We modify our example so that `lib-x` in version `1.2` is now required by a first-level dependency `lib-z` declared in `project-a`.

```
project-a
- project-b
- lib-z
  - lib-x 1.2

project-b
- lib-y
  - lib-x 1.1
```

This looks even less intuitive because the build tool should do [all the dirty work for us](https://rafael.cordones.me/blog/transitive-dependency-mediation-in-maven-and-gradle/#can-i-emhandleem-the-maven-truth), right? ;)

Summarizing, we could consider two relevant scenarios of resolving conflicting versions between sub-projects (provided that one depends on another):
1. first-level vs. transitive
2. transitive vs. transitive

## How to detect version conflicts?

To do this we need a tool that resolves all dependencies from each project separately, then lists them together to find duplicates.
In my case, the natural pick is *Intellij IDEA*.
We can check dependencies in the `Libraries` view in the `Project structure` window.
It gathers all resolved dependencies from *all configurations*.
If `<group>:<name>` is listed more than once, it could mean you are in trouble :)
The drawback is that you must find these conflicts with your own eyes.
Another thing is that `buildSrc` dependencies are also included, which could be misleading.

If you are not an Intellij user, you might try Gradle's `dependencies` task.
However, it only generates reports for a single project.
To use it for all sub-projects we should do [this trick](https://solidsoft.wordpress.com/2014/11/13/gradle-tricks-display-dependencies-for-all-subprojects-in-multi-project-build) and define our own task. We may also want to run it only for the `testRuntimeClasspath` configuration:
```groovy
allprojects {
	afterEvaluate { project ->  
		if (project.configurations.findByName("testRuntimeClasspath")) {  
			task allDeps(type: DependencyReportTask) {  
				configuration = "testRuntimeClasspath"  
			}  
		}  
	}
}
```

Then, we may try to parse the task output and find conflicting dependencies:
```bash
$ ./gradlew allDeps | sed -E 's/:[0-9][^ ]* -> /:/g' | grep -Po "[^ ]+:.+:\d[^ ]*" \
| sort | uniq | cut -d':' -f-2 | uniq -c | grep -v 1
      2  commons-beanutils:commons-beanutils
      2  commons-collections:commons-collections
	  ...
 ```
Let's explain this code.
First by `sed -E 's/:[0-9][^ ]* -> /:/g'`, we want to make sure only resolved versions are considered.
Thus, we cut off the declared version from transitive dependencies marked with an arrow (`->`).
Next with `grep -Po "[^ ]+:.+:\d[^ ]*"` only GAVs (`<group>:<name>:<version>`) are extracted.
Then we prepare a unique list of versions with `sort | uniq`.
Next, we want to group by `<group>:<name>` to identify duplicated versions, so we cut off the version with `cut -d':' -f-2`.
Lastly, we count each occurrence `uniq -c` and print only those with count greater than one: `grep -v 1`.

*Okay, but is there really no way to tell Gradle to warn me?*
Actually, there is. Gradle has a built-in check for transitive dependency conflicts.
We could try the `failOnVersionConflict` resolution strategy.
Then, with the help of the task `dependencyInsight`, we can narrow down the conflict to a single dependency.
Unfortunately, it's not really helpful in our case, because single libraries or large BOMs could have many conflicts themselves (e.g. `org.springframework.boot:spring-boot-dependencies:2.2.7.RELEASE`).

Nevertheless, there is some good news :)
In future releases (Gradle 7.x) there are some plans to introduce a new flag in [resolution strategy tuning](https://docs.gradle.org/current/userguide/resolution_strategy_tuning.html) to fail on inconsistent dependency resolution across sub-projects.
Currently we can only enforce versions for all configurations in a single project:
```groovy
java {
	consistentResolution {
		useCompileClasspathVersions()
	}
}
```

## Is there any solution?

Luckily, the official documentation regarding transitives covers a wide range of options.
I found a few approaches, including [custom plugins](https://github.com/nebula-plugins/nebula-dependency-recommender-plugin#54--transitive-dependencies).
Currently the recommended solution for sharing dependency versions across projects is **platform**.
Another solution, introduced as a feature preview in Gradle 7, is the `version catalogs`, but it doesn't offer so many [capabilities](https://docs.gradle.org/7.0/userguide/platforms.html#sub:platforms-vs-catalog).

Our goal is clear: we want the highest common version across all projects.
Does the *platform* seal the deal? Let's find out!

The whole idea of the [*platform*](https://docs.gradle.org/current/userguide/platforms.html) came from Maven's BOM (Bill of Materials), which is a centralized set of dependency versions that “work well together”.
This way we can define our first-level dependencies with just `<group>:<name>`, as long as they are defined in the *platform*.
```groovy
dependencies {  
	implementation platform(project(":platform"))
	implementation "<group>:<name>" 
}
```

This means we can use the same version all over the place. Sounds good!
Yet, there are some pitfalls... The major one being that it's not really solving our problem.

First, let's take a look at the example of first-level vs. transitive.
When the **lower** version is defined in the base project `project-b`, then despite defining `lib-x` as a first-level dependency in the *platform*, different versions are still resolved.
```
Platform
- lib-x 1.1 (first-level)
- lib-y
  - lib-x 1.2 (transitive)	

project-a
- Platform
- project-b
- lib-y
  - lib-x (version 1.2 resolved)

project-b
- Platform
- lib-x (version 1.1 resolved)
```

The *platform* is not magically making versions consistent within itself - the same resolution rules still apply.
To fix this and always use the version of the first-level dependency, we have to use `enforcedPlatform` instead:
```groovy
dependencies {  
	implementation enforcedPlatform(project(":platform"))
}
```

Now we have the same version across all sub-projects, but actually it is the **lower** one.
This means you can accidentally downgrade dependencies ;)
Yet we don't want to downgrade dependencies, but just align them.

Thus, we could consider two inspections here:
- listing places where `platform` is used instead of `enforcedPlatform`
- report dependencies that are resolved in a lower version than declared (the highest version should be OK, but it always better to upgrade related libraries with a lower version of transitive)

Another snag is the case of transitive vs. transitive.
Even with `enforcedPlatform` we still have to declare transitive dependency [explicitly to get rid of conflict](https://docs.gradle.org/current/userguide/dependency_constraints.html#sec:adding-constraints-transitive-deps).
And again we might accidentally downgrade both dependencies, so we have to be careful.

Therefore, enforcing the *platform* is not much different from using `force` in `resolutionStrategy`.
Both cases are still hard to maintain because we need additional reports of unused constraints or unintentional downgrades.

## Conclusions
The *platform* seems to be the ideal solution, but it isn't, especially for maintainers of dependencies.
Unexpected dependency conflicts still may occur, especially in multi-projects.
Thus, every dependency update needs to be done carefully and wisely. There is no easy way.
However, we can develop tools that could support us, like reports.
Let's hope that the future Gradle releases will provide us with warnings about such conflicts.
