Fabric Profiles
===============

A profile is the basic unit of deployment in a fabric. You can deploy one or more profiles to a container, and the content of those deployed profiles determines what is installed in the container.

### Contents of a profile

A profile encapsulates the following kinds of information:

The URL locations of features repositories

* A list of features to install
* A list of bundles to install (or, more generally, any suitable JAR package—including OSGi bundles, Fuse Application Bundles, and WAR files)
* A collection of configuration settings for the OSGi Config Admin service
* Java system properties that affect the Apache Karaf container (analogous to editing etc/config.properties)
* Java system properties that affect installed bundles (analogous to editing etc/system.properties)

### Base profile

Profiles support inheritance. 

This can be useful in cases where you want to deploy a cluster of similar servers.

for example, where the servers differ only in the choice of IP port number. For this, you would typically define a base profile, which includes all of the deployment data that the servers have in common. 

Each individual server profile would inherit from the common base profile, but add configuration settings specific to its server instance.

### Create a base profile

To create the *helloworld-cxf-base* profile, follow these steps:

1) Create the *helloworld-cxf-base* profile by entering this console command:

	JBossFuse:admin@root> fabric:profile-create --parents feature-cxf helloworld-cxf-base
	
NOTE: --parents <parent-feature> will add to the new profile *helloworld-cxf-base* the features you need.
	
2) Add the get-started features repository (see Define a Feature for the Application) to the *helloworld-cxf-base* profile by entering this console command:

	//install mvn:org.fusesource.example/cxf-hello-world/1.0-SNAPSHOT
	
	JBossFuse:admin@root> profile-edit -r mvn:org.fusesource.example/get-started/1.0-SNAPSHOT/xml/features helloworld-cxf-base

3) Now add the get-started-cxf feature (which provides the Web service example server) to the *helloworld-cxf-base* profile. Enter the following console command:

### Create the derived profiles

You create two derived profiles, gs-cxf-01 and gs-cxf-02, which configure different IP ports for the Web service. To do so, follow these steps:

1) Create the *gs-cxf-01* profile which derives from *helloworld-cxf-base-by* entering this console command:

	JBossFuse:admin@root> profile-create --parents *helloworld-cxf-base* gs-cxf-01
	
2) Create the *gs-cxf-02* profile which derives from *helloworld-cxf-base* by entering this console command:

	JBossFuse:admin@root> profile-create --parents *helloworld-cxf-base* gs-cxf-02
	
3) In the *gs-cxf-01* profile, set the portNumber configuration property to *8185*, by entering this console command:

	JBossFuse:admin@root> profile-edit -p org.fusesource.example.get.started/portNumber=8185 gs-cxf-01
	
4) In the *gs-cxf-02* profile, set the portNumber configuration property to *8186*, by entering this console command:

	JBossFuse:admin@root> profile-edit -p org.fusesource.example.get.started/portNumber=8186 gs-cxf-02
			
## Deploy the Profiles

Deploy profiles to the child containers

Having created the child containers, as described in [Create a Fabric](fabric.md), and the profiles, as described in Create Fabric Profiles, you can now deploy the profiles. To do so, follow these steps:

1) Deploy the gs-cxf-01 profile into the child1 container by entering this console command:

	JBossFuse:admin@root> fabric:container-change-profile child1 gs-cxf-01
	
2) Deploy the gs-cxf-02 profile into the child2 container by entering this console command:

	JBossFuse:admin@root> fabric:container-change-profile child2 gs-cxf-02
	
### Test the deployed profiles

You can test the deployed profiles using the WS client from the cxf-basic Maven project described in Create a Web Services Project. To do so, follow these steps:

1) Open a new command prompt and cd to 
	
	get-started/cxf-basic.

2) Test the gs-cxf-01 profile (which deploys a Web service listening on port 8185) by entering this command:

	mvn -Pclient -Dexec.args="http://localhost:8185/PersonServiceCF
	
3) Test the gs-cxf-02 profile (which deploys a Web service listening on port 8186) by entering this command:

	mvn -Pclient -Dexec.args="http://localhost:8186/PersonServiceCF
		
## Update a Profile

### Atomic container upgrades

Normally, when you edit a profile that is already deployed in a container, the modification takes effect immediately. This is so because the Fabric Agent in the affected container (or containers) actively monitors the fabric registry in real time.

In practice, however, immediate propagation of profile modifications is often undesirable. In a production system, you typically want to roll out changes incrementally: for example, initially trying out the change on just one container to check for problems, before you make changes globally to all containers. Moreover, sometimes several edits must be made together to reconfigure an application in a consistent way.

### Profile versioning

For quality assurance and consistency, it is typically best to modify profiles atomically, where several modifications are applied simultaneously. To support atomic updates, fabric implements profile versioning. Initially, the container points at version 1.0 of a profile. When you create a new profile version (for example, version 1.1), the changes are invisible to the container until you upgrade it. After you are finished editing the new profile, you can apply all of the modifications simultaneously by upgrading the container to use the new version 1.1 of the profile.


### Upgrade to a new profile

For example, to modify the gs-cxf-01 profile, when it is deployed and running in a container, follow the recommended procedure:

Create a new version, 1.1, to hold the pending changes by entering this console command:

	JBossFuse:admin@root> fabric:version-create
	
Created version: 1.1 as copy of: 1.0     
The new version is initialised with a copy of all of the profiles from version 1.0.

Use the fabric:profile-edit command to change the portNumber of gs-cxf-01 to the value 8187 by entering this console command:

	JBossFuse:admin@root> fabric:profile-edit -p org.fusesource.example.get.started/portNumber=8187 gs-cxf-01 1.1

Remember to specify version 1.1 to the fabric:profile-edit command, so that the modifications are applied to version 1.1 of the gs-cxf-01 profile.

Upgrade the child1 container to version 1.1 by entering this console command:

	JBossFuse:admin@root> fabric:container-upgrade 1.1 child1
	
### Roll back to an old profile

You can easily roll back to the old version of the gs-cxf-01 profile, using the fabric:container-rollback command like this:

	JBossFuse:admin@root> fabric:container-rollback 1.0 child1