# Section 4: Handle deployment, portability & scalability of microservices using Docker

In this section, the `accounts`, `loans`, and `cards` microservices are updated to support containerization with Docker. A `Dockerfile` is added to each microservice, which defines the steps to build a Docker image for the application. The Dockerfile uses a slim OpenJDK 21 base image, copies the application JAR file, and sets the entrypoint to run the application.
