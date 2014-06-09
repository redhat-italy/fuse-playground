Fabric
======

This is a simple guide to setup a fuse fabric

Setup Fuse Fabric Installation
==============================

To create the simple fabric follow these steps:

### 1) Change name to container
*(Optional)* Customise the name of the root container by editing:

    <INSTALL_DIR>/etc/system.properties 

and specifying a different name for this property:

    karaf.name=root

NOTE: For the first container this step is optional. But at some later stage, if you want to join a root container to the fabric, you must customise the new container's name to prevent it from clashing with any existing root containers in the fabric.

### 2)  Add users

If you have not already done so, create one (or more users) by adding a line of the following form 

	Username=Password[,RoleA][,RoleB]...

to file:

	<INSTALL_DIR>/etc/users.properties
	
At least one of the users must have the admin role, to enable administration of the fabric. For example:

	admin=secretpassword,admin
	
NOTE: The initialization of user data from users.properties happens only once, at the time the fabric is created. After the fabric has been created, any changes you make to users.properties will have no effect on the fabric's user data.

### 3) Create the first container

Create the first fabric container, which acts as the seed for the new fabric, enter this console command:

	JBossFuse:admin@root> fabric:create
	
The current container, named root by default, becomes a Fabric Server with a registry service installed. Initially, this is the only container in the fabric. 
The Zookeeper password is used to protect sensitive data in the Fabric registry service (all of the nodes under /fabric).

now a root container exists we can check with command: 

	JBossFuse:admin@root> fabric:container-list 
	
	[id]	[version] [connected] 		[profiles]                                   [provision status]
	root*   	1.0       true        fabric, fabric-ensemble-0000-1, jboss-fuse-full    success

#### Remove jboss-fuse-full profile from root

We don't need the jboss-fuse-full profile on root container, so we remove it with command, **without** *fabric:&lt;command&gt;*

	JBossFuse:admin@root> container-remove-profile root jboss-fuse-full

So now the root container shows:

	[id]	[version] [connected] 		[profiles]                                   [provision status]
	root*   	1.0       true        fabric, fabric-ensemble-0000-1				    success

### 4) Create child containers

Create two child containers. Assuming that your root container is named root, enter this console command:

	JBossFuse:admin@root> fabric:container-create-child root child 2
	The following containers have been created successfully:
		Container: child1.
		Container: child2.

[TODO: check this info caming from docs]You are prompted to enter a JMX username and password (the JMX port is integrated with the JAAS authentication service). 
Enter one of the username/password combinations that you defined in step 2.

### 5) List containers

Invoke the fabric:container-list command to see a list of all containers in your new fabric. You should see a listing something like this:

	JBossFuse:admin@root> fabric:container-list
	[id]                           [version] [alive] [profiles]                     [provision status]
	root                           1.0       true    fabric, fabric-ensemble-0000-1 success
	  child1                       1.0       true    default                        success
	  child2                       1.0       true    default                        success

## Shutting down the containers

Because the child containers run in their own JVMs, they do not automatically stop when you shut down the root container. To shut down a container and its children, first stop its children using the fabric:container-stop command. For example, to shut down the current fabric completely, enter these console commands:

	JBossFuse:admin@root> fabric:container-stop child1
	JBossFuse:admin@root> fabric:container-stop child2
	JBossFuse:admin@root> shutdown -f

After you restart the root container, you must explicitly restart the children using the fabric:container-start console command.