# ---------- Étape 1 : build Maven ----------
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
# Dépendances d'abord (cache Docker tant que pom.xml ne change pas)
COPY pom.xml .
RUN mvn -B -q dependency:go-offline
COPY src ./src
RUN mvn -B -q package -DskipTests

# ---------- Étape 2 : image d'exécution légère ----------
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
RUN addgroup -S spring && adduser -S spring -G spring
COPY --from=build /app/target/*.jar app.jar
USER spring
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
