MQ-Fabric Client Example :: Camel Consumer
===========================================

This project shows how to connect to JBoss A-MQ message brokers running in Fuse
Fabric from a Camel client running outside of Fuse Fabric (i.e. when the JMS
client is not running within a Fabric-enabled JBoss Fuse ESB container).

Building the example
--------------------

To build the example, execute the command: 

	> mvn clean install

Running the example
-------------------

Assumes you have setup a fabric-based network of brokers per the instructions in
[fabric-ha-setup-master-slave.md](../docs/fabric-ha-setup-master-slave.md).

Run this command:

	> mvn camel:run

You should see console messages that show the producer connected using the URL

	discovery:(fabric:amq-west)

<!-- 
  Another way to figure out which container is currently the master is to
  inspect the logs:

  cat instances/AMQ-West1/data/log/karaf.log | grep mq-fabric
-->

After the example is up and running and you see JMS messages being logged to the
consumer's console, kill the master broker on the west. You can use the
`cluster-list` command in the karaf to find out which container is currently
the master. For example:

    JBossA-MQ:karaf@root> cluster-list 
    [cluster]                      [masters]                      [slaves]                       [services]
    stats/default                                                                                
    fusemq/amq-east
       amq-east-profile           AMQ-East2                     AMQ-East1                     tcp://chirino-retina.chirino:62184
    fusemq/amq-west
       amq-west-profile           AMQ-West2                     AMQ-West1                     tcp://chirino-retina.chirino:62215

You can stop the master west container using FMC, or kill the container's process
in the OS) and watch the consumer failover, reconnect and resume consuming
messages, with console output like this:

    ...
    Got 23. message: 23. message sent
    Got 24. message: 24. message sent
    Got 25. message: 25. message sent
    WARN  Transport (tcp://192.168.0.14:51779) failed, reason:  java.io.EOFException, attempting to automatically reconnect
    Adding new broker connection URL: tcp://mbrooks1.local:51805
    Successfully reconnected to tcp://mbrooks1.local:51805
    Got 26. message: 26. message sent
    Got 27. message: 27. message sent
    ...

Deploying the example in JBoss Fuse
-----------------------------------

In the JBoss Fuse console where you initially created the fabric, run the
following commands to create a `example-camel-consumer` profile, and deploy
it to a `Consumer` container.

    fabric:profile-create --parents feature-camel example-camel-consumer
    fabric:profile-edit --repositories mvn:org.apache.activemq/activemq-karaf/\${version:activemq}/xml/features example-camel-consumer
    fabric:profile-edit --features mq-fabric-camel example-camel-consumer
    fabric:profile-edit --bundles mvn:org.fusebyexample.mq-fabric/camel-consumer/2.1.0-SNAPSHOT example-camel-consumer
    fabric:container-create-child --profile example-camel-consumer root Consumer
