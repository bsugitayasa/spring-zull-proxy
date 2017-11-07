# Spring Cloud Zull #

Roadmap

* Overview Spring Cloud Zull 
* Build from scratch
* Live Demo

## Zull Proxy ##

* Common problem microservices -> is provide a unique gateway to the client applications of system
* Service are split into small microservices apps that shouldn't be visible to users -> development / maintenance efforts

To solve : Netflix created and open-sourced its `Zull Proxy server` and Spring has adapted -> `spring cloud stack`

## Zull Proxy Abstraction ##

* Zull diagram-1 : Zull service will intercept all the requests and then route the requests to actual services

![Zull-diagram-1](img/zull-proxy-1.jpg)

* All request will be in REST 

![Zull-diagram-2](img/zull-proxy-2.jpg)

## Build and Run ##

### Spring Boot Initializr ###

1. Browse [https://start.spring.io/] (https://start.spring.io/)

2. Fill Project Metadata

    Group
    
    ```java
    com.sheringsession.balicamp.springzull    
    ```

    Artifact
    
    ```java
    zull-proxy
    ```
   
    Dependencies
    
    ```java
    Eureka Discovery, Eureka Server, Zull
    ```
   
3. Generate Project

4. Download & import to each IDE
    

### Build with Maven ###

1. In `pom.xml` makesure for dependency
    
    ```java
    <dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
	</dependencies>
    
    <dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
    ```

2. `applicatioin.properties` rename into `bootstrap.yml`

    ```java
    spring:
      application:
        name: proxy
      jpa:
        hibernate:
          ddl-auto: validate
      cloud:
        config:
          discovery:
            enabled: true
            service-id: configservice
          fail-fast: true

    #case 1 with path
        catalog:
          path: /ctx/**

    #case 2 with init key catalog directly
    #    catalog: /catalog/**

    #case 3 with initial serviceId
    #     katalog:
    #      serviceId: catalog
    #      path: /catalog/**

    ---
    spring:
      profiles: native
    server:
      port: 8799
    eureka:
      instance:
        hostname: localhost:8799
      client:
          serviceUrl:
            defaultZone: https://localhost:8761/eureka/,https://localhost:8762/eureka/
    ```

### Build with IDE ###

3. Add `@EnableZuulProxy` into your spring project configuration

    ```java
    @SpringBootApplication
    @EnableZuulProxy
    public class ProxyApplication {

        public static void main(String[] args) {
            SpringApplication.run(ProxyApplication.class, args);
        }
    }
    ```

#### Have a nice codding ####