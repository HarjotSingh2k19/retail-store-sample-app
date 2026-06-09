# Stage 1 — Build
# We need full JDK + Maven to compile the Spring Boot app
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /app

# Copy pom.xml first and download dependencies separately
# This layer is cached — if pom.xml doesn't change, Maven won't re-download
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Now copy source and build the JAR
COPY src ./src
RUN mvn clean package -DskipTests -B

# Stage 2 — Run
# We only need JRE to RUN the JAR — much smaller than JDK
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Copy only the built JAR from Stage 1
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

# Health check — Kubernetes uses this to know if the app is ready
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]