+++
draft = false
date = 2020-04-21T11:29:11+01:00
title = "Sharing dependencies between Spring Boot Microservices with Maven"
description = ""
slug = ""
tags = ["spring", "spring-boot", "maven", "java", "microservices"]
categories = []
externalLink = ""
series = []
author = "Colin Riddell"
+++


Sharing dependencies between Spring Boot Microservices and projects with Maven for better code-reuse and clean design

-------

## What's the problem

You're looking at microservices, but you realise you want to send an object from one service to the other. You write a class, and then you copy and paste it from the code of one service to the other ... 💡 there must be a better way to do this. You're a good engineer and you know that copying around this POJO is bad practice.

## Setting up a parent POM

We want to setup a common maven project that's imported by any other project that needs its dependencies. At a high level to do that we need to:

* Create a "parent" maven file (`pom.xml`) - this is a `pom.xml` which will sit outside all of the projects that make up our microservices and allow us to tie them together - the goal is to make a `common` project but the easiest way to do that is to first create a parent pom.
* Create a common maven project as one of the child projects.
* Import the common dependency from any other dependencies that need it

In a simple example where a simple "sales-service" is being built and depends on some common project, the file structure will need to reflect this. We will have something like this at the end of this guide.

```tree
.
├── pom.xml     -- the parent pom
├── common
│   ├── pom.xml  -- shared dependencies go in here
│   ├── src
└── sales-service
    ├── pom.xml   -- will be modified to point to our own custom parent pom and import code from common
    └── src
```


### Parent pom.xml

For this to work, the maven `groupId` must be shared across all maven projects in the tree. Each project will need a different `artifactId`, as usual.

It should also describe the children, listing the **directory names** of child projects that should be part of this parent. For this we'll create two children `sales-service` and `common` under the `<modules>` list. Any subsequent child projects must be added to that list.

A very minimal parent `pom.xml` for a project group `com.ordering.services` that has two child modules can be as simple as:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.ordering.services</groupId>
	<artifactId>ordering-services</artifactId>
	<version>1.0-SNAPSHOT</version>
	<packaging>pom</packaging>

	<modules>
		<module>sales-service</module>
		<module>common</module>
	</modules>

</project>
```

Notice that the `packaging` property is set to `pom` - aggregator projects (parent projects) need to be set as POM packaging. This is because when the code is packaged, we're not expecting to see a target JAR produced for the parent package.

### Adding a project

We'll now create and add a spring boot based project for `sales-service` mentioned above. We can create a new Spring Boot project using the [Spring Initializr](https://start.spring.io/). Once generated and brought into the directory structure some small changes need to be made to the sales-service projects' `pom.xml`.

Generated spring boot projects set the maven `parent` attribute to a `spring-boot-starter-parent` artifact.

**This causes some confusion because we need to set the parent to our own new parent project.**

Looking at the newly created `sales-service/pom.xml`  we need to:

* Remove the existing parent which is pointing to spring-boot
* Add our own parent, pointing to the `ordering-services` parent pom.
* refactor the parent section for spring-boot into a regular dependency in the `dependencyManagement` section.

#### Remove the existing parent
Find this code and remove it, including the `parent` tags - versions may differ. Open the pom.xml in `sales-service` or whatever project you generated using the Initilizr.

```xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.14.BUILD-SNAPSHOT</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
```

#### Point to our own parent

We want to set the parent to the project we setup earlier. To do this, we will set the parent to the parents artifact name `ordering-services` and of course the `groupId`. `version` needs to match, too.

```xml
<parent>
	<groupId>com.ordering.services</groupId>
	<artifactId>ordering-services</artifactId>
	<version>1.0-SNAPSHOT</version>
</parent>
```
#### Refactor spring boot into a management dependency

Now the spring-boot-starter-parent dependency is missing. Essentially, the Spring Boot dependency is missing, so it should be added again inside the `dependencyManagement` attribute. The `version` should match that of the one set in the deleted parent dependency setup.


```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.1.14.BUILD-SNAPSHOT</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

```

By this stage a parent project has been setup with one child project (`sales-service`) in it. The parent can see the child, and the child knows it's part of the parent 👩‍👦

## Making use of a `common` dependencies project

Any project that's setup and part of this parent-child relationship will now be able to use each others classes as if they were one big project. Conventionally that's done via creating a `common` project. A common project in a Maven build system is very powerful and saves a lot of configuration. Here are some of the things a common project can do for us:

* Share common classes (models, DTO's etc) - typically good for making sure objects are serialised and de-serialised from the same class.
* Share configuration that needs to be used in more than one project
* Dependencies that are imported by all projects can be refactored into the `pom.xml` of the common project, then any project wanting to use these can just import `common`.

### Setting up common

Create a new spring project with the Initilizr - don't add any dependencies - at least initially. Give it the `groupId` and set the `artifactId` to `common`.



Just like other maven projects that are part of this parent, their parent should be set appropriate (like we done above for the sales-service). Delete the existing parent, and point it to your own parent.

Any generated dependencies can also be removed, we will hand pick any dependencies we want to share across multiple projects and put them in here.

Finally set 1.8  on `maven.compiler.source` and `maven.compiler.target` versions in the `properties` section. A simple `common` (under `common/pom.xml`) project POM will now look something like this:



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.ordering.services</groupId>
	<artifactId>common</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>common</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>com.ordering.services</groupId>
		<artifactId>ordering-services</artifactId>
		<version>1.0-SNAPSHOT</version>
	</parent>

</project>

```
Since there are no test dependencies, be sure to delete any test classes from the common project. One is provided by the spring initializr.

**Also delete the main `CommonApplication.java`** since our common project won't actually be a runnning application but instead a project for sharing common code between projects.

**Also delete any test classes from in the common project**

### Set language versions

Move the  the language version properties into the parent pom (`./pom.xml`):

```xml
<properties>
	<java.version>1.8</java.version>
	<maven.compiler.source>1.8</maven.compiler.source>
	<maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

This means any child project won't need to set these properties now, so remove them.

So far we've setup a tree of dependencies that resembles the following with the descriptions of the changes we've made so far.


```tree
.
├── pom.xml     -- parent pom has modules attribute pointing to `common` + `sales-service`
├── common
│   ├── pom.xml  -- generated from initializr but with dependencies removed and parent set to one above
│   ├── src      -- as generated but with test classes deleted
└── sales-service
    ├── pom.xml   -- generated with desired dependencies but parent set to our own + spring boot dependencies added into the `dependencyManagement` section.
    └── src
```

### Using Common - an example

Adding a class to common will mean that any project in our tree that in turn imports `common` will be able to use that class. This is great, since if we're building with microservices pattern, then we can avoid copy paste of classes that need to be shared across multiple services.

To demonstrate this, create a class called `Monkey` in a new package called `payloads` inside the `common` project. Give `Monkey` a property `name` and generate constructor and getters and setters.

```java
//Monkey.java
public class Monkey {
    private String name;

    public Monkey(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

`Monkey` can be used in any project that imports common. So import the common project from the `sales-service` project. Add the following dependency to `sales-service/pom.xml` in the `dependencies` attribute section:

```xml
<dependency>
	<groupId>com.ordering.services</groupId>
	<artifactId>common</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<scope>compile</scope>
</dependency>
```

Notice that adding this dependency looks like any regular maven dependency that you'd add for a publically available package. But in this case notice that the group is our own group `com.ordering.services` and the artifact is that of the common project.

---
We will create a controller in `sales-service` that will use `Monkey` class (which is defined and lives in `common` to demonstrate this point.

```java
//SalesController.java

@RestController
public class SalesController {

    @GetMapping("/monkeys")
    public ResponseEntity<Monkey> getMonkey(){
        return new ResponseEntity<>(new Monkey("George"), HttpStatus.OK);
    }
}
```

That's it! Notice how the class `Monkey` can be detected in the class-path and easily imported because our `sales-service` project imports the `common` dependency where we defined `Monkey`.


**Very cool** - for any subsiquent projects that need to use the `Monkey` class, they just need to use dependency noted above to be able to do so.

## Note on Generating Jar

If we want to make sure that Jar files are generated for every project in our tree, then the maven property `packaging` should be set to jar for each project we want a jar from when packaging is run. Add this just below the project version.

```xml
	<packaging>jar</packaging>

```

We also need to enable the `repackaging` goal on the `spring-boot-maven-plugin`. This needs to be done if we're going to run the built jars from the command line with `java -jar`.

To do that here, we'll look into `sales-service/pom.xml` and find locate the build secton. Adding an execution to this with the goal name `repackage` will enable this. The whole build attribute section now looks like this:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
	   <artifactId>spring-boot-maven-plugin</artifactId>
	   <executions>                       <!--new-->
	     <execution>                    <!--new-->
		    <goals>                    <!--new-->
			   <goal>repackage</goal> <!--new-->
			</goals>                   <!--new-->
		 </execution>                   <!--new-->
		</executions>                      <!--new-->
	 </plugin>
  </plugins>
</build>
```


## Summary

We now have a project setup with an aggretator (parent) and a common that allows sharing of dependencies between multiple projects that are part of the larger parent group.

We're also able to build the entire suite of projects using maven on the command line with `mvn package` and then run an individual jar from the command line with `java -jar myjar.jar`.

### Check out the code

It is of course [all on github](https://github.com/colin-riddell/spring-maven-shared-dependencies)


I figured this out myself. So there might be an easier or nicer way to do this.. if there is please get in touch - I'd love to know it.

-------

Got feedback on the article? Got questions or corrections... If you want to talk to me, I am really open to meeting new people and learning things. Please reach out..

**Say hi on Twitter** [https://twitter.com/colin_riddell]( 🐦 https://twitter.com/colin_riddell)
