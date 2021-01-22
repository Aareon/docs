---
Order: 150
xref: qde
Title: Quick Deployment Environment (QDE)
Description: High level information about QDE
RedirectFrom: docs/quick-deployment-environment
---

## Summary

This is an overview on the Chocolatey Quick Deployment Environment (QDE).
It provides a single virtual machine appliance to be imported into your hypervisor-of-choice, which contains most of the various components of a Chocolatey organizational solution.

> :warning: **WARNING**
>
> This solution targets environments up to about **1000 nodes**.
> If you need a solution for a larger environment, QDE is generally suitable only as a proof-of-concept.
> For best results, we would recommend a distributed infrastructure, separating each component into its own discrete node.
> If you find yourself in need of a more scalable solution, please contact Support and we'll be more than happy to provide guidance for larger solutions.

![QDE Architecture](/assets/images/quickdeploy/QDE-architecture.gif)

## QDE Components

The QDE appliance provides a unified architecture containing the following components:

| Component                           | Description                                                                                                                                                                                                               |
| :---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Sonatype Nexus Repository OSS**   | This is your package repository, where you'll be storing your Chocolatey packages (nupkg files). It comes pre-configured with some initial repositories to help you keep everything organized.                            |
| **Jenkins**                         | This is your automation engine, that lets you run tasks on-demand and on a schedule. It hosts the PowerShell scripts to help you auto-internalize packages from the Chocolatey Community Repository.                      |
| **Chocolatey Central Management**   | This is your web dashboard for Chocolatey, that will allow you to track and monitor Chocolatey packages on your endpoint clients. You can see what packages are installed where, and whether or not they are out-of-date. |
| **Scripts for Internal Deployment** | Various scripts to help you configure this solution are included, for your convenience.                                                                                                                                   |

## Getting QDE

To get QDE into your environment, please [reach out to us](https://chocolatey.org/contact/quick-deployment) so we can work to get you set up.

> :memo: **NOTE**
>
> A QDE environment is a fully functional Chocolatey for Business (C4B) environment; as such, it will require a business or trial license.

## Related Articles

A lot of what is done in QDE condenses or completely eliminates the work found in these related articles:

* [Organizational Deployment Guide](xref:organizational-deployment-guide)
* [Chocolatey Installation](https://chocolatey.org/install#organization)
* [Chocolatey Commercial Installation](xref:setup-licensed)
* [Automate Package Internalization](xref:automate-package-internalization)

If you find that QDE is only good for a proof-of-concept in your environment due to having thousands of endpoints, you will want to understand how to scale out that infrastructure.
The above articles document how that is done.

## Links

* [QDE Setup](xref:v2-qde)
* [QDE Desktop ReadMe File](xref:v2-desktop-readme) (included here for convenience)
* [QDE SSL/TLS Setup](xref:v2-ssl-setup)
* [QDE Firewall Changes](xref:v2-firewall-changes)
* [QDE Client Setup](xref:v2-client-setup) (setting up your client machines)
* [How do I upgrade QDE?](#how-do-i-upgrade-qde).

## FAQ

### How do we take advantage of QDE?

You must have a [Business edition of Chocolatey](https://chocolatey.org/compare) or be on an active trial license. Fill out the [quick deployment request form](https://chocolatey.org/contact/quick-deployment) and we will reach out with all the necessary information to get you started.

### How much time does this save?

Typically, setting up a proper Chocolatey Central Management server and any accompanying infrastructure takes somewhere between 1-5 days, even when you have everything you need to get started. Setting up a new piece of infrastructure can be pretty cumbersome, we know. The QDE contains pretty much everything you need to get started in a single image, all ready to go. At most, it should only take a couple of hours to get everything ready to go from there.

### What if we're already a C4B customer?

Fill out the [quick deployment request form](https://chocolatey.org/contact/quick-deployment) and we will reach out with all the necessary information to get you started. Be sure to mention you are already a C4B customer.

### How do we upgrade QDE?

While we will continue to make improvements to the QDE, there is no upgrade path for the Virtual Machine itself.
You can choose to start over with a newer version, but that feels like the wrong way to go.

It is simple to upgrade the components and that it how we recommend upgrading aspects of QDE.
Should you want to upgrade say Central Management, you can follow the Central Management steps for upgrade at [Upgrade Central Management](xref:ccm-upgrade).

### What if we have a larger environment? (> 1k nodes)

While this solution will be great to get a glimpse of what Chocolatey can really do for you, by its nature it isn't the silver bullet for every environment.
If you have a larger environment, we would strongly recommend taking the time to set up the various services on separate hosts to make them easier to manage and ensure they have the necessary resources to handle heavier loads.

### Can we brag about how fast we were able to get configured?

Please do! :slightly_smiling_face: