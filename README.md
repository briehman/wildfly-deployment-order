# Wildfly Deployment Order

Wildfly does not appear to be respecting deployment order specified in the EAR's ```application.xml``` nor abiding by the dependencies declared in ```jboss-deployment-structure.xml```.

Sample generated ```application.xml```:
```
<?xml version="1.0" encoding="UTF-8"?>
<application xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/application_6.xsd" version="6">
  <display-name>ear</display-name>
  <initialize-in-order>true</initialize-in-order>
  <module>
    <web>
      <web-uri>webapp-one-1.0-SNAPSHOT.war</web-uri>
      <context-root>/one</context-root>
    </web>
  </module>
  <module>
    <web>
      <web-uri>webapp-two-1.0-SNAPSHOT.war</web-uri>
      <context-root>/two</context-root>
    </web>
  </module>
  <module>
    <web>
      <web-uri>webapp-three-1.0-SNAPSHOT.war</web-uri>
      <context-root>/three</context-root>
    </web>
  </module>
</application>
```

User-specified ```jboss-deployment-structure.xml```:
```
<jboss-deployment-structure xmlns="urn:jboss:deployment-structure:1.2">
    <sub-deployment name="webapp-one-1.0-SNAPSHOT.war">
    </sub-deployment>
    <sub-deployment name="webapp-two-1.0-SNAPSHOT.war">
        <dependencies>
            <module name="deployment.wildfly-deployment-order.ear.webapp-one-1.0-SNAPSHOT.war" />
        </dependencies>
    </sub-deployment>
    <sub-deployment name="webapp-three-1.0-SNAPSHOT.war">
        <dependencies>
            <module name="deployment.wildfly-deployment-order.ear.webapp-two-1.0-SNAPSHOT.war" />
        </dependencies>
    </sub-deployment>
</jboss-deployment-structure>
```

The expected order should be:
* ```webapp-one```
* ```webapp-two```
* ```webapp-three```

Here is how Wildfly deploys the application:
```
09:12:30,602 INFO  MSC service thread 1-2 [deployment] JBAS015876: Starting deployment of "wildfly-deployment-order.ear" (runtime-name: "wildfly-deployment-order.ear")
09:12:30,609 INFO  MSC service thread 1-3 [deployment] JBAS015973: Starting subdeployment (runtime-name: "webapp-two-1.0-SNAPSHOT.war")
09:12:30,609 INFO  MSC service thread 1-3 [deployment] JBAS015973: Starting subdeployment (runtime-name: "webapp-one-1.0-SNAPSHOT.war")
09:12:30,609 INFO  MSC service thread 1-3 [deployment] JBAS015973: Starting subdeployment (runtime-name: "webapp-three-1.0-SNAPSHOT.war")
09:12:30,803 INFO  DeploymentScanner-threads - 1 [server] JBAS018565: Replaced deployment "wildfly-deployment-order.ear" with deployment "wildfly-deployment-order.ear"
```
