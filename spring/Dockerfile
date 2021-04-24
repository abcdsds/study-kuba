FROM gradle:6.8.3-jdk11
WORKDIR /home/gradle/src
COPY build.gradle /home/gradle/src
RUN gradle clean build --no-daemon > /dev/null 2>&1 || true
ENV GRADLE_USER_HOME /home/gradle/src
COPY . /home/gradle/src
RUN gradle clean build --no-daemon

FROM adoptopenjdk/openjdk11:alpine as builder
WORKDIR application
COPY --from=0 /home/gradle/src/build/libs/*.jar application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM adoptopenjdk/openjdk11:alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
WORKDIR app
EXPOSE 8080
COPY ./wait-for.sh ./
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]