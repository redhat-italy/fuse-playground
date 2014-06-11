Define Feature
==============

An OSGi bundle is not a convenient unit of deployment to use with the Red Hat JBoss Fuse container.

Apache Karaf features are designed to address this problem. A feature is essentially a way of aggregating multiple OSGi bundles into a single unit of deployment. When defined as a feature, you can simultaneously deploy or undeploy a whole collection of bundles.

#### What to put in a feature

At a minimum, a feature should contain the basic collection of OSGi bundles that make up the core of your application.

#### Deployment options

You have a few different options for deploying features, as follows:

1) *Hot deploy* the simplest deployment option; just drop the XML features file straight into the hot deploy directory, InstallDir/deploy.

2) *Add a repository URL* you can tell the Red Hat JBoss Fuse container where to find your features repository file using the features:addUrl console command (see Add the local repository URL to the features service in Deploying into the Container). You can then install the feature at any time using the features:install console command.

3) *Through a Fuse Fabric profile* you can use the management console to deploy a feature inside a Fuse Fabric profile.

For more details about the feature deployment options, see Deploying a Feature in Deploying into the Container.

#### Features and Fuse Fabric

It turns out that a feature is a particularly convenient unit of deployment to use with Fuse Fabric. A Fuse Fabric profile typically consists of a list of features and a collection of related configuration settings. Hence, a Fuse Fabric profile makes it possible to deploy a completely configured application to any container in a single atomic operation.


### Create a custom features repository

Under the *get-started-examples* there is this file:

	get-started-examples/features/src/main/resources/get-started-examples.xml
	
with this content:
	
	<?xml version="1.0" encoding="UTF-8"?>
	<features name="get-started">
	  <feature name="get-started-examples">
	    <bundle>mvn:org.fusesource.example/cxf-hello-world/1.0-SNAPSHOT</bundle>
	    <bundle>mvn:org.fusesource.example/camel-hello-world/1.0-SNAPSHOT</bundle>
	  </feature>
	  <feature name="get-started-cxf">
	    <bundle>mvn:org.fusesource.example/cxf-hello-world/1.0-SNAPSHOT</bundle>
	  </feature>
	</features>
	
Under 

	get-started-examples/features
	
there is the pom.xml to build the feature

#### Install the features repository

You need to install the features repository into your local Maven repository, so that it can be located by the Red Hat JBoss Fuse container.

	mvn install

#### Deploy the custom feature (**not Fabric managed**)

To deploy the get-started-basic feature into the container, perform the following steps:

If the *cxf-hello-world* and *camel-hello-world* bundles are already installed in the JBoss Fuse container you **must first uninstall** them:
At the console prompt, use the list command to discover the bundle IDs for the *cxf-hello-world* and *camel-hello-world* bundles, and then uninstall them both using the console command, 

	uninstall <BundleID>

Before you can access features from a features repository, you must tell the container where to find the features repository. Add the features repository URL to the container, by entering the following console command:

	JBossFuse:karaf@root> features:addurl mvn:org.fusesource.example/get-started-examples/1.0-SNAPSHOT/xml/features

check with command:

    features:listurl

the list:

	Loaded   URI
      true    mvn:org.apache.karaf.assemblies.features/spring/2.3.0.redhat-610379/xml/features
      [...]
      true    mvn:org.fusesource.example/get-started-examples/1.0-SNAPSHOT/xml/features
      [...]
      true    mvn:org.apache.camel.karaf/apache-camel/2.12.0.redhat-610379/xml/features
      true    mvn:org.fusesource.patch/patch-features/1.0.0.redhat-379/xml/features
      true    mvn:org.apache.karaf.assemblies.features/standard/2.3.0.redhat-610379/xml/features
      true    mvn:io.fabric8/fabric8-karaf/1.0.0.redhat-379/xml/features
      true    mvn:org.ops4j.pax.web/pax-web-features/3.0.6/xml/features
      true    mvn:org.apache.servicemix.nmr/apache-servicemix-nmr/1.6.0.redhat-610379/xml/features

	
If necessary, you can use the command:

    features:refreshurl

To force the container to re-read its features repositories.

To install the get-started-examples and get-started-cxf feature, enter the following console command:

	JBossFuse:karaf@root> features:install get-started-examples
	JBossFuse:karaf@root> features:install get-started-cxf

After waiting a few seconds for the bundles to start up, you can test the application as described in Test the route with the WS client.

To uninstall the feature, enter the following console command:

	JBossFuse:karaf@root> features:uninstall get-started-examples
	JBossFuse:karaf@root> features:uninstall get-started-cxf