# Spring boot tutorial

This guide will present how a simple spring boot application might get built from scratch.

## Goal

...


## Generate boilerplate

Spring comes with a nice little application 'initializr' (https://start.spring.io) which enables us to create spring application boilerplates from command line.
For this little tutorial we need some simple dependencies which must be included. Simply execute the following command in the desired folder: 

```bash
$ curl https://start.spring.io/starter.tgz -d dependencies=web,data-jpa,actuator,h2 -d type=gradle-project -d javaVersion=11 -d groupId=ch.puzzle -d artifactId=serviceApi -d baseDir=serviceApi| tar -xzvf -
```

The command creates a new java spring project built for gradle. It comes as specified in java version 11. The given dependencies allow us to create a full working web application (spring web starter), some endpoints to monitor the health of our application (spring actuator) and the boilerplate for enabling JPA and Hibernate (spring data-jpa) and a simple h2 database connection (h2).

In our terminal we can verify if the application works by simply executing 

```bash
$ ./gradlew bootRun
```

which starts our created application on default port 8080. 

## REST, Controller, Services, Repositories, what?

Our basic application will follow the REST architectural style (https://restfulapi.net/). Please read about RESTful design principles before continueing. 
The spring framework provides us with a lot of convenient features, the first features you're going to face and getting to know are the RestController, Services and Repositories. From top to bottom:
  - (Rest)Controller: These Controllers serve endpoints and map http requests to functions in our application.
  - Services: Services are providers of logic, in general they are called by the Controller and provide them with data.
  - Repositories: Repositories build the connection from our application to the database with help of the java persistance api (JPA) and Hibernate

## Our first endpoint

First we will create a simple REST endpoint with help of the spring framework which just returns a simple 'Hello World' to the user.

Let's create our first Rest-Controller class. 

```bash

$ touch src/main/java/ch/puzzle/serviceApi/HelloController.java

``` 

And change it's content as shown below: 

HelloController.java
```java
package ch.puzzle.serviceApi;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping
    public String helloWorld() {
        return "Hello World";
    }
}
``` 

So what is actually happening here? We are in fact registering a rest-controller in our application which we can call on the endpoint `/hello` of our application. This endpoint serves one HTTP-Method which we can use with a GET request on the controller's basepath `/hello`. 
So if we start the application again with `$ ./gradlew bootRun` and then send an HTTP GET request to `localhost:8080/hello` we will receive the string "Hello World" as response. 

Let's try to go a bit further and use a parameter on a new GET mapping in this controller. We can create another function `helloUser(@PathVariable String username)` which should greet the user with his username passed as a path variable.

```java
package ch.puzzle.serviceApi;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping
    public String helloWorld() {
        return "Hello World";
    }

    @GetMapping("/{username}")
    public String helloUser(@PathVariable String username) {
        return "Hello " + username;
    }
}
```

If we send an HTTP GET request now to `localhost:8080/hello/TestUser` we will receive the response "Hello TestUser"!

## And now with actual data

To demonstrate the full stack we do first need a usecase. We will have in our application a table full of services, called daemons, available in our network which will be represented by the model and class `Daemon.java`. The `DaemonRepository` will be the repository to access the data and will be accessed by the `DaemonService` which will be exposed by the `DaemonController` as a REST-endpoint (e.g. `HTTP-GET: /api/daemon`). 

Were going to create the conventional folder structure first. 

```bash
$ cd serviceApi

$ mkdir -p src/main/java/ch/puzzle/serviceApi/daemon/controller src/main/java/ch/puzzle/serviceApi/daemon/service src/main/java/ch/puzzle/serviceApi/daemon/model

$ touch src/main/java/ch/puzzle/serviceApi/daemon/controller/DaemonController.java src/main/java/ch/puzzle/serviceApi/daemon/service/DaemonService.java src/main/java/ch/puzzle/serviceApi/daemon/service/DaemonRepository.java src/main/java/ch/puzzle/serviceApi/daemon/model/Daemon.java

```

The folder structure should look now like this: 

```bash
└── src
    ├── main
    │   ├── java
    │   │   └── ch
    │   │       └── puzzle
    │   │           └── serviceApi
    │   │               ├── daemon
    │   │               │   ├── controller
    │   │               │   │   └── DaemonController.java
    │   │               │   ├── model
    │   │               │   │   └── Daemon.java
    │   │               │   └── service
    │   │               │       └── DaemonService.java
    │   │               │   │   └── DaemonRepository.java
    │   │               └── DemoApplication.java
```

### Our first model 

We will create a simple data model for our daemons which looks like this: 

/model/Daemon.java
```java
package ch.puzzle.serviceApi.daemon.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Daemon {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private int port;
    private String description;

    public Daemon() { }

    public Daemon(String name, int port, String description) {
        this.name = name;
        this.port = port;
        this.description = description;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPort() {
        return port;
    }

    public void setPort(int port) {
        this.port = port;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

We annotate the daemon class with `@Entity` to enable JPA and Hibernate to work their magic and create mappings and tables for the class. But except for the annotation there is nothing special happening here. With the `@Id` annotation we indicate that the `id:Long` will be used as primary key and with `@GeneratedValue` we indicate that it will be automatically generated and incremented.

Let's create our first service and repository to access the entity. 

/service/DaemonService.java
```java
package ch.puzzle.serviceApi.daemon.service;

import ch.puzzle.serviceApi.daemon.model.Daemon;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class DaemonService {

    private final DaemonRepository daemonRepository;

    public DaemonService(DaemonRepository daemonRepository) {
        this.daemonRepository = daemonRepository;
    }

    public List<Daemon> getAllDaemons() {
        return daemonRepository.findAll();
    }
}
```

/service/DaemonRepository.java
```java
package ch.puzzle.serviceApi.daemon.service;

import ch.puzzle.serviceApi.daemon.model.Daemon;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface DaemonRepository extends CrudRepository<Daemon, Long> {
    List<Daemon> findAll();
}
```

 This is where the magic starts to happen. The classes annotated with `@Service` or `@Respository` will automatically instanciated by the spring context as a singleton (https://sourcemaking.com/design_patterns/singleton) in our application context. The singletons will be automatically provided with dependency injection (https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-spring-beans-and-dependency-injection.html) and therefore the service and repository can be used to serve data in the controller. Due to the help of JPA and spring the data can be easily accessed with the repository. 

 Let's expose our data with a controller:

/controller/DaemonController.java
 ```java
 package ch.puzzle.serviceApi.daemon.controller;

import ch.puzzle.serviceApi.daemon.model.Daemon;
import ch.puzzle.serviceApi.daemon.service.DaemonService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("api/daemons")
public class DaemonController {

    private final DaemonService daemonService;

    public DaemonController(DaemonService daemonService) {
        this.daemonService = daemonService;
    }

    @GetMapping
    public List<Daemon> getAllDaemons() {
        return daemonService.getAllDaemons();
    }
}
```

Now here is again more spring magic happening. By annotating the controller with `@RestController` we indicate that this will be a singleton which provides REST endpoints which will be exposed. With the `@RequestMapping` we map the requests coming to /api/daemons to this controller and it's functions. The annotation `@GetMapping` does indicate that all HTTP-GET requests to the basepath will call the function `DaemonController::getAllDaemons` and automatically returns the response as a JSON list of daemons to the user. 

So if you followed along you can start the application again with `$ ./gradlew bootRun` and send a GET request to localhost:8080/api/daemons and see that there is a response with content "[]". Now for demonstration purposes extend our classes as shown: 

/controller/DaemonController.java
```java
package ch.puzzle.serviceApi.daemon.controller;

import ch.puzzle.serviceApi.daemon.model.Daemon;
import ch.puzzle.serviceApi.daemon.service.DaemonService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("api/daemons")
public class DaemonController {

    private final DaemonService daemonService;

    public DaemonController(DaemonService daemonService) {
        this.daemonService = daemonService;
    }

    @GetMapping
    public List<Daemon> getAllDaemons() {
        Daemon daemon = new Daemon("TestDaemon", 8081, "A test daemon running on port 8081");
        daemonService.saveDaemon(daemon);

        return daemonService.getAllDaemons();
    }
}

```

/service/DaemonService.java
```java
package ch.puzzle.serviceApi.daemon.service;

import ch.puzzle.serviceApi.daemon.model.Daemon;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class DaemonService {

    private final DaemonRepository daemonRepository;

    public DaemonService(DaemonRepository daemonRepository) {
        this.daemonRepository = daemonRepository;
    }

    public List<Daemon> getAllDaemons() {
        return daemonRepository.findAll();
    }

    public Daemon saveDaemon(Daemon daemon) {
        return daemonRepository.save(daemon);
    }
}
```

So this time we will actually create an daemon entity, saving the entity into the database and then showing all entities in our database as JSON returned to the user. 
If you start the application again and repeat the last HTTP request you will get the following result: 

```JSON 
[
    {
        "id": 1,
        "name": "TestDaemon",
        "port": 8081,
        "description": "A test daemon running on port 8081"
    }
]
``` 

If you repeat the request a second, third ... time you will notice that the controller creates a new daemon saved to the database for each request. 

## To the cloud

In the last few years all the application migrated from VM's to containers. From barebone metal servers into the infamous cloud. I hope you are familiar with the idea of containers and docker. If not, please familiarize yourself with docker a little bit first. 
Tutorials and Readings here: 
  - https://docs.docker.com/get-started/
  - https://docker-curriculum.com/ (read until Docker AWS chapter)

What we want to achieve is to build our application, pack it in a container and run it within the container.
To get our application into a container we first need to tell docker how to build our image for this application. So we create and write a Dockerfile:

```bash

$ touch Dockerfile
```

```docker
FROM adoptopenjdk/openjdk11:alpine

USER 1001

COPY /build/libs/serviceApi-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080:8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

We tell docker we want the openjdk11:alpine image from adoptopenjdk as our baseimage. Then we use a created user 1001 to execute our build with `./gradlew clean build` and copy our builded jar `serviceApi-0.0.1-SNAPSHOT.jar` into the image. We tell docker that we want to expose the internal port 8080 to our hosts system port 8080 and start the container with `java -jar app.jar`. That's all the magic needed to make our little application ready for the cloud. 
Simple build the docker image the following command in the project's base folder: 

```bash

$ docker build .
```

Which will return you a little hash of the image we just created. For example: 

``` 
---> 46b513a9e294
Successfully built 46b513a9e294
```

Afterwards we can start the container with 

```bash

$docker run -p 8080:8080 46b513a9e294
```

and you can hit the api again!

