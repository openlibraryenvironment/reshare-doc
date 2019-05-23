# Directory-entry propagation in ReShare

<style>
  h2 {
    border-top: 1px solid lightgrey;
    margin-top: 1.5em;
    padding-top: 0.5em;
  }
</style>

<!-- md2toc -l 2 directory-entry-propagation.md -->
* [Introduction](#introduction)
* [Principles](#principles)
* [Data](#data)
* [Long-term solution](#long-term-solution)
    * [Propagation](#propagation)
    * [Security](#security)
* [MVP solution](#mvp-solution)
    * [MVP Propagation](#mvp-propagation)
    * [MVP Security](#mvp-security)


## Introduction

The ReShare project aims to allow libraries to perform inter-library loans with members of other libraries that are in the same consortium as them; and potentially in other consortia. To support this, it is necessary to furnish a directory of the available candidate lending sources, containing information about what protocols they support, how much they charge, etc. The beginnings of support for this directory are provided by the server-side module [`mod-directory`](https://github.com/openlibraryenvironment/mod-directory) and its client-side counterpart [`ui-directory`](https://github.com/openlibraryenvironment/ui-directory).

The directory must be available to all nodes in a ReShare network, in a reasonably up-to-date form. There are several possible ways to achieve this goal, varying in elegance, efficiency and development effort. In this document, we will survey some possible solutions, and try to determine a realistic path to an ideal arrangement.


## Principles

* Each library (and branch, here and elsewhere) may belong to multiple consortia.
* We want each library to be responsible for maintaining the information about itself and its branches.
* We want changes made in the information about a library or its branches to propagate to all the members of that all library's consortia.
* We would like to avoid a single point of failure in the propagation mechanism.
* Some information about library A may vary when viewed from the perspectives of libraries B and C: for example, A may charge more for a loan to B than it does to C.
* We recognise that we are not likely to achieve the full generality of this solution in time for the MVP (Minimal Viable Product, "Inevitable Narwhal").
* We aim to build an MVP direction propagation system that meets immediate needs and is on the path to the full solution.


## Data

To do this, we need to draw a distinction between two kinds of record:

* Directory-entry records are the information that makes up the actual directory, about entities of interest to the wider ReShare organization: libraries, branches, etc.
* Node records describe the nodes of the directory-propagation network, and are of interest only to the Directory system itself.


## Long-term solution

### Propagation

We want a flexible and general approach to change propagation, in which new and modified directory-entry records can be sent from their originating node to any other nodes known to the originator, and passed on from there to more distant nodes. Such a mechanism _may_ benefit from a concept of "backbone nodes", or it may be simpler and more efficient just to treat the network as homogeneous.

For this to work well, there will need to be a loop-avoidance mechanism: for example, node records could contain a "seen by nodes" list, and that list in the version of a node record that is passed on could be augmented with the identifier of the propagating node, so other nodes know not to send it back to any of those nodes on the list.

### Security

We will need to ensure that only the node that originated a directory-entry record can make changes to it. (In general, a node will provide and maintain multiple directory-entry records, as it may be responsible for multiple branches of a library.)

The best way to do this probably using public-key cryptography: the following scheme should give us what we need.

1. Each node will publish a unique identifier and its public key in its own node record.

2. Any node can create a directory-entry record. When it does so, it includes a unique directory-entry identifier, and reference to its own unique ID. The node is the _owner_ of that record.

3. Any node can update any directory-entry record that it created. When it does so, the replacement record includes the directory entry's ID and a cryptographic hash of the replacement record signed with the owning node's private key.

4. When any node receives an updated directory-entry record from any other node, it will decrypt the replacement record using the public key of the owning node.

This should ensure that records can only be updated by the node that created them.

**XXX Have I missed any loopholes? Security is notoriously hard to get right, and I will feel happier when there have been more eyeballs on this.**


## MVP solution

### MVP Propagation

It may be reasonable just to go ahead and implement the long-term solution to directory-entry propagation up front. It doesn't sound like a huge amount of work.

However, it might be quicker in the short term to initially implement a simpler hierarchical topology where each node that wants to change a directory entry sends that new version to a well-known "gatekeeper" node, and that node forwards it to all the other nodes.

But there a still simpler approach which is on the path to the long-term solution: the node making a change to a directory entry that it owns could publish it to all the nodes directly known to it (as in the long-term solution) but that could be the end of the matter. For the MVP, we would simply arrange that every node knows about every other node, so that no multi-hop propagation is necessary. In this simplified version, there is no need for the "seen by nodes list", as there are no loops to avoid. This minimal approach will probably suffice when we are dealing with only a single consortium.

### MVP Security

For the minimal viable product, it might suffice to omit security precautions entirely, and trust that consortium members will not try to change each other's directory-entry records.

In any case, we will need to think how this ties in with the standard FOLIO-platform authentification mechanisms. Presumably notification of a new or changed directory entry will be by POST or PUT to an Okapi-mediated web service, so it will be necessary to authenticate onto the remote node's FOLIO platform first. This can probably best be done using a special account on each node that is used only for this purpose. We will need to think about how credentials are made available.


## Open issues

* Do we actually need node records as well as directory-entry records? Does it suffice for nodes to have an ID, and for directory-entry records to refer to that ID? If we did that, we would still need some way for a node receiving a directory-entry record to determine the public key of the node that owns it, so it can validate the authenticity of the update.

* Is all this a solved problem? It feels like it might be. Does a secure distributed propagation system already exist in an off-the-shelf form that we can use?


