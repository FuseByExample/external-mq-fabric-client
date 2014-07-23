MQ-Fabric Client Example
========================

This project shows how to connect to JBoss A-MQ message brokers running in Fuse
Fabric from JMS clients running outside of Fuse Fabric (i.e. when the JMS client
is not running within a Fabric-enabled Fuse ESB container).

The Getting Started with ActiveMQ project uses broker URL's like this:

    failover:(tcp://host1:port1,tcp://host2:port2)

which are replaced in this project with values like this (that also handle
failover):

    discovery:(fabric:broker-group-name)

In addition to changing the broker URL's, two other changes are needed to
support fabric discovery: a `zookeeper.url` System property needs to be set in
the environment, and a few Fabric libraries must be added to the classpath.

Setting the zookeeper.url
-------------------------

On a typical developer machine, with Fuse Management Console running locally,
the `zookeeper.url` System property should be set to the URL of the Fuse Fabric's
Zookeeper instance, which defaults to `localhost:2181`. One can simply append the
property to the startup command line like this:

	-Dzookeeper.url=localhost:2181 
	
or add the following to the Maven profile configuration, as is done in this
project.

    <systemProperty>
        <key>zookeeper.url</key>
        <value>localhost:2181</value>
    </systemProperty>

When a distributed Fabric Registry is used (i.e. a Zookeeper ensemble) the
`zookeeper.url` property should be set to a comma delimited list, like this:

    -Dzookeeper.url=london:2182,seattle:2181,portland:2181

Adding the Fabric libraries
---------------------------

These are the dependencies added to the Maven project to support the `fabric`
discovery protocol (see the `pom.xml` for version info):

    <dependency>
        <groupId>org.jboss.amq</groupId>
        <artifactId>mq-fabric</artifactId>
        <version>${amq.version}</version>
    </dependency>
    <dependency>
        <groupId>io.fabric8</groupId>
        <artifactId>fabric-groups</artifactId>
        <version>${fabric.version}</version>
    </dependency>
    <dependency>
        <groupId>io.fabric8</groupId>
        <artifactId>fabric-zookeeper</artifactId>
        <version>${fabric.version}</version>
    </dependency>
    <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.core</artifactId>
        <version>${osgi.version}</version>
    </dependency>
    <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.compendium</artifactId>
        <version>${osgi.version}</version>
    </dependency>

Setup the examples against a fabric-based network of brokers
-------------------------------------------------------------

Start a fabric-based network of fault-tolerant (master/slave) brokers.
For instructions on how to configure and deploy such a network, see the
[fabric-ha-setup-master-slave.md](./docs/fabric-ha-setup-master-slave.md).

This configuration features two broker groups networked together,
named "amq-east" and "amq-west", each of which is comprised of a
master/slave pair (four brokers total). Consumers will connect to the
active broker in the "amq-west" group; producers to the active broker
in the "amq-east" group, insuring that messages flow across the
network. After the example is up and running, one can kill either or
both of the active brokers and observe continued message flow across
the network.

Running the examples
--------------------

Follow the instructions in the various consumer and producer paired example
modules:

* simple-consumer and simple-producer
* camel-consumer and camel-producer
