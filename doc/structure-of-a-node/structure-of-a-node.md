# Structure of a ReShare node

<!-- md2toc -l 2 structure-of-a-node.md -->
* [Assumptions](#assumptions)
* [Components](#components)
    * [Overview](#overview)
    * [The Resource Sharing UI](#the-resource-sharing-ui)
    * [Listeners](#listeners)
    * [mod_directory](#mod_directory)
    * [mod_rs](#mod_rs)
* [Interactions](#interactions)


## Assumptions

We must start from the assumption that ReShare nodes need to run on behalf of libraries using many different ILSs, and many different single-sign-on (SSO) systems. In particular, we are not at liberty to assume that the library is using FOLIO. This has implications for integration: it will potentially be one of the most demanding parts of the whole project, since it potentially needs to encompass an _n_ × _m_ matrix.

We must assume, then, that a library that wants to use ReShare to allow patrons to borrow items already has a discovery layer, and that layer already has a "Get this" button of some form -- most likely generating an OpenURL, but maybe issuing an ISO ILL request, sending an email in a well-known format, or some other procedure.


## Components

### Overview

A single ReShare node will be made of the following components:

![Structure including Okapi as mediator](structure-including-okapi.png)

In this diagram:
* The green box indicates the administrative UI, which can be used to generate and process requests.
* The pink boxes represent Listeners: processes that listen for incoming requests via OpenURL, ISO ILL, email or any other supported process. There may be any number of these.
* The yellow box indicates an installation of the FOLIO LSP: not necessarily running any ILS components, but ReShare modules. It can be treated as a black box for most purposed.
* The blue box indicates Okapi, FOLIO's application gateway which routes WSAPI invocations to the correct modules
* The gold boxes indicate actual modules running within the FOLIO LSP.

Note that Okapi and the rest of the FOLIO LSP are off-the-shelf code from the FOLIO project. Only the green, pink and gold components need to be build specially for ReShare.

In FOLIO, all requests to modules are routed via Okapi, which takes care of routing, authentication and other matters -- as reflected in the diagram above. However, in can be clearer in conceptual terms to ignore Okapi, or to take it for granted, and simply to show client code calling into specific back-end modules, as in the following modified version of the diagram:

![Structure ignoring Okapi](structure-ignoring-okapi.png)

In this version, it can be clearly seen that the Resource Sharing UI and the listeners all communicate with both mod_directory and mod_rs. We shall discuss these interactions [below](#interactions).

### The Resource Sharing UI

XXX

### Listeners

XXX

### mod_directory

XXX

### mod_rs

XXX


## Interactions

Before we can get to mod-rs (or any Okapi-mediated service) we need to know which tenant is hosting the ReShare modules. So the OpenURL listener needs a way of figuring out "This OpenURL is from that source, which means its associated with tenant X".

XXX Summarise discussion with Chas

