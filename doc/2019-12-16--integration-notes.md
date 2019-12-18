# Notes on ReShare Integrations

<!-- md2toc -l 2 2019-12-16--integration-notes.md -->
* [Different services](#different-services)
* [Specific integrations](#specific-integrations)
    * [Item identifiers](#item-identifiers)
    * [Bibliographic records](#bibliographic-records)
    * [Patron identification](#patron-identification)
* [Action items](#action-items)
    * [Server configuration](#server-configuration)
    * [UI changes](#ui-changes)



## Different services

Aside from the Millersville and Temple ReShare nodes that Index Data is running, we have three public nodes running for development and demonstration (details [here](https://openlibraryenvironment.atlassian.net/wiki/spaces/PR/pages/607092760/ReShare+Development+Environments)):
* https://reshare.apps.k-int.com -- run by K-Int.
* http://reshare.reshare-dev.indexdata.com -- run by Index Data
* https://demo.reshare-dev.indexdata -- run by Index Datacom 

The first of these is continuously deployed from `master`, but we are not sure at the moment what the situation is with the other two. Ian Hardy has been asked to update the server documentation with details.

We think that all of these will be running reasonably recent versions of the software, though, so that differences between them more likely represent difference of configuration than outdated software.



## Specific integrations


### Item identifiers

Item Identifiers (field `selectedItemBarcode`) are _not_ pulled in from the supplier ILS, since we don't want librarians to have to obtain a specific copy of a sought instance. Instead the barcode is supposedly pulled in by a check-in action in UI. (However, when I have tried this checkin process myself, the barcode does not seem to get added to the patron-request object.)

Note that we do not obtain item-IDs via Z39.50, only locations and call-numbers. Therefore, Z39.50-server configutation of a ReShare node is irrelevant for the purpose of obtaining and displaying item IDs.


### Bibliographic records

Bibliographic information about sought instances is obtained by looking up the `systemInstanceIdentifier` value in the Shared Index web-service, and the resulting bib record is stored as a single JSON blob in the `bibRecord` field.

This is only done when the ReShare service has had its Shared Index configuration information set correctly -- which is likely _not_ the case for the Index Data-hosted services.


### Patron identification

Requests posted to a ReShare node via the OpenURL service include a `patronReference` field. This is looked up in an NCIP server to obtain information about the patron, which is added to the patron-request record.

Caveats:

* NCIP lookup is only done when the ReShare service has had its NCIP configuration information set correctly -- which is likely _not_ the case for the Index Data-hosted services.

* NCIP-derived patron information is used in pull-slips, but not at present displayed in the "full" record display, which instead tries to interpret the `patronReference` as the UUID of a user in its own local user-register. This user-register does not presently exist, and may never exist, so we should instead use the NCIP-derived data.

* OpenURL generation must incude patron references (i.e. barcodes) that are valid in the configured NCIP server. In practical terms, this means that when we use the MillersVille VuFind, we need the ability to log in as users that are meaningful to the VuFind NCIP service. This can be done either by querying NCIP at login time, or by hardwiring a small set of known MillersVille patrons.



## Action items


### Server configuration

* Since https://demo.reshare-dev.indexdata.com/ is the main system used in demonstrating ReShare, and the one that receives OpenURL requests from VuFind, we need to configure it to know where the shared index web-service it, and what credentials to use, so it can obtain bib records. **[DONE]**

* We also need to set up NCIP configuration for that demo service, most likely to Millersville's NCIP service.

* We need the Millersville VuFind to generate OpenURLs whose `patronReference`s are the barcodes of valid users that the NCIP service knows about.


### UI changes

* The "Requesting user" card should use the NCIP-derived fields instead of trying to look up the `patronReference` in the non-existent local user-register.

* At present, the patron-request record has no fields for some parts of what the user card tries to display. We may need to exend it to include these, and extend the NCIP lookup code to obtain or identify more fields. These include:
  * patron-group -- we are not sure if this concept even exists in NCIP
  * username -- there is no patron-request field for this, but can NCIP supply it?
  * email address -- there _is_ a patron-request field for this, but can NCIP supply it? If so, then we will need to rethink the rule that OpenURLs cannot be submitted unless the patron email address is explicitly specified.

* The "Requesting user" card includes an "ID" field. We should probably just display the patron reference there.



