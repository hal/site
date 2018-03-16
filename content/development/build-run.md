---
title: "Build & Run"
date: 2018-03-08T16:29:52+01:00
description: "Explains how to build and run the console. Talks about the prerequisites and what it necessary to debug the codebase."
icon: "/img/tools.png"
toc: true
weight: 10
---
This page explains how to build, run & debug the console.

# Build

If not already done, clone the code from https://github.com/hal/hal.next/ or [fork](https://github.com/hal/hal.next/fork) the repository into your own personal GitHub account. 

HAL uses Maven for its build. Execute the following command to build the complete codebase including the GWT application:

```bash
mvn clean install
``` 

Note that this might take some time. If you just want to make sure that there are no compilation or test failures use

```bash
mvn clean install -Dgwt.skipCompilation
``` 

# Run

Pending

# Debug

Pending