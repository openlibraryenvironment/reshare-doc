# Building and running the ReShare resource-sharing module

First, to build and run the back-end `rs` module:

	mod-rs$ brew install jq
	mod-rs$ ./gradlew -Dgrails.env=vagrant-db bootRun

This process runs in the foreground, to start another shell to register the module with Okapi and enable it for the `diku` tenant:

	mod-rs$ cd okapi-scripts
	okapi-scripts$ ./register_and_enable.sh

Now load the sample records:

	okapi-scripts$ ./load_test_data.sh

The back-end module should now be running, available, and populated. Now you need to register the corresponding front-end module with Okapi, enable it for the `diku` tenant, and give the `diku_admin` user the necessary permission to access the UI module:

	okapi-scripts$ cd ../../../ui/ui-rs
	ui-rs$ stripes mod add
	Module descriptor folio_rs-1.0.0 added to Okapi
	ui-rs$ stripes mod enable
	Module folio_rs-1.0.0 associated with tenant diku
	ui-rs$ stripes perm assign --name module.rs.enabled --user diku_admin
	User diku_admin assigned permission module.rs.enabled

Now you can start Stripes:

	ui-rs$ cd ../platform-rs
	platform-rs$ stripes serve stripes.config.js

And go to http://localhost:3000 in your browser. You should see the "Resource Sharing" app in the list at the top of the page.


## Appendix: building and running mod-directory

The process is the same, except:
* You run `gradlew` in the `service` subdirectory instead of at the top level
* You run `register_and_enable.sh` in the `scripts` directory instead of `okapi-scripts`
* The script to load data -- also in `scripts` -- is called `./load_palci.sh` instead of `load_test_data.sh`

