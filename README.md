External MQ-Fabric Client Demo
==============================

This project shows how to connect to Fuse MQ message brokers running in Fuse Fabric from JMS clients running outside of
Fuse Fabric (i.e. when the JMS client is not running within a Fabric-enabled Fuse ESB container).

This project reuses the classes and resources from Fuse By Example's "Getting Started with ActiveMQ" project to
demonstrate the minimal changes necessary to discover and connect to Fuse MQ brokers deployed within Fuse Fabric.

The Getting Started with ActiveMQ project uses broker URL's like this:

    failover:(tcp://host1:port1,tcp://host2:port2)

which are replaced in this project with values like this (that also handle failover):

    discovery:(fabric:broker-group-name)

In addition to changing the broker URL's, two other changes are needed to support fabric discovery: a "zookeeper.url"
System property needs to be set in the environment, and a few fabric libraries must be added to the classpath.

Setting the zookeeper.url
-------------------------

On a typical developer machine, with Fuse Management Console running locally, the zookeeper.url System property should
be set to the URL of the Fuse Fabric's Zookeeper instance, which defaults to localhost:2181. One can simply append the
property to the startup command line like this:

	-Dzookeeper.url=localhost:2181 
	
or add the following to the maven profile configuration, as is done in this project.

    <systemProperty>
        <key>zookeeper.url</key>
        <value>localhost:2181</value>
    </systemProperty>

When a distributed fabric registry is used (i.e. a Zookeeper ensemble) the zookeeper.url property should be set to a
comma delimited list, like this:

    -Dzookeeper.url=london:2182,seattle:2181,portland:2181

Adding the fabric libraries
----------------------------------

These are the dependencies added to the maven project to support the fabric discovery protocol (see the pom.xml for
version info):

    <dependency>
        <groupId>org.fusesource.mq</groupId>
        <artifactId>mq-fabric</artifactId>
        <version>${fabric.version}</version>
    </dependency>
    <dependency>
        <groupId>org.fusesource.fabric</groupId>
        <artifactId>fabric-groups</artifactId>
        <version>${fabric.version}</version>
    </dependency>
    <dependency>
        <groupId>org.fusesource.fabric</groupId>
        <artifactId>fabric-zookeeper</artifactId>
        <version>${fabric.version}</version>
    </dependency>
    <dependency>
        <groupId>org.fusesource.fabric</groupId>
        <artifactId>fabric-linkedin-zookeeper</artifactId>
        <version>${fabric.version}</version>
    </dependency>


Building the example
---------------------------

To build the example, execute the command: 

	> mvn clean install


Running the example against a default fabric broker
-------------------------------------------------------------------

Make sure a default instance of Fuse MQ message broker is running in the local fabric; if not, see below for
instructions on how to deploy a default broker.

To start the default consumer, open a shell, change to the project root and run the command:

	> mvn -e -P consumer-default

After the consumer is running, open another shell, change to the project root and run the command:

	> mvn -e -P producer-default

Here are some console messages you should see from the consumer when running the example:

	******************************
	Connecting to Fuse MQ Broker using URL: discovery:(fabric:default)
	******************************
	Using local ZKClient
	Client environment:zookeeper.version=3.4.3-1240972...
	...
	Initiating client connection, connectString=localhost:2181 sessionTimeout=10000 watcher=org.linkedin.zookeeper.client.ZKClient@12133926
	Opening socket connection to server /0:0:0:0:0:0:0:1:2181
	...
	Socket connection established to localhost/0:0:0:0:0:0:0:1:2181, initiating session
	Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x137bcd28a79000b, negotiated timeout = 10000
	Starting StateChangeDispatcher
	Adding new broker connection URL: tcp://mbrooks1.local:50865
	Successfully connected to tcp://mbrooks1.local:50865
	Start consuming messages from queue://fabric.simple with 120000ms timeout
	Got 1. message: 1. message sent
	Got 2. message: 2. message sent
	Got 3. message: 3. message sent
	...

Running the example against a fabric-based network of brokers
-------------------------------------------------------------

Start a fabric-based network of fault-tolerant (master/slave) brokers. For instructions on how to configure and deploy
such a network, see the blog post [here](http://fusebyexample.blogspot.com/2012/06/using-fuse-management-console-and-fuse.html).
Summary instructions are also included below.

This configuration features two broker groups networked together, named "mq-east" and "mq-west", each of which is
comprised of a master/slave pair (four brokers total). Consumers will connect to the active broker in the "mq-west"
group; producers to the active broker in the "mq-east" group, insuring that messages flow across the network. After the
example is up and running, one can kill either or both of the active brokers and observe continued message flow across
the network.

To start the consumer open a shell, change to the project root and run the command:

	> mvn -e -P consumer-west

After the consumer is running, open another shell, change to the project root and run the command:

	> mvn -e -P producer-east

You should see console messages that show the producer connected using the URL

	discovery:(fabric:mq-east)

and the consumer connected using the URL:

	discovery:(fabric:mq-west)
	
After the example is up and running and you see JMS messages being logged to the consumer's console, kill the broker
running in the container MQ-East1 (you can stop the container using FMC, or kill the container's process in the OS) and
watch the producer failover, reconnect and resume sending messages, with console output like this:

	...
	Sending to destination: queue://fabric.simple this text: '23. message sent
	Sending to destination: queue://fabric.simple this text: '24. message sent
	Sending to destination: queue://fabric.simple this text: '25. message sent
	WARN  Transport (tcp://192.168.0.14:51779) failed, reason:  java.io.EOFException, attempting to automatically reconnect
	Adding new broker connection URL: tcp://mbrooks1.local:51805
	Successfully reconnected to tcp://mbrooks1.local:51805
	Sending to destination: queue://fabric.simple this text: '26. message sent
	Sending to destination: queue://fabric.simple this text: '27. message sent
	...

then kill the container MQ-West1 and observe similar failover and reconnection by the consumer.

Deploying a default Fuse MQ message broker in fabric 
----------------------------------------------------------------------

Here are brief instructions on how to deploy a default instance of Fuse MQ  message broker in fabric:

* Launch Fuse Management Console (FMC), e.g. `bin/fmc`. FMC info and downloads are
[here](http://fusesource.com/products/fuse-management-console)
	* Note: with a clean install of FMC, you will need to edit the `etc/users.properties` to  include an *admin* userid
	with password *admin* and role *admin*. The entry should look like `admin=admin,admin`.

* Connect to FMC in a browser (e.g. http://localhost:8107), create a fabric if one does not yet exist using the
following credentials, and then login (you may need to provide Username/Password you just entered).
	* Username = *admin*
	* Password = *admin*
	* ZooKeeper Password = *admin*

* Click the “Create Fuse Container” button, name the container MQ (the name is not critical), click Next, select the
“mq” profile, click Next, select “Child container”, click Next, select “FuseManagementConsole” as the parent, and click
Finish. The broker instance will be registered in fabric in the default group.

Deploying a fabric-based network of fault-tolerant message brokers
--------------------------------------------------------------------------------------

Here are brief instructions on deploying a fabric-based network of fault-tolerant (master/slave) message brokers. See
the blog post [here](http://fusebyexample.blogspot.com/2012/06/using-fuse-management-console-and-fuse.html) for a more
complete description.

* Launch Fuse Management Console (FMC). 

* Connect to FMC in a browser, create a fabric if one does not yet exist and login using *admin* | *admin* credentials.

* Execute the following four commands one at a time in the FMC terminal:

  Note: the `profile-edit` commands add the username/password for the broker being connected to. That is, for the
   "mq-east-broker" profile, the username and password of the mq-west-broker must be provided. In this example, since
   everything is a child container, they will pickup the user.properties of the parent container, i.e. fmc, but if you
   were networking to a different server with different user settings, you need to make specify those settings.

```
    mq-create --group mq-east --networks mq-west mq-east-broker
    profile-edit -p org.fusesource.mq.fabric.server-mq-east-broker/network.userName=admin mq-east-broker
    profile-edit -p org.fusesource.mq.fabric.server-mq-east-broker/network.password=admin mq-east-broker

    mq-create --group mq-west --networks mq-east mq-west-broker
    profile-edit -p org.fusesource.mq.fabric.server-mq-west-broker/network.userName=admin mq-west-broker
    profile-edit -p org.fusesource.mq.fabric.server-mq-west-broker/network.password=admin mq-west-broker

    container-create-child --profile mq-east-broker FuseManagementConsole MQ-East 2
    container-create-child --profile mq-west-broker FuseManagementConsole MQ-West 2
```
