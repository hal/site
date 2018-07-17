---
title: "Protect Wildfly HTTP Management Interface with SSL"
date: 2018-07-12T15:30:25+02:00
description: "How to use the new SSL Wizard to enable SSL for the HTTP management interface."
tags:
- configuration
- security
---
In WildFly 13, there is a wizard to enable the SSL for the HTTP management interface, it uses the elytron resource to manage the security features. The certificate may be handled in the following ways:

- Generate a self-signed certificate
- Use an existing certificate file
- Use an existing elytron key-store
- Use mutual authentication with a client

You can see more information about [elytron features](http://docs.wildfly.org/13/WildFly_Elytron_Security.html) and [HTTPS in general](https://www.instantssl.com/ssl-certificate-products/https.html).

The wizard may create the following resources to add SSL to the HTTP Management Interface:

- /subsystem=elytron/key-manager
- /subsystem=elytron/key-store
- /subsystem=elytron/server-ssl-context

For domain mode, the managed resources are under `/host=* address`.

> Note: For domain mode, this wizard works only on the HTTP Management interface of a domain controller, as there is not a HTTP Interface on a slave host controller.

Then the `/core-service=management/management-interface=http-interface` is modified to allow the SSL to work.

> **It is strongly recommended to first experiment this new feature on a non-production system.**

## Navigate to the HTTP Management Interface

For domain mode you can navigate to `Runtime` top level menu, select the `Hosts`, then click on the `View` button of the domain host controller.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-01.png" >}}

For standalone mode you can navigate to `Runtime` top level menu, then click on the `View` button of the server's column.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-01a.png" >}}

Open the Management Interface -> HTTP.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-02.png" >}}

The HTTP Management Interface view

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-03.png" >}}

## Launch the Enable SSL wizard

Click on the button `Enable SSL` located on the right of the screen. This will display the following modal dialog, for you to choose the options:

> The mutual authentication, enabled a trust relationship between the browser client and the https management endpoint. You must provide a client certificate for this to work.

Then you must select which strategy related to the certificate store, the description is self explanatory.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-04.png" >}}

For this tutorial, the selected option is to create a self-signed certificate.

After you choose the initial settings, fill in the required values as show with an asterisk. There are default values for testing purposes.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-04a.png" >}}

Then click `Next`, the following screen asks for confirmation to submit the values and create the required resources.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-05.png" >}}

After the user confirms, the resources are created and if successful, it displays the following screen.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-06.png" >}}

At this point, there are two options:

1. The user clicks on the `Reload Domain Controller` button: This will reload the domain host controller (but will not restart the servers under it), a new link is presented to the user to reload the browser to the new https address.
2. The user clicks on the `Close` button: The host domain controller will remain in `reload host` state, but will continue to work in an unsecure http protocol. The user must later reload the host domain controller for the changes to take effect.

* Also note that, if the certificate is not signed by a trusted certificate authority, upon the first https access, the browser will present a warning about this untrusted certificate.

## Using the HTTPS Management Interface

As you can see, after the first access, the https certificate used in the browser:

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-08.png" >}}

And the https certificate used in the CLI:

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-09.png" >}}


## Disable the HTTPS Management Interface

The user may also disable the SSL on the HTTP Management Interface, navigate to the HTTP Management Interface, then click on the `Disable SSL` button on the right side.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-10.png" >}}

The following dialog asks if the user wants to reload the host domain controller right after disabling it.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-11.png" >}}


## Elytron resources under host controller for domain mode

This applies only for domain mode, the elytron resources are created under `/host=*/subsystem=elytron` path, to manage them you can use the model browser. You can open it by navigating to the HTTP Management Interface screen, then click at `Switch to expert mode` on the top right corner of the screen.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-12.png" >}}

The model browser and the created elytron resources.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-13.png" >}}


## Elytron resources for standalone mode

The elytron resources are created in the regular `/subsystem=elytron` path, under the `Configuration` top level menu, then go to `Subsystem` column.

{{< screenshot src="/img/blog/ssl-wizard/ssl-wizard-14.png" >}}
