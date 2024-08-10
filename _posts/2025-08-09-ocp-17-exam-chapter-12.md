---
title: OCP 17 Exam - chapter 12 notes - modules
categories: [ Book notes ]
tags: [ ocp17, java ]
---

> Note:
> Java Platform Module System was introduced in JDK 9 in project [Jigsaw](https://openjdk.org/projects/jigsaw/spec/).
> In Java 25 modules would be available
> through [imports like regular classes](https://www.baeldung.com/java-25-features#2-module-import-declarations-jep-511--preview).
> {: .prompt-tip }

## Running and packaging simple module

Folder structure of `app`:

```
- app
  - src/pkg
    - App.java // prints hello world
  - module-info.java // only module app { }
```

> Package names must be unique: e.g. I cannot add class in package "java.util" as already exists in module "java.base".
> {: .prompt-warning }

error: duplicate module on application module path
module in products

Run below commands in parent directory <https://github.com/RG9/rg-playground-ocp17/blob/main/java-modules-demo>.

Compile to `app/out`:

```shell
$ javac --module-path mods -d app/out app/src/pkg/*.java app/module-info.java
```

> Note: --module-path is not needed now as "app" module do not have any dependencies, but good to know that parameter
> {: .prompt-info }

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

> `-p` is abbreviation of `--module-path`,
> `-m` is abbreviation of `--module`
> {: .prompt-info }

> Be aware of format of `--module` param, which should be separated by slash: `-m module/class`
> {: .prompt-warning }

> Note: we don't have to export any package to be able to run class with main method.
> {: .prompt-info }

> Side note:
> Java module system is different than one in Intellij!
> See [1](https://www.jetbrains.com/help/idea/creating-and-managing-modules.html#modules-idea-java)
> and [2](https://blog.jetbrains.com/idea/2017/03/support-for-java-9-modules-in-intellij-idea-2017-1/).
> Instead of creating a project, you should just "Open" parent directory.
> {: .prompt-tip }

## Simple dependency on other module with "requires" and automatic module

- if we want to export all packages and we don't require any module, then there is no need to define `module-info.java`

> "automatic module" is jar on module path that does not have `module-info.java` and exports all packages
> {: .prompt-info }

- automatic module name will be extracted from jar name, e.g. `products-0.1-SNAPSHOT.jar` becomes `products` module

- in app's `module-info.java` add `requires products;` and build again

## ServiceLoader that "uses" SPI and Service Provider that "provides-SPI-with-IMPL"

Let's say we want to display products (SPI - Service Provider Interface) from many different suppliers/manufactures (provider/impl).
We can load products as service to avoid building the whole application when new supplier will be added.

- in app code load `Products` SPI via `ServiceLoader` (note: no need to rebuild `products` module):
```java
	private static Products getProductsInstance() {
		return ServiceLoader.load(Products.class)
			.findFirst()
			.orElseGet(() -> getDefaultProductsInstance());
	}
```
> note: `ServiceLoader#load` returns `ServiceLoader` which is `Iterable`, so provides other methods like `stream()`, `forEach()`
> {: .prompt-info }

- in app's `module-info.java` add `uses products.Products;`

> w/o `uses` we cannot compile `ServiceLoader`:
>  java.util.ServiceConfigurationError: products.Products: module app does not declare `uses``
> {: .prompt-warning }

> another keyword is `opens` that allows reflection
> {: .prompt-info }


> Remember to end every statement with `;` in `module-info.java`
> {: .prompt-tip }

OK, this was service loader/client part, next add service provider that will provide implementation.

- let's define module `products.pl` with class:
```java
public class PolishProducts implements Products {

	@Override
	public List<Products.Product> findAll() {
		return List.of(
			() -> "Pierogi",
			() -> "Żubrówka"
		);
	}
}
```

- in `module-info.java` needs to do 2 things:
  - `requires products;` as we depend on the SPI interface
  - `provides products.Products with pl.products.PolishProducts;` - declare that `Products` SPI will be provided with `PolishProducts` implementation

## Migration strategies

- **Top down**: add all JARs to module path as "automatic" modules, then migrate one module at a time to be "named" module.
- **Bottom up**: all unmigrated "unnamed" modules stays on class path. Pick next module with the least dependencies and migrate it.

> unnamed module is not visible to other modules on the module path, because module on module path cannot refer to classpath.
> {: .prompt-info }

## Tricky review questions

- An unnamed module doesn't use a module-info.java file.

- module was defined with keyword class!! "class dragon {"

- No modules need to specify requires on the service provider since that is the implementation.

## Playground code

<https://github.com/RG9/rg-playground-ocp17/blob/main/java-modules-demo>

----

Credit: [OCP Oracle Certified Professional Java SE 17 Developer Study Guide](https://www.selikoff.net/ocp17)
