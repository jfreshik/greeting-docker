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

## Run docker container

도커 이미지 실행(docker run jfreshik/greeting-docker)
로컬 포트와 Dockerfile 에 EXPOSE 로 설정한 컨테이너포드 연결 (-p 8080:8080) 
> $ docker run -p 8080:8080 jfreshik/greeting-docker


도커 실행 시 환경변수 전달 (-e) => spring profile 적용
> $ docker run -e "SPRING_PROFILES_ACTIVE=dev" -p 8080:8080 jfreshik/greeting-docker 

## docker build and publish to dockerhub

DockerHub 계정정보 암호화. --add 옵션으로 travis.yml 에 암호토큰(secure: xxxx..) 을 저장 
> $ travis encrypt DOCKER_PASS="dockerpass" --add

암호화 된 정보를 travis.yml 의 env.global 에 복사


travis.yml
```
...

sudo: required
services:
  - docker

env:
  global:
    - DOCKER_USER=jfreshik
    - secure: AtIzjr23nOvBKAjJ+iCyui6MsLeX+fvygq3usiG/Bq746D7ki+wl0/laNrIJ/iVCVy51R+Uxg4cWQZDxZryUqioxjFxQQerMxZ4xDb+V1fh6PKN3cAf1pDW5rWCGBBkX/iuS5kY9WwLvzlJi7+gxG948hfLc9frmiybeaYoNSu6qQ1DkaKGsyMwNqLPnZFCXN6dsy9yM66BEJs6e7PFtJk4EvByMAWpMb2ME8VFaq738MX6aQrTPOZYqpJvtMtXUm2UROYu/RFlMTIUa1XvIFvcEWpmbBJi543HiSUd5unmu9sntVYVc20LGchgZBg08FL3ie2xQm02amqAUSGKmptd3dXiEUngHmeEI+oXdPeg6GSEQhkKnb6n1owD/uN1HGrPQ9AGiQm4Qcp/jUgxIIQJJ/3Y+GhjFAnd1WMWmHYiUf3ai8ZwLHmqwwGimIyxrDr4aj7b7nvAJfWh2Pipcr+04hR18J/tL2vkfHIS0rsJ3Upjz9Zy8qZunN4lH5xgVVIsdY2rAHsMvL9C43XsA3BZ9cciF4z3ebD0TzhzawMbUbPyfCApf9xQknPmQ/gI17hUCFeBK8Vtwp9i3vXM0tcNTUfDgLVsPqGmsGyzT8AbAlcZuQ42mD0doXaDx6vFDDVAYPCpjZNP7jdkfqxGi6AGhpdEM1TSdh5jvYXrKuho=
    - COMMIT=${TRAVIS_COMMIT::8}

after_success:
  - echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
  - export IMAGE_NAME=jfreshik/greeting-docker
  - echo $IMAGE_NAME
  - "./gradlew docker"
  - docker tag $IMAGE_NAME $IMAGE_NAME:latest
  - docker push $IMAGE_NAME:latest

...
```

## travis 암호화 적용 문제

travis cli 를 이용해 암호토큰을 생성할 때 아래와 같은 오류가 나타나면

```
$ travis encrypt DOCKER_PASS="mydockerpass"
repository not known to https://api.travis-ci.org/: jfreshik/greeting-docker
```

1. travis 에 로그인
> $ travis login --pro --auto

2. travis-ci.com 과 travis-ci.org 구분
> $ travis encrypt --com SOMEVAR="secretvalue"

## References
spring-boot with docker:
https://spring.io/guides/gs/spring-boot-docker/

gradle-docker: https://github.com/palantir/gradle-docker

maven-docker: https://github.com/spotify/dockerfile-maven

JAR format: https://ko.wikipedia.org/wiki/JAR_(%ED%8C%8C%EC%9D%BC_%ED%8F%AC%EB%A7%B7)

travis encryption: https://docs.travis-ci.com/user/encryption-keys/