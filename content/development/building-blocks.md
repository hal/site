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

This class is a good starting point to find out which classes are available for dependency injection. To use dependency injection in your classes use the annotation `javax.inject.Inject`. All classes bound in GIN modules, presenters, views and finder columns are available for dependency injection. We recommend to use constructor injection. 

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
- supported locales

Some information is only available after the console has been fully loaded: 

- product name / version and release name / version
- operation mode (standalone or domain)
- name of the domain controller
- management version
- access control provider (simple or rbac)
- standard and scoped roles

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

- The descriptions are used for the context help.
- The data type and flags like `required` are used to build the form items.
- The capabilities are used to build type-ahead combo boxes.
- The security related information is used to filter form items and disable / enable buttons.

The metadata must be present, before the UI is setup. That's why central classes like presenter proxies and finder columns can be annotated with `org.jboss.hal.spi.Requires`. This information is parsed by an annotation processor and made available as an implementation of `org.jboss.hal.meta.resource.RequiredResources`. Here's an example of the batch presenter:

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

Complex presenters may have a lot of required resources and reading them takes some time. That's why HAL stores the metadata in different registries and caches. There's a first level cache with a limited number of entries which lives in the memory and there's a second level cache which is based on [PouchDB](https://pouchdb.com/) and which is stored in the browser local storage.

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

In domain mode with host 'master' and server 'server-one' selected, this results in the address
`/host=master/server=server-one/subsystem=elytron`.

# Ballroom

The ballroom module contains the core UI elements. This includes many different components from PatternFly like alerts, charts, dialogs, forms, list views, tables and wizards. Most of them are created using [Elemento's](https://github.com/hal/elemento) fluent API and can be seen as [custom elements](https://github.com/hal/elemento#custom-elements). In this context custom elements are a composite of HTML elements and/or other custom elements. They're ordinary classes which can hold state or register event handlers. The only requirement is to implement `IsElement<E extends HTMLElement>` and return the root element: 

```java
public class Tabs implements IsElement<HTMLElement> {

    private final HTMLElement root;
    private final HTMLElement tabs;
    private final HTMLElement panes;
    private final Map<Integer, String> indexToId;
    private final Map<String, HTMLElement> paneElements;

    public Tabs(String id) {
        root = div().id(id)
                .add(tabs = ul()
                        .css(nav, navTabs, navTabsPf, navTabsHal)
                        .attr(UIConstants.ROLE, UIConstants.TABLIST)
                        .asElement())
                .add(panes = div().css(tabContent).asElement())
                .asElement();

        indexToId = new HashMap<>();
        paneElements = new HashMap<>();
    }

    @Override
    public HTMLElement asElement() {
        return root;
    }

    // more methds...
}
```

# CRUD Operations

The class `org.jboss.hal.core.CrudOperations` contains generic CRUD methods which can be roughly divided into the following categories:

- Add a (singleton) resource using a generic add resource dialog
- Add a (singleton) resource using an operation
- Read a resource (recursively)
- Read child resources
- Update a (singleton) resource using a change set (key/value map)
- Update a (singleton) resource using an operation
- Reset a (singleton) resource
- Remove a (singleton) resource
 
All in all there are over 60 different methods. Some methods just execute the underlying DMR operations, other methods also interact with the user by showing (confirmation) dialogs. Some methods use address templates, some resource addresses, some return RX types others accept callbacks.  

# Finder

The finder is a central UI component in HAL. Each top level category (except the homepage) uses the finder for navigation. The finder manages a number of columns and one preview pane. In HAL there's only one finder instance which is used across the different top level categories. 

Finder columns have an unique ID, a title, a number of optional actions and an item renderer which defines how the items of this column are rendered. All items of a column must be of the same type. Columns are self-contained and should not have references to other columns (only by ID).

If you want to implement your own column follow these steps:

1. Extend from `org.jboss.hal.core.finder.FinderColumn<T>`. 
1. Annotate your class with `org.jboss.hal.spi.Column` resp. `org.jboss.hal.spi.AsyncColumn`. 
1. Create an instance of `org.jboss.hal.core.finder.FinderColumn.Builder` and pass it to `super()`. The builder requires three mandatory parameters: 
    1. an instance to the finder instance
    1. an unique ID
    1. the column title
1. Use the builder to define additional column attributes: 
    - column actions
    - an implementation for `org.jboss.hal.core.finder.ItemsProvider<T>` 
    - an implementation for `org.jboss.hal.core.finder.ItemRenderer<T>` and `org.jboss.hal.core.finder.ItemDisplay<T>`
    - a custom item preview
    - the ID for the next column

Here's a complete example of a simple column which is used in the configuration section to manage the interfaces:

```java
@Column(Ids.INTERFACE)
@Requires("/interface=*")
public class InterfaceColumn extends FinderColumn<NamedNode> {

    @Inject
    public InterfaceColumn(Finder finder,
            ColumnActionFactory columnActionFactory,
            ItemActionFactory itemActionFactory,
            Places places,
            Dispatcher dispatcher,
            CrudOperations crud) {

        super(new Builder<NamedNode>(finder, Ids.INTERFACE, Names.INTERFACE)
                .columnAction(columnActionFactory.add(
                        Ids.INTERFACE_ADD,
                        Names.INTERFACE,
                        InterfacePresenter.ROOT_TEMPLATE,
                        singletonList(INET_ADDRESS)))
                .columnAction(columnActionFactory.refresh(Ids.INTERFACE_REFRESH))
                .itemsProvider((context, callback) -> crud.readChildren(ResourceAddress.root(), INTERFACE,
                        result -> callback.onSuccess(asNamedNodes(result))))
                .useFirstActionAsBreadcrumbHandler()
                .onPreview(item -> new InterfacePreview(item, dispatcher, places))
        );

        setItemRenderer(item -> new ItemDisplay<NamedNode>() {
            @Override
            public String getTitle() {
                return item.getName();
            }

            @Override
            public List<ItemAction<NamedNode>> actions() {
                return asList(
                        itemActionFactory.view(NameTokens.INTERFACE, NAME, item.getName()),
                        itemActionFactory.remove(Names.INTERFACE, item.getName(), InterfacePresenter.ROOT_TEMPLATE,
                                InterfaceColumn.this));
            }
        });
    }
}
```

HAL provides many factories and helper classes like `org.jboss.hal.core.finder.ColumnActionFactory` and `org.jboss.hal.core.finder.ItemActionFactory` which simplify adding common column and item actions. 

# MVP

HAL uses [GWTP](https://dev.arcbees.com/gwtp/) for its MVP implementation. At the heart of GWTP is a model-view-presenter architecture (MVP). Although this model [has been lauded as one of the best approaches to GWT development](https://youtu.be/PDuhR18-EdM), it is still hard to find an out-of-the-box MVP solution that supports all the requirements of modern web apps. GWTP aims to provide such a solution while reducing the amount of code required to reach it.

For example, adding history management and code splitting to your presenter is as simple as adding these lines to your class:

```java
@ProxyCodeSplit
@NameToken("datasource-configuration")
public interface MyProxy extends ProxyPlace<DataSourcePresenter> {
}
```
                                                                           
GWTP uses GWT's event bus in a clear and efficient way. Events are used to decouple loosely related objects, while program flow between strongly coupled components is kept clear using direct method invocations. The result is an easy to understand and debug application that can continually scale up.

The goal of GWTP is to offer an easy-to-use MVP architecture with minimal boilerplate, without compromising GWT's best features. Here are some of the core features of GWTP:

- Dependency injection through GIN
- Simple and powerful history management mechanism
- Lifecycle events to manage presenters
- Lazy instantiation for presenters and views
- Effortless and efficient code splitting
- Bootstrap tools to make the creation of new GWT applications dead simple.

{{< imgflow src="/img/development/mvp-classes.png" float="right" >}}
HAL adds a thin layer on top of GWTP: Every presenter in HAL extends from `org.jboss.hal.core.mvp.HalPresenter`. This class implements central behaviour like integration with the header. It also ensures that every view implements `org.jboss.hal.core.mvp.HalView`. This interface is an adapter between GWTP views which are based on GWT widgets and HAL views which are based on HTML elements. 

Presenters in HAL don't extend from `HalPresenter` directly though. Most presenters either extend from `FinderPresenter` or `ApplicationFinderPresenter`. 

In addition there are several interfaces which are implemented by the various presenters and which add specific features:

- `HasFinderPath`: Implemented by application presenters which interact with the finder.
- `SupportsExpertMode`: Interface meant to be implemented by presenters which support switching to an 'expert mode' using the model browser
- `SupportsExternalMode`: Tagging interface meant to be implemented by presenters which can be opened in an external browser window / tab.   
{{</ imgflow >}}

# MBUI

Many application screens in HAL which are used to configure resources follow a very similar layout: 

- an optional vertical navigation at the left
- a header and an optional description (often taken from the resource description)
- an optional table showing the resources
- one or several tabs holding the forms to view and modify the resource

In addition also the behaviour is very similar. The table has buttons to add and remove resources, the forms have links to view, modify or reset resources. 

HAL can generate presenter and view implementations described by MBUI XML files. MBUI stands for **m**odel **b**ased **u**ser **i**nterface. The XML files use the [Relax NG](http://www.relaxng.org/) schema defined by [`MbuiView.rng`](https://raw.githubusercontent.com/hal/console/develop/spi/src/main/resources/org/jboss/hal/spi/MbuiView.rng). The code generation is triggered by an annotation processor.

To implement a MBUI presenter / view you need the following compilation units:

1. presenter which extends `MbuiPresenter<V extends MbuiView, Proxy_ extends ProxyPlace<?>>`
1. abstract view annotated with `MbuiView` which implements `MbuiView<P extends MbuiPresenter>`
1. MBUI XML file which defines the UI

The following code snippets show the path presenter / view tuple which uses MBUI to define the UI *and* common behaviour:

**PathsPresenter.java**

```java
public class PathsPresenter extends MbuiPresenter<PathsPresenter.MyView, PathsPresenter.MyProxy> {

    @Inject
    public PathsPresenter(EventBus eventBus, MyView view, MyProxy proxy, Finder finder, CrudOperations crud) {
        super(eventBus, view, proxy, finder);
        this.crud = crud;
    }

    // business methods omitted 

    @ProxyCodeSplit
    @Requires("/path=*")
    @NameToken(NameTokens.PATH)
    public interface MyProxy extends ProxyPlace<PathsPresenter> { }

    public interface MyView extends MbuiView<PathsPresenter> {
        void update(List<NamedNode> paths);
    }
}
```

**PathsView.java**

```java
@MbuiView
public abstract class PathsView extends MbuiViewImpl<PathsPresenter> implements PathsPresenter.MyView {

    public static PathsView create(final MbuiContext mbuiContext) {
        return new Mbui_PathsView(mbuiContext);
    }

    @MbuiElement("path-table") Table<NamedNode> table;
    @MbuiElement("path-form") Form<NamedNode> form;

    PathsView(MbuiContext mbuiContext) {
        super(mbuiContext);
    }

    @Override
    public void update(List<NamedNode> paths) {
        form.clear();
        table.update(paths);
    }
}
```

**PathsView.mbui.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<view>
    <metadata address="/path=*">
        <h1>Paths</h1>
        <p>${metadata.getDescription().getDescription()}</p>
        <table id="path-table" form-ref="path-form">
            <actions>
                <action handler-ref="add-resource">
                    <attributes>
                        <attribute name="name"/>
                        <attribute name="path"/>
                        <attribute name="relative-to" suggest-handler="${new PathsAutoComplete()}"/>
                    </attributes>
                </action>
                <action handler-ref="remove-resource" scope="selected" name-resolver="${table.selectedRow().getName()}"/>
            </actions>
            <columns>
                <column name="name"/>
                <column name="path"/>
            </columns>
        </table>
        <form id="path-form" auto-save="true" reset="true" name-resolver="${form.getModel().getName()}">
            <attributes>
                <attribute name="name"/>
                <attribute name="path"/>
                <attribute name="relative-to" suggest-handler="${new PathsAutoComplete()}"/>
            </attributes>
        </form>
    </metadata>
</view>
```
