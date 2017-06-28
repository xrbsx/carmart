carmart: Example Using Local and Remote Infinispan Cache 
===================================
Author: Martin Gencur, Tristan Tarrant
Level: Intermediate
Technologies: Infinispan, CDI
Summary: The `carmart` quickstart demonstrates how to configure and access Infinispan cache in a simple web application.
Target Product: JDG
Product Versions: JDG 7.x , JWS 3.x
Source: <https://github.com/infinispan/jdg-quickstart>

What is it?
-----------

CarMart is a simple web application that uses Infinispan instead of a relational database.

Users can list cars, add new cars or remove them from the CarMart. Information about each car is stored in a cache. The application also shows cache statistics like stores, hits, retrievals, etc.
 
The CarMart quickstart can work in two modes: 

* _Library mode_  - In this mode, the application and the data grid are running in the same JVM. All libraries (JAR files) are bundled with the application and deployed to Red Hat JBoss Web Server (JWS). The library mode enables fastest (local) access to the entries stored on the same node as the application instance, but also enables access to data stored in remote nodes (JVMs) that comprise the embedded distributed cluster.

* _Client-server mode_ - In this mode, the Cache is stored in a managed, distributed and clusterable data grid server.  Applications can remotely access the data grid server using Hot Rod, memcached or REST client APIs. This web application bundles only the HotRod client and communicates with a remote RedHat JBoss Data Grid (JDG) server. The JDG server is configured via the `standalone.xml` configuration file.


System requirements
-------------------

All you need to build this project is Java SDK 1.8 or better, Maven 3.0 or better.

The application this project produces is designed to be run on Red Hat JBoss Web Server 3 or Tomcat 8. 

 
Configure JWS or Tomcat
-----------------------

Before starting JWS/Tomcat, add the following lines to `conf/tomcat-users.xml` to allow the Maven Tomcat plugin to access the manager application:

        <role rolename="manager-script"/>
        <user username="admin" password="admin" roles="manager-script"/>
        
Configure Maven
---------------

If you have not yet done so, you must [Configure Maven](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_MAVEN.md#configure-maven-to-build-and-deploy-the-quickstarts) before testing the quickstarts.


Add a `<server>` element into your Maven settings.xml with `<id>` equal to tomcat and correct credentials:

        <server>
            <id>tomcat</id>
            <username>admin</username>
            <password>admin</password>
        </server>

        
Start JWS or Tomcat
-------------------

1. Open a command line and navigate to the root of the JWS/Tomcat server directory.
2. The following shows the command line to start the server with the web profile:

        For Linux:   $TOMCAT8_HOME/bin/catalina.sh run
        For Windows: %TOMCAT8_HOME%\bin\catalina.bat run


Build and Deploy the Application in Library Mode
-----------------------------------------------

1. Make sure you have started JWS/Tomcat as described above.
2. Open a command line and navigate to the root directory of this quickstart.
3. Type this command to build and deploy the archive:

        mvn -Plibrary-tomcat clean package tomcat:deploy
        
4. This will deploy `target/jboss-carmart.war` to the running instance of JWS/tomcat.


Access the application
---------------------

The application will be running at the following URL: <http://localhost:8080/jboss-carmart/>


Undeploy the Archive
--------------------

1. Make sure you have started Tomcat/JWS as described above.
2. Open a command line and navigate to the root directory of this quickstart.
3. When you are finished testing, type this command to undeploy the archive:

        mvn -Plibrary-tomcat tomcat:undeploy


Build and Start the Application in Client-Server Mode (using HotRod client)
---------------------------------------------------------------------------

1. Obtain the JDG server distribution. See the following for more information: <http://www.redhat.com/products/jbossenterprisemiddleware/data-grid/>

2. Configure the remote datagrid in the `$JDG_HOME/standalone/configuration/standalone.xml` file. Copy the following XML into the Infinispan subsystem before the ending </cache-container> tag. If you have an existing `carcache` element, be sure to replace it with this one.
       
        <local-cache name="carcache" start="EAGER"/>
   
3. Start the JDG server on localhost using port offset: 
    
        $JDG_HOME/bin/standalone.sh -Djboss.socket.binding.port-offset=100

4. Start JWS or Tomcat into which you want to deploy your application

        $TOMCAT8_HOME/bin/catalina.sh run

5. The application finds the JDG server using the values in the src/main/resources/META-INF/datagrid.properties file. If you are not running the JDG server on the default host and port, you must modify this file to contain the correct values. If you need to change the JDG address:port information, edit src/main/resources/META-INF/datagrid.properties file and specify address and port of the JDG server

        datagrid.host=localhost
        datagrid.hotrod.port=11322

6. Build the application in the example's directory:

        mvn clean package -Premote-tomcat

7. Deploy the application

        mvn tomcat:deploy -Premote-tomcat

8. The application will be running at the following URL: <http://localhost:8080/jboss-carmart/>

9. Undeploy the application

        mvn tomcat:undeploy -Premote-tomcat


Test the Application in Library Mode   
------------------------------------

If you want to test the application, there are simple Arquillian Selenium tests prepared.
To run these tests on Tomcat:

1. Undeploy the archive as described above
2. Stop Tomcat Server
3. Open a command line and navigate to the root directory of this quickstart.
4. Build the quickstart using:

        mvn clean package -Plibrary-tomcat

5. Type this command to run the tests:

        mvn test -Puitests-tomcat -DtomcatHome=/path/to/server
