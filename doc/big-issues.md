# Big issues

A user of a university's existing discovery system clicks "get this" to generate an ILL request. But how can we obtain the patron information, to make it part of this request?

And how can we know what tenant the user's ReShare node runs on. We can look up the location in mod-directory, but how do we know that? This bit has to happen before we get into Okapi and mod-rs, because we need to specify the tenant in the Okapi request.

Solving these may involve talking to the university's SSO system. And different participating universities will no doubt use different SSO solutions.

This bit of the work will need to be done by a Listener -- one which knows how to deal with the relevant SSO.

For example, we will need a Listener that serves HTTP and accepts an OpenURL link, and examines the accompanying headers to obtain information that can be used to interrogate the SSO system about the patron and the location. We currently have no-one assigned to write these.

(XXX note to self: learn more about SSO systems: identity providers, service providers, SAML.)

We can shift this work sideways by saying we're providing a specific interface which discovery layers must conform to.

We can mock this up using a form.

Ian thinks that every discovery layer will include, along with its OpenURLs, headers that tell us what we need to know about the patron.

As a matter of urgency, we need to see some real OpenURL requests from some real discovery systems: not just the OpenURL itself, but the entire HTTP request, including the headers.

* How can we send the patron notifications? Will we know enough about them? Will we have access to the necessary APIs in the local discovery system or other related system?

Constructing the rota potentially involves many processes:
* Searching in the central index
* Broadcast searching of providers not included in the central index
* Availability checking
* Sorting

ReShare code will need to talk directly to the ILS: for example, when it checks out a loaned item to a virtual patron representing the requesting library. Bur we will _not_ need a layer to abstract operations to be executed on the local ILS, as they will all implement SIP.

Can we automate the addition of shipping IDs to ReShare requests? Maybe we can extract the relevant information from their automatically generated emails, or maybe there is a standard API that they implement.

