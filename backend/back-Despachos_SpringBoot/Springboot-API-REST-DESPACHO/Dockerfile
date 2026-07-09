# ETAPA DE CONSTRUCCIÓN
FROM eclipse-temurin:25-jdk AS build
WORKDIR /app

# Copiar wrappers y configuración
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .

RUN tr -d '\r' < gradlew > gradlew.tmp && mv gradlew.tmp gradlew
RUN chmod +x gradlew

# Copiar el código fuente
COPY src src

# COMPILACIÓN:
RUN ./gradlew clean build -x test \
    -Porg.gradle.java.installations.auto-download=false \
    -Dorg.gradle.java.home=$JAVA_HOME

# ETAPA DE EJECUCIÓN
FROM eclipse-temurin:25-jre
WORKDIR /app

COPY --from=build /app/build/libs/*.jar app.jar

EXPOSE 8081

ENV SERVER_PORT=8081

ENTRYPOINT ["java", "-jar", "app.jar"]