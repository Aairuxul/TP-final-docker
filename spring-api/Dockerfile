FROM maven:3.9-eclipse-temurin-25-alpine AS dev

WORKDIR /app

COPY pom.xml .

RUN mvn -B -q dependency:go-offline

COPY src ./src

EXPOSE 8080

CMD ["mvn", "spring-boot:run", "-DskipTests"]
