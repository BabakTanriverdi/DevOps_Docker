FROM eclipse-temurin:11-jdk-jammy AS builder
WORKDIR /app
COPY . .
RUN javac App.java

FROM eclipse-temurin:11-jre-jammy
WORKDIR /myapp
COPY --from=builder /app .
CMD ["java", "App"]