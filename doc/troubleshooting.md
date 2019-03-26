# Troubleshooting a FOLIO platform

## Introduction

Sometimes you get unexpected responses from Okapi. For example, you might be trying to log into Stripes, and find that the POST to
is rejected with status 422 Unprocessable entity and content like:

> Error verifying user existence: Error looking up user at url 'http://10.0.2.15:9130/users?query=username==diku_admin' Expected status code 200, got '404' :No running module instance found for mod-users-15.3.1-SNAPSHOT.67

Here are some things to try in such circumstances.

## Get information from Okapi:

Is the module registered? Is it deployed and enabled for the tenant? Is it healthy? What are its details?

* http://localhost:9130/_/proxy/modules
* http://localhost:9130/_/proxy/tenants/diku/modules
* http://localhost:9130/_/discovery/health
* http://localhost:9130/_/discovery/modules/mod-users-15.3.1-SNAPSHOT.67

## Get information about the Docker containers

Use `vagrant ssh` to get into the VM, then make inquiries. If things look bad, restart the relevant container:

	guest$ sudo docker ps -a | grep mod-users
	guest$ sudo docker logs jovial_dubinsky
	guest$ sudo docker restart jovial_dubinsky

You may also need to restart Okapi, and to monitor it to determine when its lengthy startup process is complete:

	guest$ sudo systemctl restart okapi
	guest$ tail -f /var/log/folio/okapi/okapi.log

## If all else fails ...

Just destroy the VM and remake it:

	host$ vagrant destroy
	host$ vagrant up

