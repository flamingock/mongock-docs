---
title: 'Module Self Registration'
date: 2014-04-18 11:30:00
permalink: /v5/features/multi-module-support/index.html
toc: true
eleventyNavigation:
  version: v5
  order: 30
  parent: features
  key: features module self registration
  title: 'Module Self Registration'
---

# Module Self Registration

[[TOC]]

## Introduction

Mongock now supports a **modular approach** for managing migration packages.  
This allows each module in a multi-module project to be responsible for registering its own migration scan packages, enabling a decentralized configuration.

This feature works for both:

- **Annotation-based configuration** (in Spring Boot applications)
- **Builder-based configuration** (in Standalone or non-Spring environments)

---

## Why modular support?

In traditional setups, the migration scan packages are usually registered centrally, in a single configuration. This creates a coupling between modules and forces central coordination.

With this feature:

- Modules become more **autonomous**
- Migration logic can be kept **encapsulated** in each module
- Applications become more **scalable and maintainable**

---

## Usage

### 🟢 Spring Boot (with Annotations)

Mongock introduces the ability to define migration packages directly within a Spring `@Configuration` class using the annotations `@MongockScanPackage` or `@MongockScanPackages`.

#### Example:

```java
@Configuration
@MongockScanPackage({
  "com.package.example.migrations",
  "com.package.example2.migrations"
})
public class ModuleConfig {
}
```

To integrate this with the application, you can just let Spring know ModuleConfig or you can create a custom enabling annotation:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(ModuleConfig.class)
public @interface EnableModuleA {
}
```

Then use it in your application:
```java
@EnableModuleA
@EnableMongock
@SpringBootApplication
public class SomeApp {
  public static void main(String[] args) {
    SpringApplication.run(SomeApp.class, args);
  }
}
```
This allows each module to define its own migration paths independently.

### 🟡 Standalone (with Builder)
In standalone mode, Mongock introduces a new interface: ``MongockModuleConfig``.

Each module can implement this interface and register its migration packages through code.

Interface:

```java
public interface MongockModuleConfig {

    void configure(MongockScanPackage scan);
}
```

Example usage:

```java
public class ModuleAConfig implements MongockModuleConfig {

    @Override
    public void configure(MongockScanPackage scan) {
        scan.add("com.example.packageone.migrations");
        // or
        List<String> packages = Arrays.asList(Array.asList(
                "com.example.packageone.migrations", 
                "com.example.packagetwo.migrations"
        ));
        scan.addAll(packages);
    }
};
```

Register the module configuration:

```java
MongockStandalone.builder()
  .setDriver(driver)
  .addModuleConfig(new ModuleAConfig())
  .build();
```
This allows the migration setup to grow with your system without editing central files.

## Benefits

✅ Decoupled and clean modular architecture

✅ Easier to maintain and extend

✅ Enables plugin-like architectures

✅ Works with both Spring and standalone setups

## Notes

* This feature is fully backward compatible.
* Centralized configuration is still supported.
* Multiple module configs can be loaded in parallel.