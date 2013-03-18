MQ-Fabric Client Example :: Camel Producer
===========================================

This project shows how to connect to JBoss A-MQ message brokers running in Fuse
Fabric from a Camel client running outside of Fuse Fabric (i.e. when the JMS
client is not running within a Fabric-enabled JBoss Fuse ESB container).

Building the examples
---------------------

To build the example, execute the command: 

	> mvn clean install

Running the example
-------------------

Assumes you have setup a fabric-based network of brokers per the instructions in
[fabric-ha-setup.md](https://github.com/FuseByExample/external-mq-fabric-client/blob/master/fabric-ha-setup.md).

After the consumer is running, run this command:

	> mvn camel:run

You should see console messages that show the producer connected using the URL

	discovery:(fabric:mq-east)

<!--
  Another way to figure out which container is currently the master is to
  inspect the logs:

  cat instances/MQ-East1/data/log/karaf.log | grep mq-fabric
-->

After the example is up and running and you see JMS messages being logged to the
producer's console, kill the master broker on the east. You can use the
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
    Sending to destination: queue://fabric.simple this text: 23. message sent
    Sending to destination: queue://fabric.simple this text: 24. message sent
    Sending to destination: queue://fabric.simple this text: 25. message sent
    WARN  Transport (tcp://192.168.0.14:51779) failed, reason:  java.io.EOFException, attempting to automatically reconnect
    Adding new broker connection URL: tcp://mbrooks1.local:51805
    Successfully reconnected to tcp://mbrooks1.local:51805
    Sending to destination: queue://fabric.simple this text: 26. message sent
    Sending to destination: queue://fabric.simple this text: 27. message sent
    ...

Deploying the example in JBoss Fuse
-----------------------------------

In the JBoss Fuse console where you initially created the fabric, run the
following commands to create a `example-camel-producer` profile, and deploy
it to a `Producer` container.

    profile-create --parents activemq-client example-camel-producer
    profile-edit --repositories mvn:org.apache.camel.karaf/apache-camel/2.10.0.redhat-60015/xml/features example-camel-producer
    profile-edit --features activemq,activemq-camel,camel-spring example-camel-producer
    profile-edit --bundles mvn:org.fusebyexample.mq-fabric/camel-producer/2.0.0-SNAPSHOT example-camel-producer
    container-create-child --profile example-camel-producer root Producer
