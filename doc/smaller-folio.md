# Towards a smaller FOLIO: using optional dependencies

<!-- md2toc -l 2 smaller-folio.md -->
* [Motivation](#motivation)
* [Using optional dependencies](#using-optional-dependencies)
* [Specifying what to include in a FOLIO server platform](#specifying-what-to-include-in-a-folio-server-platform)


## Motivation

[Project ReShare](https://projectreshare.org/) is a resource-sharing (inter-library loan) application built on [the FOLIO library services platform](https://www.folio.org/). It consists of a small number of apps (Request, Supply, Directory, Scan, etc.) running as FOLIO modules with front-end components (e.g. the Stripes module `ui-rs`) and back-end components (e.g. `mod-rs`). Also included is one of the standard FOLIO apps, Users.

It turns out that a "minimal" FOLIO platform to support these apps is pretty big, and includes a lot of back-end modules that ReShare has no use for. For example, [the development snapshot server](http://reshare.reshare-dev.indexdata.com/), though it supports only five apps (Users, Directory, Request, Supply and Update), has pulls in [27 modules in total](http://reshare.reshare-dev.indexdata.com/settings/about) at the time of writing:

* Developer settings (`folio_developer-1.10.1`)
* Interact with the library services directory (`folio_directory-1.1.0`)
* Top-level platform for ReShare UI (`folio_platform-rs-1.0.0`)
* Request resources for ILL (`folio_request-1.0.0`)
* Stripes framework (`folio_stripes-3.1.2`)
* Supply resources for ILL (`folio_supply-1.0.0`)
* Quickly change the status of items. Part of the ReShare project. (`folio_update-1.0.0`)
* User management (`folio_users-2.26.3`)
* authtoken (`mod-authtoken-2.3.0`)
* Circulation Storage Module (`mod-circulation-storage-10.0.0-SNAPSHOT.209`) **UNUSED?**
* Configuration (`mod-configuration-5.1.0`)
* mod-directory (`mod-directory-1.2.2`)
* email (`mod-email-1.5.0`)
* mod-event-config (`mod-event-config-1.3.2`) **UNUSED?**
* Inventory Storage Module (`mod-inventory-storage-17.0.0`) **UNUSED?**
* login (`mod-login-6.1.1`)
* Notify (`mod-notify-2.3.1`)
* Password Validator Module (`mod-password-validator-1.4.1`)
* permissions (`mod-permissions-5.8.3`)
* mod-rs (`mod-rs-1.9.0`)
* Mod sender (`mod-sender-1.1.0`) **UNUSED?**
* Source Record Storage Module (`mod-source-record-storage-2.6.1`) **UNUSED?**
* Tags (`mod-tags-0.5.0`)
* Template engine module (`mod-template-engine-1.6.1`) **UNUSED?**
* users (`mod-users-15.6.2`)
* users business logic (`mod-users-bl-5.1.0`)
* Okapi (`okapi-2.32.0`)

It seems that six of these modules are probably not used by ReShare (though this needs checking).

That's fewer than we expected, isn't it?


## Using optional dependencies

XXX


## Specifying what to include in a FOLIO server platform

XXX

