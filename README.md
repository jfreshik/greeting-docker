## SpringBoot docker build with gradle

1. build.gradle setup
```
buildscript {
    ...
    dependencies {
        ... 
        classpath('gradle.plugin.com.palantir.gradle.docker:gradle-docker:0.13.0')
    }
}
...
plugins {
    id 'com.palantir.docker' version '0.20.1'
}
...
ext {
    dockerUser = 'jfreshik'
}

task unpack(type: Copy) {
    dependsOn bootJar
    from(zipTree(tasks.bootJar.outputs.files.singleFile))
    into("build/dependency")
}

docker {
    // set docker image name
    name "${project.dockerUser}/${bootJar.baseName}"
    copySpec.from(tasks.unpack.outputs).into("dependency")
    buildArgs(['DEPENDENCY': "dependency"])
}

```

2. build docker image
 * command line

> $ ./gradlew docker



 * IntelliJ IDE

> intellij Gradle Tool Window -> Tasks -> docker -> docker 

3. push docker image


## Dockerfile
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
EXPOSE 8080
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","hello.GreetingApplication"]
```

## References
spring-boot with docker:
https://spring.io/guides/gs/spring-boot-docker/

gradle-docker: https://github.com/palantir/gradle-docker

maven-docker: https://github.com/spotify/dockerfile-maven

JAR format: https://ko.wikipedia.org/wiki/JAR_(%ED%8C%8C%EC%9D%BC_%ED%8F%AC%EB%A7%B7)
