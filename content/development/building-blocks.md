---
title: "Building Blocks"
date: 2018-03-08T16:30:11+01:00
description: "Explains the basic building blocks used in HAL. Read this page if you want to know more about the concepts and layers in HAL or if you plan to contribute to the codebase."
icon: "/img/puzzle.png"
markup: "mmark"
weight: 30
---
HAL consists of many different maven modules, each of which serves a specific purpose. Here's the list of modules and a quick description:

| Module | Description |
|--------|-------------|
| app | Main application containing the GWT [entry point](http://www.gwtproject.org/doc/latest/DevGuideCodingBasicsClient.html#creating) |
| ballroom | Core UI classes like `Form`, `Dialog` or `Table` |
| bom | Declaration of all HAL modules |
| config | Configuration classes for the environment, operation mode, current user and its roles |
| core | Core HAL API |
| db | Thin wrapper around [PouchDB](https://pouchdb.com/) |
| dmr | DMR related code to execute operation, read results and work with model nodes |
| docker | Docker image to run the console in [independent mode]({{< relref "documentation/get-started.md#independent-mode" >}}) |
| flow | Execute asynchronous tasks in order |
| fraction | WildFly Swarm fraction for HAL |
| js | JavaScript related helper classes |
| meta | Metadata related classes to encapsulate the different parts of the resource descriptions |
| parent-with-dependencies | Parent POM for all other modules except `bom` |
| parent-with-gwt | Parent POM for all GWT related modules |
| processors | Annotation processors for code generation |
| resources | I18n resources, images and HTML snippets |
| spi | SPI related classes and annotations |
| standalone | Local [Undertow](http://undertow.io/) server to start the console in [independent mode]({{< relref "documentation/get-started.md#independent-mode" >}}) |
| testsuite | Maven setup to assemble classes from different modules and make them available as one dependency for the test suite |
| themes | Different HAL themes |
|     eap | Theme used for [JBoss EAP](https://developers.redhat.com/products/eap/overview/) |
|     hal | Theme used for the [independent mode]({{< relref "documentation/get-started.md#independent-mode" >}}) |
|     wildfly | Theme used for [WildFly](http://wildfly.org/) |
| yarn | NPM / Yarn setup to start the console in [independent mode]({{< relref "documentation/get-started.md#independent-mode" >}}) |
  
  
Building blocks - Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum.
 
Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.
