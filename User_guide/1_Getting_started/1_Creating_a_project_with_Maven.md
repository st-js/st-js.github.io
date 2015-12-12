---
title: 1.1. Creating an ST-JS project using Maven
---

In this Chapter, you will learn how to build a "Hello World" application using ST-JS. In order to follow this
chapter knowledge of Maven, Java, JavaScript and HTML is required. We will configure this Hello World project
as a standard Java EE web application .war file, but given that the project contains only static content, you could
just extract the resulting .war and serve the files with your favorite web server (NodeJS, nginx, apache, IIS,
etc...)

This chapter demonstrates the usage of ST-JS with maven, but you can also use other build systems. The
documentation for the various supported build systems is documented in chapter
<a href="{{userguide.building.url}}">{{userguide.building.title}}</a>.

Instead of following all the steps in this chapter manually, you can also clone the following github repository:
<a href="https://github.com/st-js/bridges-examples/tree/master/html-hello">https://github.com/st-js/bridges-examples/tree/master/html-hello</a>



1. ## Create the project descriptor (pom.xml)

Create a file named `pom.xml` in the root folder of your project with the following content:

{% highlight xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                            http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.st-js</groupId>
    <artifactId>hello</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <!-- We package our Hello World as a webapp -->
    <packaging>war</packaging>

    <name>Hello world</name>

    <properties>
        <!-- Declare the version of ST-JS in a property to make it easier to keep the versions of the various -->
        <!-- ST-JS artifacts in sync -->
        <stjs.version>{{last_stjs_version}}</stjs.version>

        <!-- Configure the version of the Java compiler to use for our project. 1.6 is the minimum supported -->
        <!-- by ST-JS, but we strongly recommend you to use 1.8 to make full use of the newest language Java -->
        <!-- improvements. -->
        <java.compiler.source>1.8</java.compiler.source>
        <java.compiler.target>1.8</java.compiler.target>
    </properties>

    <dependencies>
        <!-- We depend on the standard JavaScript/HTML/DOM bridge that exposes the JavaScript standard API as -->
        <!-- well as the DOM API to Java so that we can directly use it in our ST-JS code -->
        <dependency>
            <groupId>org.st-js.bridge</groupId>
            <artifactId>html</artifactId>
            <version>{{stjs_bridges.html.version}}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>

            <!-- Configure the ST-JS maven plugin that will generate the JavaScript output from our Java source -->
            <!-- The plugin must be configured to run the stjs:generate goal after the Java compilation has run -->
            <plugin>
                <groupId>org.st-js</groupId>
                <artifactId>stjs-maven-plugin</artifactId>
                <version>${stjs.version}</version>
                <executions>
                    <execution>
                    <id>main</id>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- Configure a web server to run our Hello World application. The easiest to use with Maven -->
            <!-- is Jetty embedded -->
            <plugin>
                <groupId>org.mortbay.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <configuration>
                    <webAppConfig>
                        <contextPath>/HelloWorld</contextPath>
                        <baseResource implementation="org.eclipse.jetty.util.resource.ResourceCollection">
                            <resourcesAsCSV>
                                src/main/webapp,
                                target/${project.artifactId}-${project.version}
                            </resourcesAsCSV>
                        </baseResource>
                    </webAppConfig>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
{% endhighlight %}



2. ## Create the webapp descriptor (web.xml)

At the root of your project, create the following directory structure: `src/main/webapp/WEB-INF`

In the `src/main/webapp/WEB-INF` folder, create a file named `web.xml` with the following content:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
        xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
                            http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <!-- This file is otherwise empty. Our HelloWorld project doesn't contain any code that runs -->
    <!-- on the server side (after compilation, it's only made of static assets) so there is -->
    <!-- nothing to configure -->

</web-app>
{% endhighlight %}



3. ## Add the home page

In the `src/main/webapp` folder create a new file named `index.html`. This is our home page.

{% highlight html %}
<html>
<body>
    <h1>ST-JS example: Hello world</h1>

    <form id="form">
        Say hello to:
            <input type="text" name="to">
            <input type="button" name="say" value="Go">
    </form>
</body>
</html>
{% endhighlight %}



4. ## Build and run the project

Run the project by invoking maven from the command line

{% highlight bash %}
# build the project from scratch
mvn clean install

# Alternately, re-run only ST-JS. Keep this command in mind, it is really useful
# to shorten your edit-compile-run cycle while developing. You can also run this
# command from another terminal while your web server is running to avoid having to
# restart it all the time
mvn compile stjs:generate

# start the embedded Jetty server
mvn jetty:run
{% endhighlight %}

You can now point your web browser to the following URL to see your new Hello World application:
<a href="http://localhost:8080/HelloWorld/index.html">http://localhost:8080/stjs/index.html</a>

The skeleton of your HelloWorld project is now setup. In the next step of this guide, you will write
your first lines of JavaScript in Java using ST-JS
