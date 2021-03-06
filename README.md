<p align='right'>
<small>Sunil Samuel<br>
web_github@sunilsamuel.com<br>
http://www.sunilsamuel.com
</small>
</p>

**<h1 align='center'>[Docs] BRMS :: Kie-Server KJar Deployment</h1>**

<!-- BEGIN HEADERS (copy into root page) -->
* [Overview](#overview)
	* <sub>[Introduction](#introduction)</sub>
	* <sub>[Technology Stack](#technology-stack)</sub>
* [Installation](#installation)
	* <sub>[Create a KJar with Rules](#create-a-kjar-with-rules)</sub>
	* <sub>[Install the Kie-Server](#install-the-kie-server)</sub>
		* <sub>[Install EAP 7](#install-eap-7)</sub>
		* <sub>[Install BRMS for EAP 7](#install-brms-for-eap-7)</sub>
		* <sub>[Create User and Role](#create-user-and-role)</sub>
		* <sub>[Start the EAP Server](#start-the-eap-server)</sub>
	* <sub>[Deploy the KJar into Kie-Server Instance](#deploy-the-kjar-into-kie-server-instance)</sub>
	* <sub>[Invoke the ReST Service](#invoke-the-rest-service)</sub>
<!-- END HEADERS (copy into root page) -->
<hr>

# Overview
This documentation provides some detail on deploying kjars into a kie-server.  

## Introduction 
A kjar is essentially a jar file that has additional information within the manifest to help the Drools engine process the rules.  This documentation has details from creating a kjar to deploying this kjar into the kie-server.  The documentation assumes that the reader has some understanding of the Drools (BRMS) engine and the kie-server.  For more information on kie-server, please use the <a href='https://access.redhat.com/documentation/en/red-hat-jboss-bpm-suite/'>Red Hat documentation</a>.

In order to deploy your rules into the kie-server and run rules against your data, you will need to do the following:

1. [Create a KJar with Rules](#create-a-kjar-with-rules)
2. [Install the Kie-Server](#install-the-kie-server)
3. [Deploy the KJar into Kie-Server Instance](#deploy-the-kjar-into-kie-server-instance)
4. [Invoke the ReST Service](#invoke-the-rest-service)

## Technology Stack
1. EAP 7 
2. kie-server
3. maven
4. BRMS (Drools)

# Installation

The following applications need to be installed.

```
BRMS-Rules-KJar
```

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

The following project that I created can be used as a starting point for creating a kjar.

```
BRMS-Rules-KJar
```

Once the project is complete.  Run the mvn install command to install the jar file into the maven m2 directory.  The kie-server deploys your kjar from the maven repository using the artifactid.  Therefore, you must have your kjar installed prior to deploying into kie-server.

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
This will create a directory named `jboss-eap-7.0.0`.  This will also be the directory where the kie-server will be installed.

### Install BRMS for EAP 7
Download the kie-server from the Red Hat access site.<br>
https://access.redhat.com/products/red-hat-jboss-brms/<br>
This documentation uses the *Red Hat JBoss BRMS 6.4.0 Deployable for EAP 7* namely **jboss-brms-6.4.0.GA-deployable-eap7.x.zip** zip file.

NOTE: Make sure to unzip this file into the EAP 7 directory.  It will update the jboss-eap-7.0.0 directory.

```sh
> unzip jboss-brms-6.4.0.GA-deployable-eap7.x.zip
```

### Create User and Role
The kie-server is protected by basic authentication.  Therefore, an appropriate user with the correct role must be created.  The following command can be used to create the user.
```sh
> jboss-eap-7.0/bin/add-user.sh -a --user controllerUser --password controllerUser1234 --role kie-server,rest-all
```

NOTE: Use your own `--user` and `--password` values.

### Start the EAP Server
The EAP server is now ready to be started.  Again, there are several ways to start the server depending on your configuration.  For this documentation, we will use `standalone.sh`.

First, set the `M2_HOME` environment variable to the location of the `.m2` directory.

```sh
> export M2_HOME=/home/sunil/.m2
```
The KJar would have been installed into the `.m2/repository` directory.  This is from where the kie-server will install the kjar.  **That is, the kie-server uses the maven repository to find the kjar and to deploy it into the service**

Now, start EAP.
```sh
> jboss-eap-7.0/bin/standalone.sh
```

Use any browser to access the following page (if you chose the default values).

```
[GET] http://localhost:8080/kie-server/services/rest/server/containers
```

You should see the following output:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response type="SUCCESS" msg="List of created containers">
    <kie-containers/>
</response>
```

The only remaining item is to install a kie-container (deploying the kjar).

## Deploy the KJar into Kie-Server Instance
A KJar is deployed into the kie-server using the kie-server ReSTful services.  All of the kie-server configuration items can be updated using ReSTful services provided by the kie-server.

The container is deployed using the following ReST service using the **PUT** HTTP method.

```
[PUT] http://localhost:8080/kie-server/services/rest/server/containers/[CONTAINER_ID]
```

If you use the example project `BRMS-Rules-KJar`, then use the same name as the container id.  For instance:

```
[PUT] http://localhost:8080/kie-server/services/rest/server/containers/brms-rules-kjar
```

Where the CONTAINER_ID is the name that will be used to invoke the rules. The payload (below) will be used to configure and define the kjar.

The complete documentation can be found on Red Hat access website.

https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.4/html/development_guide/the_rest_api_for_managing_the_realtime_decision_server#unmanaged_intelligent_process_server_environment

At this point, the `.m2` directory should have the kjar as a package and artifact that we want to deploy into the kie-server.  Remember, the kie-server will look in the `M2_HOME/repository` to find the artifact.  Therefore, this kjar must be compiled and installed onto the maven repository.

The **PUT** request will contain the following payload.  Note that the `<release-id>` element will need to be updated to the artifact that you would be deploying.
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
    <!-- This is where you define your kjar as an artifactid so that 
         the kie-server can find and deploy it. -->
    <release-id>
        <artifact-id>BRMS-Rules-KJar</artifact-id>
        <group-id>com.sunilsamuel.brms.rules</group-id>
        <version>0.0.1-SNAPSHOT</version>
    </release-id>
    <scanner poll-interval="5000" status="STARTED"/>
</kie-container>
```
At this point, the rules (kjar) has been deployed and you are ready to send requests (commands) to it.  Re-run the 'container' **GET** command as follows:

```
[GET] http://localhost:8080/kie-server/services/rest/server/containers
```
Now, the ReST service will respond with the following payload:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response type="SUCCESS" msg="List of created containers">
    <kie-containers>
        <kie-container container-id="service-sight-rules" status="STARTED">
            <config-items>
                <itemName>RuntimeStrategy</itemName>
                <itemValue>SINGLETON</itemValue>
                <itemType>java.lang.String</itemType>
            </config-items>
            <config-items>
                <itemName>UpdatePolicy</itemName>
                <itemValue>never</itemValue>
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
                <artifact-id>BRMS-Rules-KJar</artifact-id>
                <group-id>com.tke.servicesight.rules</group-id>
                <version>0.0.1-SNAPSHOT</version>
            </release-id>
            <resolved-release-id>
                <artifact-id>my-example-rules</artifact-id>
                <group-id>com.sunilsamuel.brms.rules</group-id>
                <version>0.0.1-SNAPSHOT</version>
            </resolved-release-id>
            <scanner poll-interval="5000" status="STARTED"/>
        </kie-container>
    </kie-containers>
</response>
```

**NOTE: If you see the following errors in the log file.**

```
[org.eclipse.aether.internal.impl.DefaultUpdatePolicyAnalyzer] (default task-1) Unknown repository update policy '', assuming 'never'
```

This implies that your `settings.xml` file in your `.m2` directory needs to have the &lt;updatePolicy/&gt; element within the &lt;release/&gt;, and &lt;snapshot/&gt; elements.

```xml
<repository>
  <releases>
    <updatePolicy>always</updatePolicy>
    <enabled>true</enabled>
  </releases>
  <snapshots>
    <updatePolicy>always</updatePolicy>
    <enabled>false</enabled>
  </snapshots>
</repository>
```

You are now ready to make requests to the rules engine using the **POST** HTTP Method to process your data.

## Invoke the ReST Service

Now that the KJar is installed, you are ready to execute the rules given your data (facts).  This is done by invoking the instance ReST service using **POST** HTTP Method.  The instance id is the same as the CONTAINER_ID that you defined when you deployed using the **PUT** HTTP Method.

The ReST service to invoke is:

```
[POST] /kie-server/services/rest/server/containers/instances/brms-rules-kjar
```

For instance, given the raw HTTP post request:

```http
POST /kie-server/services/rest/server/containers/instances/brms-rules-kjar HTTP/1.1
Host: localhost:8080
content-type: application/json
accept: application/json
X-KIE-ContentType: JSON
Accept: application/json
Authorization: Basic [your authorization]
Cache-Control: no-cache

{
   "lookup":"defaultStatelessKieSession",
   "commands":[
      {
         "insert":{
            "object":{
               "com.sunilsamuel.brms.model.UserInformation":{
                  "firstName":"Sunil",
                  "lastName":"Samuel",
                  "identifier":2342342,
                  "age":18,
                  "collegeStatus":"PartTime",
                  "familyIncome":46000
               }
            }
         }
      },
      {
         "insert":{
            "object":{
               "com.sunilsamuel.brms.model.UserInformation":{
                  "firstName":"Joel",
                  "lastName":"Samuel",
                  "identifier":234234245,
                  "age":18,
                  "collegeStatus":"FullTime",
                  "familyIncome":43000
               }
            }
         }
      },
      {
         "fire-all-rules":""
      },
      {
         "query":{
            "name":"Query LoanAmount",
            "out-identifier":"loanAmount"
         }
      }
   ]
}
``` 
