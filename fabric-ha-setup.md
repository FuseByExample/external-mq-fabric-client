Deploying Complex ActiveMQ Networks
===================================

This document provides details on how to configure and deploy a network
of ActiveMQ master/slave networked brokers. We will use the Fuse
Management Console (FMC) to simplify configuring and deploying the 
configuration to multiple broker JVMs.

Download and unpack a JBoss Fuse distribution if you haven't already.
Launch JBoss Fuse by running `bin/fuse` or `bin\fuse.bat`. You should see a
terminal session. Install the FMC by running: 

    fabric:create -p fmc

It will prompt you to create an admin user if you had not already
defined one in `etc/users.properties`. The rest of this document
will assume you have set the user/password combination to admin/admin.

Keep the terminal session running, and connect to
`http://localhost:8181/index.html` in a web browser. Once connected
login using the admin userid that you setup. You should see the
Containers page, with one active container (for the JVM that is running
the FMC)

Click the Profiles tab and have a look at some of the profiles that
already exist, like the mq and mq-base profiles. Profiles aggregate
sets of features, bundles, repositories, properties and configs; when
you assign a profile to a container everything referenced by the
profile is provisioned to the container. So if you just wanted to stand
up a message broker, you could simply assign the pre-existing mq
profile to a container. For this example, we want to create a complex
message broker configuration, so we'll create our own profiles.

We could easily create our custom profiles using the FMC GUI, but we'll
do it even faster by executing some Fuse Fabric commands from the FMC
command line. Information about Fuse Fabric is located here. Docs on
the commands we'll use are here.

Lets start by creating two containers to hold a master/slave broker
pair for an East region, and two containers to hold a master/slave
broker pair for a West region. As I'm running this all on one machine,
I'll make them all child containers (JVMs) of the initial JVM that is
running the FMC. Note that default name of that initial container is
'root'.

Execute this command from the FMC terminal session to create containers
named MQ-East1 and MQ-East2:

    container-create-child root MQ-East 2

Execute this command to create containers named MQ-West1 and MQ-West2:

    container-create-child root MQ-West 2

Execute this command (all on one line) to provision the East containers
with a master/slave broker pair identified with the group name
"mq-east", and networked to the mq-west brokers:

    mq-create --group mq-east --networks mq-west --networks-username admin --networks-password admin --assign-container MQ-East1,MQ-East2 mq-east-broker

Execute this command (all on one line) to provision the West containers
with a master/slave broker pair identified with the group name
"mq-west", and networked to the mq-east brokers:

    mq-create --group mq-west --networks mq-east --networks-username admin --networks-password admin --assign-container MQ-West1,MQ-West2 mq-west-broker

At this point, our network of brokers -- four instances of ActiveMQ, each
running in its own container, networked together and with failover -- is up and]
running!

To verify if everything is working properly lets test running some messages over
the new deployment. We can use the mq-client.jar that's used to verify a
standalone server.

<!-- NOTE: You need an jboss-fuse more recent than the 015 build for the following to work. -->

You can use the `cluster-list` command to see the status of the cluster and find
out which containers were elected to be the masters and which are assigned to
the slaves. Example:

    JBossFuse:karaf@root> cluster-list
    [cluster]                      [masters]                      [slaves]                       [services]
    stats/default                                                                                
    fusemq/mq-east                                                                               
       mq-east-broker              MQ-East2                       MQ-East1                       tcp://chirino-retina.chirino:62184
    fusemq/mq-west                                                                               
       mq-west-broker              MQ-West2                       MQ-West1                       tcp://chirino-retina.chirino:62215

First lets start a consumer running against a broker running in the 'mq-west' group:

    java -jar extras/mq-client.jar consumer --user admin --password admin --brokerUrl 'discovery:(fabric:mq-west)'

Then lets start a producer running against a broker running in the 'mq-east' group:

    java -jar extras/mq-client.jar producer --user admin --password admin --brokerUrl 'discovery:(fabric:mq-east)'
