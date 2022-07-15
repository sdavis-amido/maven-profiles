# Java Dependency Management using Maven Profiles

How to get a similar outcome in Java as is available using .Net conditional compilation to reduce
code dependencies.

## .Net Conditional Compilation

It should be noted that .Net conditional compilation uses a pre-processor that is invoked against
code templates. These templates contain code lines that a relevant for a number of scenarios all at
once,
and the lines that are emitted for compilation are done so via properties passed to the
pre-processor.

An example in C# is as follows:

```
#if debug
    Console.WriteLine("Debug build");
#elif VC7
    Console.WriteLine("Visual Studio 7");
#endif
```

If the conditional compilation process is passed the `debug` flag then the first `Console.WriteLine`
would
be included in the packaged code. If the `VC7` flag were passed instead then the second line of code
would be
used.

## Project Introduction

This small project (deliberately kept minimalistic) shows how a Java Spring Boot application can be
made to only import specified dependencies (or modules) at runtime and build time.

Spring is an inversion of control (IoC) container which works at the object level - it manages
objects that
are instances of Java classes (by default singleton instances),
using a nomenclature of 'beans'. A bean is an object that is instantiated, assembled, and otherwise
managed by a Spring IoC container. Whereas .Net conditional compilation
is able to work at the individual code line level, this is not possible in Java & Spring, however it
is
possible though good design to achieve approximately the same end goal.

NOTE: It should be pointed out that if the end-goal is to end up with project code (or `pom.xml`)
that has zero reference to a component if it's not wanted then the only way to achieve this would
be to physically remove those dependencies from the project using post-processing text techniques (
in a similar way to the way that .Net does it for conditional compilation) - see the section below
related to config file post-processing.

This example code uses a combination of Maven Profiles and Spring Boot profiles. The former is used
to import code dependencies and the latter is used to only define (or override) properties if a
specific profile is active. It's not strictly necessary to combine the two (either could be used in
isolation) but it does provide better separation of concerns.

## Maven Profiles

Maven profiles are defined in the `pom.xml` of the project.

## Spring Profiles

Spring Profiles helps to easily set right the configuration whether that be for a deployment
environment (such as dev, prod etc.) or through user selection.

# Executing the code

There are 4 levels of dependencies provided:

1. Global dependencies - always included
2. Generic dependencies - included by default (or "generic" explicitly specified) unless another
   profile explicitly specified
3. Spring Actuator dependencies - included if profile "with-actuator" is explicitly specified
4. Spring Security dependencies - included if profile "with-security" is explicitly specified

## Running with just global dependencies

```bash
$ mvn clean spring-boot:run -P -generic

$ mvn clean package -P -generic
```

## Running with global dependencies & Generic dependencies (not explicitly specified)

```bash
$ mvn clean spring-boot:run 

$ mvn clean package
```

## Running with global dependencies & Generic dependencies (explicitly specified)

```bash
$ mvn clean spring-boot:run -P generic 

$ mvn clean package -P generic
```

## Running or building with global & Spring Actuator dependencies

```bash
$ mvn clean spring-boot:run -P with-actuator 

$ mvn clean package -P with-security
```

## Running or building with global, Generic & Spring Actuator dependencies

```bash
$ mvn clean spring-boot:run -P generic,with-actuator 

$ mvn clean package -P generic,with-security
```

## Running or building with global, Spring Actuator and Spring Security dependencies

```bash
$ mvn clean spring-boot:run -P with-security,with-actuator 

$ mvn clean package -P with-security,with-actuator
```

# Config File Post-processing

To post-process the XML and permanently remove dependencies one option is to use a utility such
as `xmlstarlet`.

The example below removes all references to anything pulled in by `with-actuator`, in this example
the
Spring Boot Actuator libraries.

```bash
# Install the utility 
$ brew install xmlstarlet

# Permanently remove the 'with-actuator' references (property and maven profile)
$ xmlstarlet edit -N ns='http://maven.apache.org/POM/4.0.0' \
      --delete './/ns:project/ns:properties/ns:with-actuator.profile.name' \
      --delete './/ns:project/ns:profiles/ns:profile[ns:id="with-actuator"]' \
      pom.xml        
```

For completeness, it would also be necessary to remove the relevant Spring profile include
statements - these are
the
lines at the top of the `resources/application.yml` that look
like `- @with-actuator.profile.name@`

```yaml
spring:
  profiles:
    include:
      - @generic.profile.name@
      - @with-actuator.profile.name@         ### Remove this line, one option is to use 'sed'
      - @with-security.profile.name@
```

and also to delete the features (aka profile) properties defined in its
own `resources/application-with-actuator.yml` file.