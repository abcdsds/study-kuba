FROM gradle:latest
WORKDIR /home/gradle/src
COPY build.gradle /home/gradle/src
RUN gradle clean build --no-daemon > /dev/null 2>&1 || true
ENV GRADLE_USER_HOME /home/gradle/src
COPY . /home/gradle/src
RUN gradle build --no-daemon 

FROM openjdk:8-jdk-alpine as builder
WORKDIR app
COPY --from=0 /home/gradle/src/build/libs/*.jar spring-boot-application.jar
RUN java -Djarmode=layertools -jar spring-boot-application.jar extract

FROM openjdk:8-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
WORKDIR app
USER spring:spring
WORKDIR app
EXPOSE 8080

COPY --from=builder app/dependencies/ ./
COPY --from=builder app/spring-boot-loader/ ./
COPY --from=builder app/snapshot-dependencies/ ./
COPY --from=builder app/app/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]