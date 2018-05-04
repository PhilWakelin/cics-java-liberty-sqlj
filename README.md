cics-java-liberty-sqlj
=====================

Sample SQLJ Java EE web application demonstrating how to use a SQLJ in a web servlet to access DB2.



## Repository structure

* [`projects/`](projects) - Eclipse projects suitable for importing into a CICS Explorer environment.
* [`etc/`](etc) - Liberty server configuration file

## Samples overview

* `com.ibm.cicsdev.sqlj.web` - Dynamic web project containing the SimpleSQLJServlet servlet.  The servlet uses the DoSQLJ class which connects to 
DB2 by obtaining a DataSource via a JNDI lookup and that uses SQLJ and returns the current DB2 timestamp from the DB2 SYSIBM.SYSDUMMY1 table
* `com.ibm.cicsdev.sqlj.web.cicsbundle` - CICS bundle project that references the WAR (Dynamic web project) bundle part for deployment in a CICS bundle

## Pre-requisites
* CICS TS V5.3 with APAR PI77502, or CICS TS V5.4
* Java SE 7 or later on the z/OS system
* IBM Db2 for z/OS
* CICS Explorer V5.4 with the IBM CICS SDK for Java EE and Liberty feature installed [available here](https://developer.ibm.com/mainframe/products/downloads) and 
IBM Data Studio Version 4.1.3 [available here](https://www.ibm.com/developerworks/downloads/im/data) with DS APAR1 installed, [available here](http://www.ibm.com/support/fixcentral/quickorder?product=ibm%2FInformation+Management%2FIBM+Data+Studio&fixids=DS_413_APAR1_v20180413_0111&source=SAR)

## Configuration
The sample code can be deployed as a WAR file into a CICS Liberty JVM server.  CICS Liberty can be configured to use either a local DB2 database with 
JDBC type 2 connectivity,  or a remote database with a JDBC type 4 connectivity. The SimpleSQLJServlet servlet can then be used to display the current timestamp from DB2

### To import the samples into Eclipse
1. Import the projects into CICS Explorer using **File -> Import -> General -> Existing** projects into workspace
1. Resolve the build path errors on the Dynamic web project using the following menu from each project: **Build Path -> Configure Build Path -> Libraries -> Add Library -> CICS with Java EE and Liberty** and select the version of CICS TS for deployment (either CICS TS V5.3 or CICS TS V5.4)

**Note**: The Dynamic web project has `SQLJ support` and when imported into Eclipse, the `Java Resources` folder has a [`src`](projects/com.ibm.cicsdev.sqlj.web/src) folder.  This contains both `.java` and `.sqlj` source files. Additionally, there is a [`SQLJJavaSource`](projects/com.ibm.cicsdev.sqlj.web/SQLJJavaSource) folder.  This contains the Java source and serialized profile (`.ser`) files which are generated automatically by the Data Studio tooling from the `.sqlj` source. 



### To configure CICS for JDBC type 2 connectivity to DB2
1. Create a Liberty JVM server as described in [4 easy steps](https://developer.ibm.com/cics/2015/06/04/starting-a-cics-liberty-jvm-server-in-4-easy-steps/)

1. Update the CICS STEPLIB with the DB2 SDSNLOAD and SDSNLOD2 libraries

1. Configure CICS DB2CONN, DB2TRAN and DB2ENTRY resource definitions as required in [How you can define the CICS DB2 connection](https://www.ibm.com/support/knowledgecenter/en/SSGMCP_5.4.0/configuring/databases/dfhtk2c.html)

1. Bind the DB2 plan that is specified in the CICS DB2CONN or DB2ENTRY definition with a PKLIST of NULLID.* 

1. Add the following properties in the JVM profile to set the location of the DB2 drivers to allow CICS to automatically configure the default DataSource 

     ```
    -Dcom.ibm.cics.jvmserver.wlp.autoconfigure=true
    -Dcom.ibm.cics.jvmserver.wlp.jdbc.driver.location=/usr/lpp/db2v12/jdbc
    ```
    where  ```/usr/lpp/db2v12/jdbc``` is the location of the DB2 JDBC and SQLJ drivers

1. Edit the server.xml and add the DB2 JCC driver to the Liberty global library as shown in the supplied sample [`server.xml`](etc/config/server.xml). 
This enables Web applications to access the required SQLJ Java packages.
    ```xml
    <library id="global">
        <fileset dir="/usr/lpp/db2v12/jdbc/classes" includes="db2jcc4.jar"/>
    </library>
    ```

An example Liberty server configuration of a DataSource with a type 2 connection is supplied in [etc/config/type-2-server.xml](etc/config/type-2-server.xml). Configuration with DataSource and a type 4 connection is in [etc/config/type-4-server.xml](etc/config/type-4-server.xml)

	
### To deploy the sample into a CICS region 
1. Optionally, change the name of the JVMSERVER in the .warbundle file in the com.ibm.cicsdev.sqlj.web.cicsbundle project from DFHWLP to the name of the JVMSERVER resource defined in CICS

1. Using the CICS Explorer export the ```com.ibm.cicsdev.jdbc.web.cicsbundle``` project to a zFS directory

1. Define and install a CICS BUNDLE resource definition referring to the deployed bundle directory on zFS in step 2, and ensure all resources are enabled

## Running the sample
* The servlet is accessed with the following URL:
[http://host:port/com.ibm.cicsdev.sqlj.web/](http://host:port/com.ibm.cicsdev.sqlj.web/)  

If the test is successful, you will see the a response similar to the following written to the browser:  

`SimpleSQLJServlet: DB2 CurrentTimeStamp = 2017-08-02 11:28:46.18055`

## Reference
*  Sample JDBC Git repository  [cics-java-liberty-jdbc](https://github.com/cicsdev/cics-java-liberty-jdbc)
*  CICS Knowledge Center [Configuring a Liberty JVM server](https://www.ibm.com/support/knowledgecenter/SSGMCP_5.4.0/configuring/java/config_jvmserver_liberty.html)
*  CICS Knowledge Center [Configuring a JVM server to support DB2](https://www.ibm.com/support/knowledgecenter/en/SSGMCP_5.4.0/applications/developing/database/dfhtk4b.html)

## License
This project is licensed under [Apache License Version 2.0](LICENSE).

