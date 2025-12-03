FROM eclipse-temurin:11-jdk-jammy

WORKDIR /app

# Install Maven
RUN apt-get update && apt-get install -y maven && rm -rf /var/lib/apt/lists/*

# Copy source code
COPY . .

# Build the application
RUN mvn clean package -DskipTests

EXPOSE 9005

ENTRYPOINT ["java", "-jar", "target/profile-service-1.0.0.jar"]
