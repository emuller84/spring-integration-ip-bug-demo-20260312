# Description

This repository provides the means to attempt to replicate the following error in the `org.springframework.integration:spring-integration-ip:6.5.7` java library:

```text
java.lang.NullPointerException: Cannot invoke "java.util.concurrent.CountDownLatch.await(long, java.util.concurrent.TimeUnit)" because "this.writingLatch" is null
	at org.springframework.integration.ip.tcp.connection.TcpNioConnection.convert(TcpNioConnection.java:380)
	at org.springframework.integration.ip.tcp.connection.TcpNioConnection.run(TcpNioConnection.java:259)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
	at java.base/java.lang.Thread.run(Thread.java:1583)
```

The error happens inside the `org.springframework.integration.ip.tcp.connection.TcpNioConnection` class. It's being used to connect to a TCP server.

# Contents of this repository

- Spring boot application: the root of this repository contains a Java, Gradle-based, project. It's an HTTP JSON API project using spring boot and sprint integration that mimics the TCP connectivity used by the real-world application were the error being investigated actually happened.
- [`device_simulator.py`](device_simulator.py): a python 3 script the creates the TCP server that the spring application will connect to.
- [`spring-integration-ip-bug-demo-20260312.postman_collection.json`](spring-integration-ip-bug-demo-20260312.postman_collection.json): a Postman collection with a sample request to hit the spring application
  - You can use Postman's "performance run" on this collection to hit the application with high TPS and, hopefully, trigger the error.

# API request

A request to the API application looks like this:

```text
POST http://localhost:8014/v0/api/endpointA
```

```json
{
  "value": "this is a test"
}
```

And a successful response:

```json
{
  "deviceResponseValue": "DEVICE RESPONSE - ECHO: this is a test",
  "deviceError": null,
  "otherError": null
}
```

# Steps to reproduce the error

1. Launch the TCP server: `python3 device_simulator.py`
2. Launch the spring application
   - If possible, run it with Amazon Corretto distribution of Java 21. This is what I used.
   - I ran it from the IntelliJ IDEA IDE
   - Running it in debug mode may help increase the possibility of triggering the error
   - This spring app uses two TCP connections (two `TcpOutboundGateway`s plus connection factories) to replicate the connection setup used in real-world application where I experienced this potential bug. I have also tried with a single connection and was able to see the error, but it took more time and luck to trigger it. If you want to try it with a just one connection, which I guess would be easier to debug, then set the configuration property `device-service.enable-second-connection` to `false` in [src/main/resources/application.yaml](src/main/resources/application.yaml).
3. On the Postman application:
   - Import the [`spring-integration-ip-bug-demo-20260312.postman_collection.json`](spring-integration-ip-bug-demo-20260312.postman_collection.json) collection
   - Go to the "Overview" page of the imported collection -> "Runs" -> "Performance" -> "Run" -> "Performance" and run
     - A "fixed" load profile and "20" virtual users should work
     - It may be necessary to let it run for some minutes to finally see the error

**IMPORTANT!:** again, a reminder: it's not guaranteed you will experience the error, but it should be possible with time and luck.

**NOTE:** something that may help: I have more luck triggering the error by running this project on a linux machine than on a Windows one.

