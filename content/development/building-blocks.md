---
title: "Building Blocks"
date: 2018-03-08T16:30:11+01:00
description: "Explains the basic building blocks used in HAL. Read this page if you want to know more about the concepts and layers in HAL or if you plan to contribute to the codebase."
icon: "/img/puzzle.png"
toc: true
weight: 30
---
This section describes the building blocks and central concepts in HAL. The chapters follow a logical order starting from the fundamental concepts to the core and application level classes. 

# Dependency Injection

HAL uses [GIN](https://code.google.com/archive/p/google-gin/) for dependency injection. Although it is no longer actively developed, GIN is a stable and mature DI framework. In the long run we want to replace GIN with [Dagger](https://google.github.io/dagger/). But this requires a new GWTP version which currently depends on GIN.

Dependencies in GIN are declared in classes which extend `com.google.gwt.inject.client.AbstractGinModule`. Here's an example from the maven module `hal-config`: 

```java
@GinModule
public class ConfigModule extends AbstractGinModule {

    @Override
    protected void configure() {
        bind(Endpoints.class).in(Singleton.class);
        bind(Environment.class).in(Singleton.class);
        bind(Settings.class).in(Singleton.class);

        requestStaticInjection(Endpoints.class);
        requestStaticInjection(Settings.class);
    }

    @Provides
    public User providesCurrentUser() {
        return User.current();
    }
}
```

Most maven modules include their own GIN module. All GIN modules are annotated with `org.jboss.hal.spi.GinModule` and collected by the annotation processor `org.jboss.hal.processor.GinModuleProcessor`. This processor generates one composite GIN module which includes *all* other GIN modules. To see how it looks like open the class `org.jboss.hal.client.gin.CompositeModule` in your IDE.

Dependency injection via `javax.inject.Inject` is available in all classes which are bound in GIN modules, presenters, views and finder columns. We recommend to use constructor injection. 

# Logging

HAL uses the [SL4J](https://www.slf4j.org/) API for logging. Just declare a static logger instance in your class:

```java
public class Dispatcher implements RecordingHandler {

    @NonNls private static final Logger logger = LoggerFactory.getLogger(Dispatcher.class);
    
    ...
}
```

Log messages are printed to the browser console. In development mode the log level is set to `FINE`. But you can adjust the level using the request parameter `logLevel=<LEVEL>`, where level is one of the enum values from `java.util.logging.Level`: 

- SEVERE (highest value)
- WARNING
- INFO
- CONFIG
- FINE
- FINER
- FINEST (lowest value)

# Resources

HAL includes many different resources which are all available using the class `org.jboss.hal.resources.Resources`. The class implements the following interfaces: 

- `org.jboss.hal.resources.Ids`: IDs used in HTML elements and across multiple classes. The IDs defined here are reused by QA. 
- `org.jboss.hal.resources.Names`: Common names and technical terms which are not meant to be translated.
- `org.jboss.hal.resources.UIConstants`: UI related constants used in more than one place.
- `org.jboss.hal.resources.CSS`: Common CSS classes from HAL, PatternFly & Bootstrap. The constants in this interface are not involved in any kind of code generation or GWT magic. They're just collected in this interface to have them in one place.
- `org.jboss.hal.resources.Icons`: Collection of common icons.

In addition it has methods which return the following classes: 

- `org.jboss.hal.resources.Constants`: I18n constants.
- `org.jboss.hal.resources.Messages`: I18n messages which can contain variable parts.
- `org.jboss.hal.resources.Previews`: HTML snippets mainly used for the finder previews.
- `org.jboss.hal.resources.Images`: Collection of images used in HAL.
- `org.jboss.hal.resources.Theme`: Theme interface meant to be implemented by specific themes.

# Config

`org.jboss.hal.config.Environment`

# Flow

RxGWT

`org.jboss.hal.flow.Flow`

# DMR

operations, resource address, model nodes, ...

# Metadata

resource description, security context, capabilities, registries, first and second level cache

`org.jboss.hal.meta.StatementContext`

# Ballroom

core UI elements, Elemento, PatternFly 

# Core

`org.jboss.hal.core.CoreStatementContext`

`org.jboss.hal.core.CrudOperations`

model node form, table, finder

# MBUI

annotation processor

# MVP

presenter view, place manager, tokens 

