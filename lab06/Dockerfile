FROM maven:3-eclipse-temurin-17 AS build
RUN mkdir /usr/src/project
COPY . /usr/src/project
WORKDIR /usr/src/project
RUN mvn package -DskipTests
RUN jar xf target/testing-web-complete-0.0.1-SNAPSHOT.jar
RUN jdeps --ignore-missing-deps -q  \
    --recursive  \
    --multi-release 17  \
    --print-module-deps  \
    --class-path 'BOOT-INF/lib/*'  \
    target/testing-web-complete-0.0.1-SNAPSHOT.jar > deps.info
RUN jlink \
    --add-modules $(cat deps.info) \
    --strip-debug \
    --compress 2 \
    --no-header-files \
    --no-man-pages \
    --output /myjre

FROM debian:12.10-slim
ENV JAVA_HOME /user/java
ENV PATH $JAVA_HOME/bin:$PATH
COPY --from=build /myjre $JAVA_HOME
RUN mkdir /project
COPY --from=build /usr/src/project/target/testing-web-complete-0.0.1-SNAPSHOT.jar /project/
WORKDIR /project
EXPOSE 8080
ENTRYPOINT java -jar testing-web-complete-0.0.1-SNAPSHOT.jar