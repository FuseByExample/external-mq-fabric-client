External MQ-Fabric Client Demo
==============================

This project shows how to connect to JBoss A-MQ message brokers running in Fuse
Fabric from JMS clients running outside of Fuse Fabric (i.e. when the JMS client
is not running within a Fabric-enabled Fuse ESB container).

The Getting Started with ActiveMQ project uses broker URL's like this:

    failover:(tcp://host1:port1,tcp://host2:port2)

which are replaced in this project with values like this (that also handle
failover):

    discovery:(fabric:broker-group-name)

In addition to changing the broker URL's, two other changes are needed to
support fabric discovery: a "zookeeper.url" System property needs to be set in
the environment, and a few fabric libraries must be added to the classpath.

Setting the zookeeper.url
-------------------------

On a typical developer machine, with Fuse Management Console running locally,
the zookeeper.url System property should be set to the URL of the Fuse Fabric's
Zookeeper instance, which defaults to localhost:2181. One can simply append the
property to the startup command line like this:

	-Dzookeeper.url=localhost:2181 
	
or add the following to the maven profile configuration, as is done in this
project.

    <systemProperty>
        <key>zookeeper.url</key>
        <value>localhost:2181</value>
    </systemProperty>

When a distributed fabric registry is used (i.e. a Zookeeper ensemble) the
zookeeper.url property should be set to a comma delimited list, like this:

    -Dzookeeper.url=london:2182,seattle:2181,portland:2181

Adding the fabric libraries
----------------------------------

These are the dependencies added to the maven project to support the fabric
discovery protocol (see the pom.xml for version info):

    <dependency>
        <groupId>org.jboss.amq</groupId>
        <artifactId>mq-fabric</artifactId>
        <version>${amq.version}</version>
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


Running the example against a fabric-based network of brokers
-------------------------------------------------------------

Start a fabric-based network of fault-tolerant (master/slave) brokers.
For instructions on how to configure and deploy such a network, see the
[fabric-ha-setup.md](https://github.com/FuseByExample/external-mq-fabric-client/blob/master/fabric-ha-setup.md).

This configuration features two broker groups networked together,
named "mq-east" and "mq-west", each of which is comprised of a
master/slave pair (four brokers total). Consumers will connect to the
active broker in the "mq-west" group; producers to the active broker
in the "mq-east" group, insuring that messages flow across the
network. After the example is up and running, one can kill either or
both of the active brokers and observe continued message flow across
the network.

To start the consumer open a shell, change to the project root and run the command:

	> mvn -e -P consumer-west

After the consumer is running, open another shell, change to the project root
and run the command:

	> mvn -e -P producer-east

You should see console messages that show the producer connected using the URL

	discovery:(fabric:mq-east)

and the consumer connected using the URL:

	discovery:(fabric:mq-west)
	
<!-- 
  Another way to figure out which container is currently the master is to
  inspect the logs:
  cat instances/MQ-East1/data/log/karaf.log | grep mq-fabric
-->

After the example is up and running and you see JMS messages being logged to the
consumer's console, kill the master broker on the east. You can use the
`cluster-list` command in the karaf to find out which container is currently
the master. For example:

    JBossA-MQ:karaf@root> cluster-list 
    [cluster]                      [masters]                      [slaves]                       [services]
    stats/default                                                                                
    fusemq/mq-east                                                                               
       mq-east-broker              MQ-East2                       MQ-East1                       tcp://chirino-retina.chirino:62184
    fusemq/mq-west                                                                               
       mq-west-broker              MQ-West2                       MQ-West1                       tcp://chirino-retina.chirino:62215

You can stop the master east container using FMC, or kill the container's process
in the OS) and watch the producer failover, reconnect and resume sending
messages, with console output like this:

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

Then kill the container host the west container and observe similar failover and
reconnection by the consumer.