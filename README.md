<p align='right'>
<small>Sunil Samuel<br>
web_github@sunilsamuel.com<br>
http://www.sunilsamuel.com
</small>
</p>

**<h1 align='center'>BRMS :: Kie-Server KJar Deployment</h1>**

# Overview
This documentation provides some detail on deploying kjars into a kie-server.  

## Introduction 
A kjar is essentially a jar file that has additional information within the manifest to help the Drools engine process the rules.  This documentation has details from creating a kjar to deploying this kjar into the kie-server.  The documentation assumes that the reader has some understanding of the Drools (BRMS) engine and the kie-server.  For more information on kie-server, please use the <a href='https://access.redhat.com/documentation/en/red-hat-jboss-bpm-suite/'>Red Hat documentation</a>.

In order to deploy your rules into the kie-server and run rules against your data, you will need to do the following:

1. Create a KJar with Rules
2. Install the Kie-Server
3. Deploy the KJar into Kie-Server Instance
4. Invoke the ReST Service

## Technology Stack
1. EAP 7 
2. kie-server
3. maven

# Installation
The following applications need to be installed.

## Create a KJar with Rules
A KJar is created using the following plugin in pom.xml.

```xml
<plugin>
  <groupId>org.kie</groupId>
  <artifactId>kie-maven-plugin</artifactId>
  <version>${kie.version}</version>
<extensions>true</extensions>
</plugin>
```

The following project can be used as a starting point for creating a kjar.

Once the project is complete.  Run the mvn install command to install the jar file into the maven m2 directory.

```sh
> mvn clean install
```
 
## Install the Kie-Server
There are several different ways to install the kie-server.  This documentation will use the EAP server to install the kie server.

### Install EAP 7
Download EAP 7 from the Red Hat access site.<br>
https://access.redhat.com/products/red-hat-jboss-enterprise-application-platform/<br>
This documentation uses **jboss-eap-7.0.0.zip**

Unzip this file into an installation directory
```sh
> unzip jboss-eap-7.0.0.zip
```
This will create a file named `jboss-eap-7.0.0`.  This will be the directory where the kie-server will be installed.

### Install BRMS for EAP 7
Download the kie-server from the Red Hat access site.<br>
https://access.redhat.com/products/red-hat-jboss-brms/<br>
This documentation uses the *Red Hat JBoss BRMS 6.4.0 Deployable for EAP 7* namely **jboss-brms-6.4.0.GA-deployable-eap7.x.zip**

NOTE: Make sure to unzip this file into the EAP 7 directory.  It will update the jboss-eap-7.0.0 directory.

```sh
> unzip jboss-brms-6.4.0.GA-deployable-eap7.x.zip
```

### Create User and Role
The kie-server is protected by basis authentication.  Therefore, an appropriate user with the correct role must be created.  The following command can be used to create the user.
```sh
> jboss-eap-7.0/bin/add-user.sh -a --user controllerUser --password controllerUser1234 --role kie-server,rest-all
```
NOTE: Use an appropriate `--user` and `--password` values.

### Start the EAP Server
The EAP server is now ready to be started.  Again, there are several ways to start the server depending on your configuration.  For this documentation, we will use standalone.sh.

First, set the `M2_HOME` environment variable to the location of the .m2 directory.

```sh
> export M2_HOME=/home/sunil/.m2
```
The KJar would have been installed into the .m2/repository directory.  This is from where the kie-server will install the kjar.

Now, start EAP.
```sh
> jboss-eap-7.0/bin/standalone.sh
```

Use any browser to access the following page.
```
http://localhost:8080/kie-server/services/rest/server/containers
```

You should see the following output:
```xml

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response type="SUCCESS" msg="List of created containers">
    <kie-containers/>
</response>
```

The only remaining item is to install a kie-container (deploying the kjar)

## Deploy the KJar into Kie-Server Instance
A KJar is deployed into the kie-server using the kie-server ReSTful services.  All of the kie-server configuration items can be updated using ReSTful services provided by the kie-server.

The container is deployed using the following ReST service.

```
[PUT] http://localhost:8080/kie-server/services/rest/server/containers/[CONTAINER_ID]
```
Where the CONTAINER_ID is the name of that will be used to invoke the rules and the payload (below) will be used to configure the kjar.

The complete documentation can be found on Red Hat access website.

https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.4/html/development_guide/the_rest_api_for_managing_the_realtime_decision_server#unmanaged_intelligent_process_server_environment

At this point, the .m2 directory will have the kjar as a package and artifact that we want to deploy into the kie-server.

The **PUT** request will contain the following payload.  Note that the `<release-id>` element will be updated to the artifact that you would be deploying.
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<kie-container>
    <config-items>
        <itemName>RuntimeStrategy</itemName>
        <itemValue>SINGLETON</itemValue>
        <itemType>java.lang.String</itemType>
    </config-items>
    <config-items>
        <itemName>MergeMode</itemName>
        <itemValue>MERGE_COLLECTIONS</itemValue>
        <itemType>java.lang.String</itemType>
    </config-items>
    <config-items>
        <itemName>KBase</itemName>
        <itemValue></itemValue>
        <itemType>java.lang.String</itemType>
    </config-items>
    <config-items>
        <itemName>KSession</itemName>
        <itemValue></itemValue>
        <itemType>java.lang.String</itemType>
    </config-items>
    <release-id>
        <artifact-id>my-example-rules</artifact-id>
        <group-id>com.sunilsamuel.example.rules</group-id>
        <version>0.0.1-SNAPSHOT</version>
    </release-id>
    <scanner poll-interval="5000" status="STARTED"/>
</kie-container>
```
