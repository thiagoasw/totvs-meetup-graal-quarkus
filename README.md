# TOTVS Meetup: GraalVM & Quarkus

Simple demo showed in the TOTVS Meetup about GraalVM and Quarkus.

## Create the base web project

```cmd
mvn io.quarkus:quarkus-maven-plugin:0.20.0:create -DprojectGroupId="com.tasw.meetup" -DprojectArtifactId="graalvm-quarkus" -DclassName="com.tasw.meetup.graalvm.quarkus.GreetingResource" -Dpath="/hello"
```

## Build the native image

```cmd
cd .\graalvm-quarkus\
```

For Windows OS generate a `Dockerfile.multistage` file at `src/main/docker/` with this content:

```cmd
cd . > ./src/main/docker/Dockerfile.multistage
```

```yaml
## Stage 1 : build with maven builder image with native capabilities
FROM quay.io/quarkus/centos-quarkus-maven:19.1.1 AS build
COPY src /usr/src/app/src
COPY pom.xml /usr/src/app
USER root
RUN chown -R quarkus /usr/src/app
USER quarkus
RUN mvn -f /usr/src/app/pom.xml -Pnative clean package

## Stage 2 : create the docker final image
FROM registry.access.redhat.com/ubi8/ubi-minimal
WORKDIR /work/
COPY --from=build /usr/src/app/target/*-runner /work/application
RUN chmod 775 /work
EXPOSE 8080
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

Remove the .dockerignore.

```cmd
rm .\.dockerignore
```

Build the native image.

```cmd
docker build -f src/main/docker/Dockerfile.multistage -t totvs-meetup/graal-quarkus .
```

## Inspect the generated image

```cmd
docker image ls totvs-meetup/graal-quarkus
```

## Run the native image

```cmd
docker run -i --rm -p 8080:8080 totvs-meetup/graal-quarkus
```

Run with less memory...

```cmd
docker run -i --rm --memory=20m -p 8080:8080 totvs-meetup/graal-quarkus
```

## Watch the container stats

```cmd
docker stats $(docker ps -q)
```