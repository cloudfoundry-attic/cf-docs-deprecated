---
title: Lift
description: Using Lift on Cloud Foundry
tags:
    - scala
    - lift
---

## Background

The blog entry announcing support for [Scala / Lift on CloudFoundry](http://blog.cloudfoundry.com/2011/06/02/cloud-foundry-now-supporting-scala/) has valuable information on getting your Lift application deployed onto CloudFoundry. This KB article provides some additional context and information to help address some common challenges in doing so.

Note that the instructions here have been tested on Lift v2.3 & Scala v2.8.1 with Maven. Please let us know if the process differs substantially with newer versions of Lift as well as Scala.

## Creating a Lift project

You can create a Lift project by following the instructions in the [“Getting Started” section of Lift](http://www.assembla.com/spaces/liftweb/wiki/Getting_Started).
As highlighted there, projects can be set up using [Maven](http://www.assembla.com/wiki/show/liftweb/Using_Maven), [SBT](http://www.assembla.com/wiki/show/liftweb/Using_SBT), [Gradle](http://www.assembla.com/wiki/show/liftweb/Using_Gradle) or [Ant](http://www.assembla.com/spaces/liftweb/wiki/Using_Ant).

## Externalizing service dependencies configuration

Lift applications that do no use any services (such as a database, key value store or a message bus) should run on Cloud Foundry with no changes. Applications that do use a service should externalized the configuration of that service relative to the application.

When starting from the instructions above (say with Maven), the generated project structure will already externalize the needed service configuration for a database. Note that this assumes that you started with a project archetype that generates database artifacts (like 'lift-archetype-basic' or 'lift-archetype-jpa' for Maven). For e.g:

```java
mvn archetype:generate \
 -DarchetypeGroupId=net.liftweb \
 -DarchetypeArtifactId=lift-archetype-basic_2.8.1 \
 -DarchetypeVersion=2.3 \
 -DarchetypeRepository=http://scala-tools.org/repo-releases \
 -DremoteRepositories=http://scala-tools.org/repo-releases \
 -DgroupId=com.company \
 -DartifactId=lift_test \
 -Dversion=1.0
```

The generated project artifacts will contain the configuration that is typically used for local development in $APPLICATION_HOME/src/main/resources/props/default.props. An example of such a file is given below.

```java
db.class=com.mysql.jdbc.Driver
db.url=jdbc:mysql://localhost/my_app
db.user=root
db.pass=root
```

## Resolving the externalized service configuration to use in the application

Lift uses externalized configuration information to facilitate running of the application in various environments with no change to the application. Depending on the environment in which the application is run, a different properties file is used. The rules to determine the properties file to use are specified in the Lift utility class net.util.liftweb.Props. The [Run mode](http://www.assembla.com/wiki/show/liftweb/Run_Modes) of an application specifies the environment it is running in. As long as a properties file named according to the rules exists at $APPLICATION_HOME/src/main/resources/props/<run-mode>.props, it will be used. If not, the default.props file is used. Some examples of property files are [provided here](http://www.assembla.com/spaces/liftweb/wiki/Properties).

Code artifacts that use the external service configuration
Given a project that has been set up with externalized database configuration, the net.util.liftweb.Props utility is used to get the relevant properties. A code example highlighting the use of the Props utility is given below. This is from the $APPLICATION_HOME/src/main/scala/bootstrap/liftweb/Boot.scala file:

```java
def boot {
    if (!DB.jndiJdbcConnAvailable_?) {
      val vendor =
            new StandardDBVendor(Props.get("db.class") openOr "Invalid Class",
                 Props.get("db.url") openOr "Invalid DB url",
                 Props.get("db.user"),
                             Props.get("db.pass"))

      LiftRules.unloadHooks.append(vendor.closeAllConnections_! _)

      DB.defineConnectionManager(DefaultConnectionIdentifier, vendor)
    }
```

In the code above, the run mode is used to derive the properties file to use and based on that file, the associated properties are read in to connect to the database.

## Auto-reconfiguration of database configuration during Lift application deployment into Cloud Foundry

As long as the application is using externalized service configuration as indicated above and has bound a database service instance of Cloud Foundry (mysql) deploying the application into Cloud Foundry (using 'vmc' or with STS) should be straight forward.
Auto-reconfiguration and starting of the application will result in a new run mode properties file following the pattern of $APPLICATION_HOME/src/main/resources/props/<username>.<hostname>.props where the <username> corresponds to the user deploying the application, <hostname> to the host where the application is started and the db.* values to those of the database service instance end point that is bound to the application. The Lift application should start up be connected to the database at this point.

## Manual binding of database and other service configuration in Lift

Applications that do not externalize their configuration (i.e the configuration is hard-code in the code) or are using services other than databases (currently, Cloud Foundry auto reconfiguration only works for a single instance of the database service for Lift), need to explicitly resolve their configuration dependencies.
To do so, the Lift project needs to add the Cloud Foundry runtime to its dependencies. This runtime is a Java package that provides access to all of the service end point information available to the user. To include the dependency in Maven, you have to add to the following elements [properties and dependencies] in the pom.xml. Ideally, the latest version of the runtime should be used.

```xml
...
  <properties>
    ...
    <org.cloudfoundry-version>0.6.1</org.cloudfoundry-version>
    ...
  </properties>

  ...

  <dependencies>
    ...
    <!-- CloudFoundry -->
    <dependency>
      <groupId>org.cloudfoundry</groupId>
      <artifactId>cloudfoundry-runtime</artifactId>
      <version>${org.cloudfoundry-version}</version>
    </dependency>
    ...
  </dependencies>
  ...
```

To get at the service properties in your code, you would need to use the cloud runtime as follows:

import the environment with a "import org.cloudfoundry.runtime.env._"
import the service specific runtime package such as "org.cloudfoundry.runtime.service.relational._"
Finally you can invoke a method to create a service and get a connection to it.
An example usage is given below:

```java
object DBVendor extends ConnectionManager {
  def newConnection(name: ConnectionIdentifier): Box[Connection] = {
    try {

      import org.cloudfoundry.runtime.env._
      import org.cloudfoundry.runtime.service.relational._
      Full(new MysqlServiceCreator(new CloudEnvironment()).createSingletonService().service.getConnection())

    } catch {
      case e : Exception => e.printStackTrace; Empty
    }
  }
```