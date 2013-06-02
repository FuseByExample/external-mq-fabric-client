MQ-Fabric Client Example :: Simple Consumer
===========================================

This project shows how to connect to JBoss A-MQ message brokers running in Fuse
Fabric from a pure JMS client running outside of Fuse Fabric (i.e. when the JMS
client is not running within a Fabric-enabled JBoss Fuse ESB container).

Building the examples
---------------------

To build the example, execute the command: 

	> mvn clean install

Running the example
-------------------

Assumes you have setup a fabric-based network of brokers per the instructions in
[fabric-ha-setup.md](https://github.com/FuseByExample/external-mq-fabric-client/blob/master/fabric-ha-setup.md).

Run this command:

	> mvn exec:java

To figure out which container is currently the master from the command line
is to inspect the logs:

    cat instances/MQ-East1/data/log/karaf.log | grep mq-fabric

After the example is up and running and you see JMS messages being logged to the
consumer's console, kill the master broker on the west. You can use the
`cluster-list` command in the karaf to find out which container is currently
the master. For example:

    JBossA-MQ:karaf@root> fabric:cluster-list
    [cluster]                      [masters]                      [slaves]                       [services]
    stats/default                                                                                
    fusemq/mq-east                                                                               
       mq-east-broker              MQ-East2                       MQ-East1                       tcp://chirino-retina.chirino:62184
    fusemq/mq-west                                                                               
       mq-west-broker              MQ-West2                       MQ-West1                       tcp://chirino-retina.chirino:62215

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
