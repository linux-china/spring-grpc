# Spring gRPC
!["Build Status"](https://github.com/spring-projects-experimental/spring-grpc/actions/workflows/deploy.yml/badge.svg)

Welcome to the Spring gRPC project!

The Spring gRPC project provides a Spring-friendly API and abstractions for developing gRPC applications. There is a core library that makes it easy to work with gRPC and dependency injection, and a Spring Boot starter that makes it easy to get started with gRPC in a Spring Boot application (with autoconfiguration and configuration properties, for instance).

For further information go to our [Spring gRPC reference documentation](https://docs.spring.io/spring-grpc/reference/).

# Getting Started

This section offers jumping off points for how to get started using Spring gRPC. There is a simple sample project in the `samples` directory (e.g. [`grpc-server`](https://github.com/spring-projects-experimental/spring-grpc/tree/main/samples/grpc-server)). You can run it with `mvn spring-boot:run` or `gradle bootRun`. You will see the following code in that sample.

You should follow the steps in each of the following section according to your needs.

**📌 NOTE**\
Spring gRPC supports Spring Boot 3.3.x

## Add Milestone and Snapshot Repositories

If you prefer to add the dependency snippets by hand, follow the directions in the following sections.

To use the Milestone and Snapshot version, you need to add references to the Spring Milestone and/or Snapshot repositories in your build file.

For Maven, add the following repository definitions as needed (if you are using snapshots or milestones):

```xml
  <repositories>
    <repository>
      <id>spring-milestones</id>
      <name>Spring Milestones</name>
      <url>https://repo.spring.io/milestone</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
    <repository>
      <id>spring-snapshots</id>
      <name>Spring Snapshots</name>
      <url>https://repo.spring.io/snapshot</url>
      <releases>
        <enabled>false</enabled>
      </releases>
    </repository>
  </repositories>
```

For Gradle, add the following repository definitions as needed:

```groovy
repositories {
  mavenCentral()
  maven { url 'https://repo.spring.io/milestone' }
  maven { url 'https://repo.spring.io/snapshot' }
}
```

## Dependency Management

The Spring gRPC Dependencies declares the recommended versions of all the dependencies used by a given release of Spring gRPC.
Using the dependencies from your application’s build script avoids the need for you to specify and maintain the dependency versions yourself.
Instead, the version you’re using determines the utilized dependency versions.
It also ensures that you’re using supported and tested versions of the dependencies by default, unless you choose to override them.

If you’re a Maven user, you can use the dependencies by adding the following to your pom.xml file -

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-grpc-dependencies</artifactId>
            <version>0.2.0-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Gradle users can also use the Spring gRPC Dependencies by leveraging Gradle (5.0+) native support for declaring dependency constraints using a Maven BOM.
This is implemented by adding a 'platform' dependency handler method to the dependencies section of your Gradle build script.
As shown in the snippet below this can then be followed by version-less declarations of the Starter Dependencies for the one or more spring-grpc modules you wish to use, e.g. spring-grpc-openai.

```gradle
dependencies {
  implementation platform("org.springframework.ai:spring-grpc-dependencies:0.2.0-SNAPSHOT")
}
```

You need a Protobuf file that defines your service and messages, and you will need to configure your build tools to compile it into Java sources. This is a standard part of gRPC development (i.e. nothing to do with Spring). We now come to the Spring gRPC features.

## gPRC Server

Create a `@Bean` of type `BindableService`. For example:

```java
@Service
public class GrpcServerService extends SimpleGrpc.SimpleImplBase {
...
}
```

(`BindableService` is the interface that gRPC uses to bind services to the server and `SimpleImplBase` was created for you from your Protobuf file.)

Then, you can just run your application and the gRPC server will be started on the default port (9090). Here’s a simple example (standard Spring Boot application):

```java
@SpringBootApplication
public class GrpcServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(GrpcServerApplication.class, args);
	}
}
```

Run it from your IDE, or on the command line with `mvn spring-boot:run` or `gradle bootRun`.

## gRPC Client

To create a simple gRPC client, you can use the Spring Boot starter (see above - it’s the same as for the server). Then you can inject a bean of type `GrpcChannelFactory` and use it to create a gRPC channel. The most common usage of a channel is to create a client that binds to a service, such as the one above. The Protobuf-generated sources in your project will contain the stub classes, and they just need to be bound to a channel. For example, to bind to the `SimpleGrpc` service on a local server:

```java
@Bean
SimpleGrpc.SimpleBlockingStub stub(GrpcChannelFactory channels) {
	return SimpleGrpc.newBlockingStub(channels.createChannel("0.0.0.0:9090").build());
}
```

Then you can inject the stub and use it in your application.

The default `GrpcChannelFactory` implementation can also create a "named" channel, which you can then use to extract the configuration to connect to the server. For example:

```java
@Bean
SimpleGrpc.SimpleBlockingStub stub(GrpcChannelFactory channels) {
	return SimpleGrpc.newBlockingStub(channels.createChannel("local").build());
}
```

then in `application.properties`:

```properties
spring.grpc.client.channels.local.address=0.0.0.0:9090
```

There is a default named channel (named "default") that you can configure in the same way, and then it will be used by default if there is no channel with the name specified in the channel creation.
