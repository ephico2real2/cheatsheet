# Pipeline ideas

[] Reference: https://codefresh.io/docs/docs/example-catalog/ci-examples/spring-boot-2/


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

```bash
The example Java project
You can see the example project at https://github.com/codefresh-contrib/spring-boot-2-sample-app. The repository contains a Spring Boot 2 project built with Maven with the following goals:

mvn package creates a jar file that can be run on its own (exposes port 8080). It also runs unit tests.
mvn verify runs integration tests as well. The application is launched locally as part of the Maven lifecycle.
Once launched the application presents a simple message at localhost:8080 and also at the various /actuator/health endpoints. You can use the standard spring-boot:run command to run it locally (without Docker).

Spring Boot 2 and Docker (package only)
A Dockerfile is also provided at the same repository. It uses the base JRE image and just copies the JAR file inside the container.

Dockerfile.only-package

```

```bash
FROM java:8-jre-alpine

EXPOSE 8080

RUN mkdir /app
COPY target/*.jar /app/spring-boot-application.jar

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app/spring-boot-application.jar"]

HEALTHCHECK --interval=1m --timeout=3s CMD wget -q -T 3 -s http://localhost:8080/actuator/health/ || exit 1

```

```bash
This means that before building the Docker image, the compilation step (mvn package) is expected to be finished already. Therefore, in the codefresh.yml file we need at least two steps. The first one should prepare the JAR file and the second one should create the Docker image.

Create a CI pipeline for Spring
The repository also contains a premade Codefresh YAML file that you can use as a starting point in your own Spring Boot 2 projects.

Here are the full contents of the file.

codefresh-package-only.yml

```

```yaml
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

```bash
Notice that because the Maven lifecycle also executes the previous steps in a build, the mvn verify command essentially will run mvn package as well. In theory we could just have the Integration step in this pipeline on its own. That step would build the code, run unit and integration tests all in one stage. For demonstration purposes however, we include two steps so that you can see the correct usage of Maven cache.
```


```bash
Spring Boot 2 and Docker (multi-stage builds)
Docker added multi-stage builds at version 17.05. With multi-stage builds a Docker build can use one base image for compilation/packaging/unit tests and a different one that will hold the runtime of the application. This makes the final image more secure and smaller in size (as it does not contain any development/debugging tools).

In the case of Java, multistage builds allow for the compilation itself to happen during the build process, even though the final Docker image will not contain a full JDK.

Here is the multi-stage build definition:

Dockerfile
```

```yaml
FROM maven:3.5.2-jdk-8-alpine AS MAVEN_TOOL_CHAIN
COPY pom.xml /tmp/
RUN mvn -B dependency:go-offline -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml
COPY src /tmp/src/
WORKDIR /tmp/
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package

FROM java:8-jre-alpine

EXPOSE 8080

RUN mkdir /app
COPY --from=MAVEN_TOOL_CHAIN /tmp/target/*.jar /app/spring-boot-application.jar

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app/spring-boot-application.jar"]

```

```bash
This docker build does the following:

Starts from the standard Maven Docker image
Copies only the pom.xml file inside the container
Runs a mvn command to download all dependencies found in the pom.xml
Copies the rest of the source code in the container
Compiles the code and runs unit tests (with mvn package)
Discards the Maven image with all the compiled classes/unit test results etc
Starts again from the JRE image and copies only the JAR file created before
The order of the steps is tuned so that it takes advantage of the layer caching built-in to Docker. If you change something in the source code Docker already has a layer with Maven dependencies so they will not be re-downloaded again. Only if you change the pom.xml file itself, Docker will start again from the lowest layer.

Again, we define a custom location for the Maven cache (using the settings-docker.xml file). This way the Maven dependencies are placed inside the container and they will be cached automatically with the respective layer (Read more about this technique at the official documentation.

```

```bash
Create a CI pipeline for Spring (multi-stage Docker builds)
Because in multi-stage builds Docker itself handles most of the build process, moving the project to Codefresh is straightforward. We just need a single step that creates the Docker image after checking out the code. The integration test step is the same as before.


```

```yaml
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
  build_app_image:
    title: Building Docker Image
    type: build
    stage: build
    image_name: spring-boot-2-sample-app
    working_directory: ./
    tag: 'multi-stage'
    dockerfile: Dockerfile
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

