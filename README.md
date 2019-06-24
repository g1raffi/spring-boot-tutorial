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
import javax.persistence.Id;

@Entity
public class Daemon {

    @Id
    private Long id;
    private String name;
    private int port;
    private String description;

    public Daemon() { }

    public Daemon(Long id, String name, int port, String description) {
        this.id = id;
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

We annotate the daemon class with `@Entity` to enable JPA and Hibernate to work their magic and create mappings and tables for the class. But except for the annotation there is nothing special happening here. 

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
        Daemon daemon = new Daemon(1L, "TestDaemon", 8081, "A test daemon running on port 8081");
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

