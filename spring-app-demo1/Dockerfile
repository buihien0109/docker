#Stage 1
FROM maven:ibmjava-alpine as builder

COPY . /app

WORKDIR /app

RUN mvn clean package

#Stage 2
FROM openjdk:8-alpine

WORKDIR /java

COPY --from=builder /app/target/websocket-demo-0.0.1-SNAPSHOT.jar /java

CMD ["java", "-Djava.security.egd=file:/dev/./urandom ", "-jar", "websocket-demo-0.0.1-SNAPSHOT.jar"]