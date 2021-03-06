
**Mule4 & Heroku Kafka**: *Throughput Test*
---
***What is this application for?***

This is a simple mule application that can be use to send messages to Heroku Kafka and measure its throughput.  In this application I'm using a single producer with some fine tuning options and depending on your heroku kafka configuration you can have one or more topic(s) with one or more partition(s). You will need to setup Heroku Kafka before start using this application.

![enter image description here](https://i.imgur.com/miqqqUM.jpg)

***How to use this application?***

**STEP 1:**  
---
Setup Heroku Kafka  (using the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli#download-and-install) on a Mac)
-
-- Create an application a Heroku from the terminal:
`heroku create myAppName --team=myTeamName`

-- Provision the application with the Kafka add-on:
*note*:  replace kafkaPlan with the name of the selected [Plan](https://devcenter.heroku.com/articles/kafka-on-heroku#plans-and-configurations))
`heroku addons:create heroku-kafka:kafkaPlan --a myAppName`

-- Get the Kafka Broker URL (save it on a notepad)
`heroku config:get KAFKA_URL —app myAppName` 
*(Format kafka+ssl://host:port, kafka+ssl://host:port, kafka+ssl://host:port....)*

-- Create Keystore and TrustStore:

a) From a terminal, Pass certificates and keys from heroku to local variables:
``client_key=`heroku config:get KAFKA_CLIENT_CERT_KEY --app myAppName` ``  
``client_cert=`heroku config:get KAFKA_CLIENT_CERT --app myAppName` ``  
``trusted_cert=`heroku config:get KAFKA_TRUSTED_CERT --app myAppName` ``  

b) Pass the variable’s value into .PEM files with the following commands:  
`echo "$client_key" >> keystore.pem`  
`echo -n "$client_cert" >> keystore.pem`  
`echo -n "$trusted_cert" > truststore.pem`  

c) At the same terminal, setup a Trust-Store and KeyStore passwords as variables with the commands:  
`TRUSTSTORE_PASSWORD=whatEverYouWant1234!`  
`KEYSTORE_PASSWORD=whatEverYouWant1234!`  

d- Create the TrustStore with the following command:  
`keytool -importcert -file truststore.pem -keystore kafka.client.truststore.jks -deststorepass $TRUSTSTORE_PASSWORD -noprompt`  

e- Create the Key-store in PKCS12 format by submitting the command:  
`openssl pkcs12 -export -in keystore.pem -out keystore.pkcs12 -password pass:$KEYSTORE_PASSWORD
`  

f- Finally, convert the pkcs12 file format in jsk:  
`keytool -importkeystore -srcstoretype PKCS12 -destkeystore kafka.client.keystore.jks -deststorepass $KEYSTORE_PASSWORD -srckeystore keystore.pkcs12 -srcstorepass $KEYSTORE_PASSWORD`

-- Create a Topic (replace 1 with the number of partitions you want)
`heroku kafka:topics:create myTopicName --partitions 1 --app myAppName`

**STEP 2:**
---
Setup the Mule application and connector
  - 
-- Download or Clone this repo
-- Import the project in Anypoint Studio 7.x

-- Place the **kafka.client.keystore.jks** and **kafka.client.truststore.jks** files created in step 1 in the  **src/main/resources** folder.
-- Open the file **src/main/resources/config.yaml**
-- Replace and update the values of the variables:

 - kafka.broker 
 - kafka.topic 
 - kafka.keystore.password
 - kafka.truststore.password

-- Save all changes.


**Mental Notes:**
-- Recommendation: Set the "*clear application data*" to "always" (from the running config)
-- There is no consumer flow configured but the kafka consumer config is already on global.xml in case you want to test. Remember that in this case you need to create the consumer group first in heroku, this can be done only by CLI with the command `heroku kafka:consumer-groups:create consumerGroupName --app HerokuAppName`
-- If you have to keep messages in order you need to do one partition per topic
-- If you need to monitor the cluster and check for consumer lag more closely, there are tools online.. some work others dont with heroku. Im trying to document how to setup each one on  [Mulesoft, Kafka & Heroku](https://salesforce.quip.com/qP1AAKzPbChN)

STEP 4:
Test!
  - 
-- Deploy
-- Call the endpoint http://localhost:8081/kafka
-- Head to your heroku kafka dashboard and note the messages and kb per second.

![1300 messages per second :) ](https://i.imgur.com/Cb0Dyrz.jpg)
---
In the Quip document [Mulesoft, Kafka & Heroku](https://salesforce.quip.com/qP1AAKzPbChN) I document my own findings and use that a a tweaking guide, feel free to collaborate. 

Thanks for your interest and please feel free to provide feedback, update the app to your needs and more!
For more information, please take a look at these Quip documents:

Kudos to Gulal and Ashwani for the support and fedback!

-- [Mulesoft, Kafka & Heroku](https://salesforce.quip.com/qP1AAKzPbChN) by Santiago Moneta

-- [Heroku Kafka: User Guide](https://salesforce.quip.com/WrRRAygxUyt0) by Ashwani Kumar Tripathi

-- [Kafka vs AnypointMQ vs RabbitMQ](https://quip.com/08CZA0FamR6U#ACaACANZsp5) by Gulal Kumar

Thanks, 
Santiago Moneta

