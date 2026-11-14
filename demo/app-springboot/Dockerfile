# ── Stage 1 : Build ──────────────────────────────────────────────────────────
# Images Ubuntu multi-arch (amd64 + arm64) : les variantes -alpine n'existent qu'en amd64
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /build

# Téléchargement des dépendances séparé pour profiter du cache Docker
COPY pom.xml .
RUN mvn dependency:go-offline -B -q

COPY src ./src
RUN mvn clean package -DskipTests -B -q

# ── Stage 2 : Runtime minimal ─────────────────────────────────────────────────
FROM eclipse-temurin:17-jre

WORKDIR /app

# Utilisateur non-root (principe du moindre privilège)
RUN groupadd --system spring && useradd --system --gid spring --no-create-home --shell /usr/sbin/nologin spring
USER spring:spring

COPY --from=builder /build/target/*.jar app.jar

EXPOSE 8090

ENTRYPOINT ["java", \
            "-XX:+UseContainerSupport", \
            "-XX:MaxRAMPercentage=75.0", \
            "-Djava.security.egd=file:/dev/./urandom", \
            "-jar", "app.jar"]