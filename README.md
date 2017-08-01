# artemis-jms-bridge

An example project showing how to do JMS bridging in Apache Artemis using Apache Camel.

## Requirements

- [Apache Maven 3.x](http://maven.apache.org)
- [Apache Artemis 2.0.0](https://activemq.apache.org/artemis/)

## Preparing

Install and run IBM MQ [[https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.ins.doc/q008640_.htm](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.ins.doc/q008640_.htm)]

_Note: For my tests, I chose to run the docker image [https://hub.docker.com/r/ibmcom/mq/]. You can run it using the command `docker run --name ibmmq --env LICENSE=accept --env MQ_QMGR_NAME=QM1 --publish 1414:1414 --publish 9443:9443 --detach ibmcom/mq`. You can then connect to the web console at [https://localhost:9443/ibmmq/console/]._

Install and run Apache Artemis [[https://activemq.apache.org/artemis/docs/2.0.0/using-server.html](https://activemq.apache.org/artemis/docs/2.0.0/using-server.html)]

Add the InVM acceptor to the Apache Artemis configuration [[https://activemq.apache.org/artemis/docs/2.0.0/configuring-transports.html](https://activemq.apache.org/artemis/docs/2.0.0/configuring-transports.html)]

```xml
<acceptors>
  <acceptor name="in-vm">vm://0</acceptor>
  ...
</acceptors>
```

Build the project source code

```
$ cd $PROJECT_ROOT
$ mvn clean install
```

Install the war

```
$ cp $PROJECT_ROOT/target/artemis-jms-bridge-1.0.0-SNAPSHOT.war $ARTEMIS_HOME/war/artemis-jms-bridge.war
```

Add the war's app configuration to the `$ARTEMIS_INSTANCE/etc/bootstrap.xml`

```xml
<web ...>
  ...
  <app url="artemis-jms-bridge" war="artemis-jms-bridge.war"/>
</web>
```

Install the bridge configuration

```
$ cp $PROJECT_ROOT/src/test/spring/bridge.xml $ARTEMIS_INSTANCE/etc/
```

Install the IBM MQ JMS client libraries

```
$ mkdir $ARTEMIS_INSTANCE/lib/
$ cp $IBM_CLIENT_LIB_DIR/*.jar $ARTEMIS_INSTANCE/lib/
```

Start the Artemis broker

```
$ $ARTEMIS_INSTANCE/bin/artemis run
```

## Testing

You can use the `$ARTEMIS_HOME/bin/artemis` tool to send messages

```
$ $ARTEMIS_HOME/bin/artemis producer --url 'tcp://localhost:61616' --user 'admin' --password 'admin' --message-count 1 --destination 'queue://TEST.FOO' --verbose
```

You should be able to see the message land on the IBM MQ queue using the web console [[https://localhost:9443/ibmmq/console/](https://localhost:9443/ibmmq/console/)]
