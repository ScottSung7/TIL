```
FROM gradle:8.7-jdk17 AS base
WORKDIR /build
COPY build.gradle settings.gradle /build/
RUN gradle build --parallel --continue > /dev/null 2>&1 || true
COPY . /build

FROM base AS test
RUN gradle test --no-daemon

FROM base AS build
RUN gradle build --no-daemon -x test

FROM openjdk:17.0-slim
WORKDIR /app
COPY --from=build /build/build/libs/checkin-0.0.1-SNAPSHOT.jar .
USER nobody
ENTRYPOINT ["java","-DSpring.profiles.active=docker","-jar","checkin-0.0.1-SNAPSHOT.jar"]


```
