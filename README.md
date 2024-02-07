# Step-by-Step-Guide-Creating-a-Dockerfile-for-Keycloak-from-Scratch

Docker has revolutionized the way we deploy and manage applications, offering a lightweight and consistent environment across different platforms. Keycloak, an open-source identity and access management solution, is a popular choice for securing applications. While an official Keycloak base image is available, creating a Dockerfile for Keycloak from scratch provides additional flexibility and customization. In this article, we will explore the reasons behind building a Dockerfile for Keycloak from scratch and guide you through the process.

Reasons to Create a Dockerfile for Keycloak from Scratch:

Custom Configuration
The official Keycloak base image comes with default configurations. By creating a Dockerfile from scratch, you have the flexibility to tailor the configuration to meet your specific requirements.
Customizing the standalone.xml file allows you to define database connections, logging settings, and other runtime configurations according to your application’s needs.
2. Version Control

Building a Dockerfile from scratch allows you to explicitly control the version of Keycloak being used. This ensures that your application remains consistent and predictable across different environments.
Pinning the Keycloak version in the Dockerfile provides better reproducibility and helps prevent unexpected issues that might arise from automatic updates in the official base image.
3. Reduced Image Size

The official Keycloak base image includes a complete Keycloak distribution, which might contain components that are not necessary for your specific use case. Building from scratch allows you to include only the essential components, resulting in a smaller and more efficient Docker image.

4. Security Auditing

By starting with a base WildFly image and adding Keycloak manually, you have greater visibility into the components that make up your Docker image. This can benefit security audits, allowing you to understand and control the entire software stack.

5. Fine-grained control Over Dependencies

Creating a Dockerfile from scratch gives you control over the installation and configuration of dependencies. You can explicitly install and configure only the libraries and tools your application needs, reducing the attack surface and potential vulnerabilities.

6. Educational Purposes

Building a Dockerfile from scratch provides a valuable learning experience. It allows you to better understand the components that make up a Keycloak deployment, enhancing your knowledge of both Docker and Keycloak.

Dockerfile Setup
Choose a Base Image
The first step is to select a suitable base image. In this example, we’ll use the Eclipse Temurin image with Java 21 for the build stage and an Alpine-based image for the runtime stage.

# Use a base image with Java for the build stage
FROM eclipse-temurin:21-jdk-jammy AS builder
2. Set the Working Directory and Copy the Source Code

Set the working directory inside the Docker image and copy the Keycloak source code into it.

WORKDIR /keycloak

COPY . /keycloak
3. Build the Project with Maven

Build the Keycloak project using Maven. This step compiles the source code and prepares the distribution.

RUN ./mvnw -pl quarkus/deployment,quarkus/dist -am -DskipTests clean install
4. Create the Runtime Image

Switch to a new base image for the runtime stage. Here, we use an Alpine-based image to reduce the image size.

FROM --platform=linux/amd64 eclipse-temurin:21-jre-alpine

RUN addgroup -S keycloak && adduser -S keycloak -G keycloak
5. Copy Keycloak Distribution

Copy the Keycloak distribution from the builder image to the runtime image.

COPY --from=builder /keycloak/quarkus/dist/target/keycloak-*.tar.gz /opt/keycloak.tar.gz
6. Set up Keycloak

Extract and configure Keycloak in the runtime image.

RUN tar -xzf /opt/keycloak.tar.gz -C /opt/ && \
    mv /opt/keycloak-* /opt/keycloak && \
    rm /opt/keycloak.tar.gz && \
    chown -R keycloak:keycloak /opt/keycloak
7. Configure Environment Variables

If needed, set environment variables for Keycloak. Uncomment and modify as required.

ENV KEYCLOAK_USER=admin \
    KEYCLOAK_PASSWORD=admin
8. Set Working Directory and User

Set the working directory and switch to the Keycloak user.

WORKDIR /opt/keycloak

USER keycloak
9. Expose Ports

Expose the ports that Keycloak runs on.

EXPOSE 8080
EXPOSE 8443
10. Set the Startup Command

Define the entry point for the Docker container.

ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
The complete Dockerfile would be like below.

# Use a base image with Java
FROM eclipse-temurin:21-jdk-jammy AS builder

# Set the working directory in the Docker image
WORKDIR /keycloak

# Copy the Keycloak source code into the image
COPY . /keycloak

# Build the project using Maven
RUN ./mvnw -pl quarkus/deployment,quarkus/dist -am -DskipTests clean install

FROM --platform=linux/amd64 eclipse-temurin:21-jre-alpine

RUN addgroup -S keycloak && adduser -S keycloak -G keycloak

# Copy the Keycloak distribution from the builder image
COPY --from=builder /keycloak/quarkus/dist/target/keycloak-*.tar.gz /opt/keycloak.tar.gz

# Set up Keycloak
RUN tar -xzf /opt/keycloak.tar.gz -C /opt/ && \
    mv /opt/keycloak-* /opt/keycloak && \
    rm /opt/keycloak.tar.gz && \
    chown -R keycloak:keycloak /opt/keycloak

# Set the working directory
WORKDIR /opt/keycloak

# Switch to the Keycloak user
USER keycloak

# Expose the port Keycloak runs on
EXPOSE 8080
EXPOSE 8443

# Set environment variables if needed
# ENV KEYCLOAK_USER=admin \
#     KEYCLOAK_PASSWORD=admin

# Set the startup command
ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
This Dockerfile is now ready to be used to build a Docker image for Keycloak. It encapsulates the necessary steps to build and run Keycloak in a containerized environment, making it easier to deploy and manage.
