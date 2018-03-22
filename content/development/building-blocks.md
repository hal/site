---
title: "Building Blocks"
date: 2018-03-08T16:30:11+01:00
description: "Explains the basic building blocks used in HAL. Read this page if you want to know more about the concepts and layers in HAL or if you plan to contribute to the codebase."
icon: "/img/puzzle.png"
toc: true
weight: 30
---
This section describes the main building blocks and central concepts in HAL. The chapters follow a logical order starting from the fundamental concepts to the higher-level modules. 

# Dependency Injection

HAL uses [GIN](https://code.google.com/archive/p/google-gin/) for dependency injection. Although no longer actively developed, GIN is a stable and mature DI framework. In the long run we want to replace GIN with [Dagger](https://google.github.io/dagger/). But this requires a new GWTP version which currently depends on GIN.

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

Most maven modules in HAL have their own GIN module. All GIN modules are annotated with `org.jboss.hal.spi.GinModule` and are collected by an annotation processor. This processor generates one composite GIN module which includes *all* other GIN modules: 

```java
/*
 * WARNING! This class is generated. Do not modify.
 */
@Generated("org.jboss.hal.processor.GinModuleProcessor")
public class CompositeModule extends AbstractGinModule {

    @Override
    protected void configure() {
        install(new org.jboss.hal.resources.ResourcesModule());
        install(new org.jboss.hal.config.ConfigModule());
        ...
    }
}
```

To use dependency injection in your classes use the annotation `javax.inject.Inject`. All classes bound in GIN modules, presenters, views and finder columns are available for dependency injection. We recommend to use constructor injection. 

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
- `org.jboss.hal.resources.CSS`: Common CSS classes from HAL, PatternFly & Bootstrap. The constants in this interface are not involved in any kind of code generation or GWT magic. They're collected in this interface in order to easily detect unused CSS classes. 
- `org.jboss.hal.resources.Icons`: Collection of common icons.

In addition it has methods which return the following classes: 

- `org.jboss.hal.resources.Constants`: I18n constants.
- `org.jboss.hal.resources.Messages`: I18n messages which can contain variable parts.
- `org.jboss.hal.resources.Previews`: HTML snippets mainly used for the finder previews.
- `org.jboss.hal.resources.Images`: Collection of images used in HAL.
- `org.jboss.hal.resources.Theme`: Theme used in HAL.

# Config

Information about the console and its environment is available using the interface `org.jboss.hal.config.Environment`. It provides the following data:

- HAL version
- HAL build (community or product, ie. WildFly or JBoss EAP)
- Supported locales

Some information is only available after the console has been fully loaded: 

- Instance info containing the product name and version and the release name and version
- Operation mode (standalone or domain)
- Name of the domain controller
- Management version
- Access control provider (simple or rbac)
- Standard and scoped roles

Another important building block is the interface `org.jboss.hal.config.Endpoints`. It provides access to the endpoints used in HAL. Finally there's the class `org.jboss.hal.config.Settings` which provides access to the user settings. 

# Flow

HAL runs in the browser. As such it uses a lot of (asynchronous) callbacks. We all know that nested callbacks are hard to read, maintain and quickly lead to an anti pattern better known as callback hell.

One solution for this problem is to use reactive programming. Instead of using callbacks as arguments, methods return reactive classes like `Single<T>` or `Observable<T>`. With [RxGWT](https://github.com/intendia-oss/rxgwt) there's an implementation of RxJava available for GWT. At the moment it's still based on RxJava 1.x, but support for 2.x is in the works. 

HAL uses a reactive programming model in many classes to get rid of callbacks. You can find examples in `org.jboss.hal.dmr.dispatch.Dispatcher` and `org.jboss.hal.core.CrudOperations`. This really pays off when you want to execute several asynchronous operations in order. Therefore you can use the methods in `org.jboss.hal.flow.Flow` and the task interface `org.jboss.hal.flow.Task`.   

Say you want to add a system property only if it's not already there. You could implement this using the following code snippet:

```java
Logger logger = ...;
Dispatcher dispatcher = ...;
ResourceAddress address = ResourceAddress.from("/system-property=foo");

Task<FlowContext> check = context -> {
    return dispatcher.execute(new Operation.Builder(address, READ_RESOURCE_OPERATION).build())
            .doOnSuccess(result -> context.push(200))
            .doOnError(exception -> context.push(404))
            .toCompletable();
};
Task<FlowContext> add = context -> {
    int status = context.pop();
    return status == 200 
            ? Completable.complete() 
            : dispatcher.execute(new Operation.Builder(address, ADD).build()).toCompletable();
};

Flow.series(new FlowContext(Progress.NOOP), check, add).subscribe(new Outcome<FlowContext>() {
        @Override
        public void onError(FlowContext context, Throwable error) {
            logger.error("Unable to create system property 'foo': {}", error.getMessage());
        }

        @Override
        public void onSuccess(FlowContext flowContext) {
            logger.info("System property 'foo' successfully created.");
        }
});
``` 

# DMR

This section assumes you're familiar with the basic concepts of the WildFly management model. If not please read the [admin guide](http://docs.wildfly.org/Admin_Guide.html) in the WildFly documentation.

The communication with the management endpoint, heavily relies on the [detyped management representation](http://docs.wildfly.org/Admin_Guide.html#Detyped_management_and_the_jboss-dmr_library) as defined in [JBoss DMR](https://github.com/jbossas/jboss-dmr). Due to restrictions in GWT (no threading, no IO) HAL comes with its own fork of JBoss DMR. It's a clone of the original code without all the pieces which don't make sense and don't work in GWT. 

HAL adds a thin layer of more strongly typed classes on top of that. They all extend from `org.jboss.hal.dmr.ModelNode` so you don't lose the flexibility, but are more specific so that the the code becomes more readable. 

**Operation**\
Represents a DMR operation like `read-resource` or `add`. An operation always requires a resource address and the actual operation name. Optionally you can add parameters as key/value pairs. Operations should be built using the builder `org.jboss.hal.dmr.Operation.Builder`:

```java
Operation operation = new Operation.Builder(address, "read-log-file")
        .param("lines", 100)
        .param("tail", true)
        .build();
```

**Composite**\
Represents a composite operation consisting of several operations. When executed by the dispatcher you get back an instance of `org.jboss.hal.dmr.CompositeResult`. Use this class to easily access the different step results by index or name:

```java
List<Operation> operations = ...;
dispatcher.execute(new Composite(operations), (CompositeResult result) -> {
    List<Property> list0 = result.step(0).get(RESULT).asPropertyList();
    List<Property> list1 = result.step(1).get(RESULT).asPropertyList();
    ...
});
```

**ResourceAddress**\
Represents a fully qualified DMR address ready to be put into a DMR operation. The address consists of 0-n segments
 with a name and a value for each segment. Implemented by `org.jboss.hal.dmr.ResourceAddress`. 

**ModelNodeHelper**\
Contains static helper methods related to model nodes. Some of them deal with reading deeply nested model nodes using a path seperated by "/":

```java
Dispatcher dispatcher = ...;
ResourceAddress address = ResourceAddress.from("/subsystem=jca");
Operation operation = new Operation.Builder(address, READ_RESOURCE_OPERATION)
        .param(RECURSIVE, true)
        .build();
dispatcher.execute(operation, result -> {
    ModelNode beanValidation = ModelNodeHelper.failSafeGet("bean-validation/bean-validation");    
});
```

**Dispatcher**\
Executes operations against the management endpoint. You can execute normal and composite operations. There are signatures which use callbacks and RX variants which return `Single<CompositeResult>` resp. `Single<ModelNode>`. 

```java
dispatcher.execute(new Operation.Builder(ResourceAddress.root(), READ_RESOURCE_OPERATION).build())
        .subscribe(payload -> logger.info("Root resource: {}", payload));

dispatcher.execute(new Operation.Builder(ResourceAddress.root(), READ_RESOURCE_OPERATION).build(), 
        payload -> logger.info("Root resource: {}", payload));
```

{{< callout warning >}}
**Payload != Response**

Whether you use a callback or an RX type, please note that the model node passed to the callback resp. used in the RX type is *not* the full response, but just the `result` part of what you normally get when executing operations in the CLI:

```
[standalone@localhost:9990 /] :read-resource
{
    "outcome" => "success",
    "result" => {
        "management-major-version" => 6,
        "management-micro-version" => 0,
        "management-minor-version" => 0,
        "name" => "hpehl-macbook",
        ...
    }
}
```
{{</ callout >}}

# Metadata

HAL heavily relies on metadata from the resource descriptions. This data is used when building the user interface in many different ways:

- the descriptions are used for the context help
- the data type and flags like `required` are used to build the form items
- the capabilities are used to build type-ahead combo boxes
- the security related information is used to filter form items and disable / enable buttons

The resource descriptions must be present, before the UI is setup. That's why central classes like presenter proxies and finder columns can be annotated with `org.jboss.hal.spi.Requires`. This information is parsed by an annotation processor and made available as an implementation of `org.jboss.hal.meta.resource.RequiredResources`. Here's an example of the batch presenter:

```java
public class BatchPresenter extends MbuiPresenter<BatchPresenter.MyView, BatchPresenter.MyProxy>
        implements SupportsExpertMode {
        
    ...
    
    @ProxyCodeSplit
    @NameToken("batch-jberet-configuration")
    @Requires("/{selected.profile}/subsystem=batch-jberet")
    public interface MyProxy extends ProxyPlace<BatchPresenter> {}
}
```

When the user navigates to http://localhost:9990/#batch-jberet-configuration the following steps are executed:

1. The required resources for the name token "batch-jberet-configuration" are looked up from the registry.
1. The first and second level cache are checked whether the metadata is already present.
1. If not present, the DMR operation `/subsystem=batch-jberet:read-resource-description` is executed.
1. The response is parsed and stored in the registries and caches. 

Complex presenters may have a lot of required resources and reading them takes some time. That's why HAL stores the meta data in different registries and caches. There's a first level cache with a limited number of entries which lives in the memory and there's a second level cache which is based on [PouchDB](https://pouchdb.com/) and which is stored in the browser local storage.

Use the class `MetadataRegistry` to get metadata for a given address template. The class `Metadata` is an umbrella around the different parts of the resource description: descriptions, attribute metadata, security context and capabilities. An `AddressTemplate` is like a resource address, but can contain variable parts:
 
- `{domain.controller}`: The name of the domain controller
- `{selected.profile}`: The selected profile 
- `{selected.group}`: The selected server group
- `{selected.host}`: The selected host
- `{selected.server-config}`: The selected server-config
- `{selected.server}`: The selected server

The values for these variables are stored in class `StatementContext` and updated as the user navigates through the console. The address template class has methods to resolve an template into a real resource address:

```java
StatementContext statementContext = ...;
AddressTemplate template = AddressTemplate.of("{selected.host}/{selected.server}/subsystem=elytron");
ResourceAddress address = template.resolve(statementContext);
```  

# Ballroom

Core UI elements, Elemento, PatternFly 

# Core

`org.jboss.hal.core.CoreStatementContext`

`org.jboss.hal.core.CrudOperations`

model node form, table, finder

# MBUI

annotation processor

# MVP

presenter view, place manager, tokens 

