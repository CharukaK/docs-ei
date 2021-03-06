# Supporting Different Transports

Follow the relevant section for the steps that need to be carried out
before using the required transport to receive and publish events via
WSO2 SP.

-   [Kafka
    transport](#SupportingDifferentTransports-KafkatransportKafka)
-   [JMS transport](#SupportingDifferentTransports-JMStransportJMS)
-   [MQTT transport](#SupportingDifferentTransports-MQTTtransportMQTT)
-   [RabbitMQ
    transport](#SupportingDifferentTransports-RabbitMQtransport)

### Kafka transport

To enable WSO2 SP to receive and publish events via the Kafka transport,
follow the steps below:

1.  Download the Kafka broker from
    [here](https://kafka.apache.org/downloads) .
2.  Convert and copy the Kafka client jars from the
    `          <KAFKA_HOME>/libs         ` directory to the
    `          <SP_HOME>/lib         ` directory as follows.
    1.  Create a directory in a preferred location in your machine and
        copy the following JARs to it from the
        `             <KAFKA_HOME>/libs            ` directory.

                !!! info
        
                This directory will be referred to as the
                `             SOURCE_DIRECTORY            ` in the next steps.
        

        -   `              kafka_2.11-0.10.2.1.jar                           `
        -   `              kafka-clients-0.10.2.1.jar                           `
        -   `              metrics-core-2.2.0.jar                           `
        -   `              scala-library-2.11.8.jar                           `
        -   `              scala-parser-combinators_2.11-1.0.4.jar                           `
        -   `              zkclient-0.10.jar                           `
        -   `               zookeeper-3.4.9.jar              `

    2.  Create another directory in a preferred location in your
        machine.

                !!! info
        
                This directory will be referred to as the
                `             DESTINATION_DIRECTORY            ` in the next
                steps.
        

    3.  To convert all the Kafka jars you copied into the
        `            <SOURCE_DIRECTORY>           ` , issue the
        following command.
        -   **For Windows:**
            `              <SP_HOME>/bin/jartobundle.bat <SOURCE_DIRECTORY_PATH> <DESTINATION_DIRECTORY_PATH>             `
        -   **For Linux** :
            `              <SP_HOME>/bin/jartobundle.sh <SOURCE_DIRECTORY_PATH> <DESTINATION_DIRECTORY_PATH>             `
    4.  Copy the converted files from the
        `             <DESTINATION_DIRECTORY>            ` to the
        `             <SP_HOME>/lib            ` directory.

    5.  Copy the jars that are not converted from the
        `            <SOURCE_DIRECTORY>           ` to the
        `            <SP_HOME>/samples/sample-clients/lib           `
        directory.

3.  The Kafka server should be started before sending events from WSO2
    SP to a Kafka consumer.

### JMS transport

Follow the steps to configure the **Apache ActiveMQ** message broker:

1.  Install [Apache ActiveMQ JMS](http://activemq.apache.org/) .

        !!! info
    
        This guide uses ActiveMQ versions 5.7.0 - 5.9.0. If you want to use
        a later version, for instructions on the necessary changes to the
        configuration steps, go to [Apache ActiveMQ
        Documentation](http://activemq.apache.org/activemq-580-release.html)
        .
    

2.  Download the `          activemq-client-5.x.x.jar         ` from
    [here](http://central.maven.org/maven2/org/apache/activemq/activemq-client/5.9.0/activemq-client-5.9.0.jar)
    .
3.  Register the `           InitialContextFactory          `
    implementation according to the OSGi JNDI spec and copy the client
    jar to the `           <SP_HOME>/lib          ` directory as
    follows.

    1.  Navigate to the `             SP_HOME>/bin            `
        directory and issue the following command.

        -   **For Linux** :
            `              ./icf-provider.sh org.apache.activemq.jndi.ActiveMQInitialContextFactory <Downloaded Jar Path>/activemq-client-5.x.x.jar <Output Jar Path>             `
        -   **For Windows** :
            `              ./icf-provider.bat org.apache.activemq.jndi.ActiveMQInitialContextFactory <Downloaded Jar Path>\activemq-client-5.x.x.jar <Output Jar Path>             `

                !!! info
        
                If required, you can provide privileges via
                `             chmod +x icf-provider.(sh|bat)            ` .
        

          
        Once the client jar is successfully converted, the
        `             activemq-client-5.x.x            ` directory is
        created. This directory contains the following:  

        -   `              activemq-client-5.x.x.jar             `
            (original jar)  
        -   `              activemq-client-5.x.x_1.0.0.jar             `
            (OSGi-converted jar)

        In addition, the following messages are logged in the
        terminal.  

        ``` java
                INFO: Executing 'jar uf <absolute_path>/activemq-client-5.x.x/activemq-client-5.x.x.jar -C <absolute_path>/activemq-client-5.x.x /internal/CustomBundleActivator.class' [timestamp] org.wso2.carbon.tools.spi.ICFProviderTool addBundleActivatorHeader - INFO: Running jar to bundle conversion [timestamp] org.wso2.carbon.tools.converter.utils.BundleGeneratorUtils convertFromJarToBundle - INFO: Created the OSGi bundle activemq_client_5.x.x_1.0.0.jar for JAR file <absolute_path>/activemq-client-5.x.x/activemq-client-5.x.x.jar
        ```

    2.  Copy
        `            activemq-client-5.x.x/activemq-client-5.x.x.jar           `
        and place it in the
        `            <SP_HOME>/samples/sample-clients/lib           `
        directory.
    3.  Copy
        `            activemq-client-5.x.x/activemq_client_5.x.x_1.0.0.jar           `
        and place it in the `            <SP_HOME>/lib           `
        directory.

4.  Create a directory in a preferred location in your machine and copy
    the following JARs to it from the
    `           <ActiveMQ_HOME>/libs          ` directory.

        !!! info
    
        This directory is referred to as the
        `           SOURCE_DIRECTORY          ` in the next steps.
    

    -   `            hawtbuf-1.9.jar           `
    -   `            geronimo-jms_1.1_spec-1.1.1.jar           `
    -   `            geronimo-j2ee-management_1.1_spec-1.0.1.jar           `

5.  Create another directory in a preferred location in your machine.

        !!! info
    
        This directory will be referred to as the
        `           DESTINATION_DIRECTORY          ` in the next steps.
    

6.  To convert all the Kafka jars you copied into the
    `          <SOURCE_DIRECTORY>         ` , issue the following
    command.
    -   **For Windows:**
        `            <SP_HOME>/bin/jartobundle.bat <SOURCE_DIRECTORY_PATH> <DESTINATION_DIRECTORY_PATH>           `
    -   **For Linux** :
        `            <SP_HOME>/bin/jartobundle.sh <SOURCE_DIRECTORY_PATH> <DESTINATION_DIRECTORY_PATH>           `
7.  Copy the converted files from the
    `           <DESTINATION_DIRECTORY>          ` to the
    `           <SP_HOME>/lib          ` directory.

8.  Copy the jars that are not converted from the
    `          <SOURCE_DIRECTORY>         ` to the
    `          <SP_HOME>/samples/sample-clients/lib         ` directory.
    `                   `

### MQTT transport

Follow the steps to configure the **MQTT** message broker:

1.  Download the
    `          org.eclipse.paho.client.mqttv3-1.1.1.jar         ` file
    from here.
2.  `           Place the file you downloaded in the           <SP_HOME>/lib          `
    directory.

### RabbitMQ transport

Follow the steps below to configure the RabbitMQ message broker:

1.  Download RabbitMQ from here.
2.  Create a directory in a preferred location in your machine. This
    directory is referred to as at the \<SOURCE\_DIRECTORY\> in the rest
    of the procedure.
3.  Copy the following files from the
    `          <RabbitMQ_HOME>/plugins         ` directory to the
    `          <SOURCE_DIRECTORY>         ` you created.
4.  Create another directory in a preferred location in your
    machine. This directory is referred to as the
    `          <DESTINATION_DIRECTORY>         ` in this procedure.
