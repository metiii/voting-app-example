FROM maven:3.6-jdk-8-slim

WORKDIR /app

COPY . .

RUN mvn package -DskipTest && \
mv target/worker-jar-with-dependencies.jar /run/worker.jar && \
rm -rf /app/*


CMD java -jar /run/worker.jar

