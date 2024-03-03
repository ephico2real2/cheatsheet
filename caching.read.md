```

In practice, this means that you need to look at the documentation of your build system and test framework and make sure that all folders you want cached are placed under the Codefresh volume.

This is a typical pattern with Java applications.

For Maven use mvn -Dmaven.repo.local=/codefresh/volume/m2_repository package as shown in the example.
For Gradle use gradle -g /codefresh/volume/.gradle -Dmaven.repo.local=/codefresh/volume/m2 as explained in the example.
For SBT use -Dsbt.ivy.home=/codefresh/volume/ivy_cache.
For Pip use pip install -r requirements.txt --cache-dir=/codefresh/volume/pip-cache as shown in the example
For Golang pass an environment variable GOPATH=/codefresh/volume/go to the freestyle step that is running go commands
For Rust pass an environment variable CARGO_HOME=/codefresh/volume/cargo to the freestyle step that is running rust/cargo commands

```

```
The example Java project
You can see the example project at https://github.com/codefresh-contrib/spring-boot-2-sample-app. The repository contains a Spring Boot 2 project built with Maven with the following goals:

mvn package creates a jar file that can be run on its own (exposes port 8080). It also runs unit tests.
mvn verify runs integration tests as well. The application is launched locally as part of the Maven lifecycle.
Once launched the application presents a simple message at localhost:8080 and also at the various /actuator/health endpoints. You can use the standard spring-boot:run command to run it locally (without Docker).

Spring Boot 2 and Docker (package only)
A Dockerfile is also provided at the same repository. It uses the base JRE image and just copies the JAR file inside the container.

Dockerfile.only-package

```

```
FROM java:8-jre-alpine

EXPOSE 8080

RUN mkdir /app
COPY target/*.jar /app/spring-boot-application.jar

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app/spring-boot-application.jar"]

HEALTHCHECK --interval=1m --timeout=3s CMD wget -q -T 3 -s http://localhost:8080/actuator/health/ || exit 1

```

```
This means that before building the Docker image, the compilation step (mvn package) is expected to be finished already. Therefore, in the codefresh.yml file we need at least two steps. The first one should prepare the JAR file and the second one should create the Docker image.

Create a CI pipeline for Spring
The repository also contains a premade Codefresh YAML file that you can use as a starting point in your own Spring Boot 2 projects.

Here are the full contents of the file.

codefresh-package-only.yml

```

```
version: '1.0'
stages:
  - prepare
  - test
  - build
  - 'integration test'
steps:
  main_clone:
    title: Cloning main repository...
    stage: prepare
    type: git-clone
    repo: 'codefresh-contrib/spring-boot-2-sample-app'
    revision: master
    git: github
  run_unit_tests:
    title: Compile/Unit test
    stage: test
    image: 'maven:3.5.2-jdk-8-alpine'
    commands:
      - mvn -Dmaven.repo.local=/codefresh/volume/m2_repository package      
  build_app_image:
    title: Building Docker Image
    type: build
    stage: build
    image_name: spring-boot-2-sample-app
    working_directory: ./
    tag: 'non-multi-stage'
    dockerfile: Dockerfile.only-package
  run_integration_tests:
    title: Integration test
    stage: 'integration test'
    image: maven:3.5.2-jdk-8-alpine
    commands:
     - mvn -Dmaven.repo.local=/codefresh/volume/m2_repository verify -Dserver.host=http://my-spring-app
    services:
      composition:
        my-spring-app:
          image: '${{build_app_image}}'
          ports:
            - 8080
      readiness:
        timeoutSeconds: 30
        periodSeconds: 15
        image: byrnedo/alpine-curl
        commands:
          - "curl http://my-spring-app:8080/"

```
