# DockerNotes

## How to install root certificate into a Docker container 

This will be the Steps to Do so

Firstly, You need to obtain the root certificate: You will need to obtain the root certificate that you want to install on the Docker container. This can usually be obtained from the certificate authority that issued the certificate.

Secondly, Create a Dockerfile: Create a Dockerfile that includes the root certificate. You can use the ADD command to add the certificate file to the Docker container. For example, your Dockerfile might look like this:


    FROM your-image

    # Add the root certificate
    ADD root.crt /usr/local/share/ca-certificates/

    # Update the CA certificates
    RUN update-ca-certificatess
In this example, root.crt is the name of the root certificate file.    

Build the Docker image: Build the Docker image using the Dockerfile. You can use the docker build command to build the image. For example:
    
    docker build -t your-image 

Run the Docker container: Run the Docker container using the image you just built. You can use the docker run command to run the container. For example:
    
    docker run -it your-image

The update-ca-certificates command in the Dockerfile will update the CA certificates on the container with the root certificate you added. This will allow the container to trust SSL connections that use the root certificate.
 
Once you have installed the root certificate on the Docker container, you should be able to use SSL connections that rely on the root certificate without encountering certificate validation errors.


### Dockerfile to build a Spring Boot application transactional-portal image and inject a root certificate, you can follow these steps:

Start with a base image: Choose a base image that will form the foundation for your Dockerfile. For example, you might choose an OpenJDK or Oracle JDK image.

Set working directory: Set the working directory where the application will be built and run. For example:
    
    WORKDIR /app

Copy the application code: Copy the application code into the Docker image. For example, if your application code is in a directory named transactional-portal on your local machine, you would use the following command to copy it to your Docker image:

    COPY transactional-portal /app

Build the application: Use the RUN command to build the application inside the Docker image. For example, if you're using Maven as your build tool, you might use the following command to build the application:
    
    RUN mvn clean package

Create an environment variable for the certificate file path: Set an environment variable to the path of the root certificate file that will be injected into the image. For example:
    
    ENV ROOT_CERT_FILE=/app/root.crt

Copy the root certificate: Copy the root certificate into the Docker image. For example, if your root certificate file is named root.crt and is located in the same directory as your Dockerfile, you would use the following command to copy it to your Docker image: 
    
    COPY root.crt /app

Install the root certificate: Use the RUN command to install the root certificate into the Java truststore inside the Docker image. For example, you might use the following command to install the root certificate:
    
    RUN keytool -import -file $ROOT_CERT_FILE -alias root -storetype JKS -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit -noprompt
This command installs the root certificate into the Java truststore located at $JAVA_HOME/lib/security/cacerts. The default password for the truststore is changeit

Set the entry point: Set the entry point for the Docker image to run the Spring Boot application. For example:
    ENTRYPOINT ["java", "-jar", "/app/target/transactional-portal.jar"]

This command specifies that the container will run the transactional-portal.jar file located in the /app/target directory.

Your final Dockerfile might look something like this:

    FROM openjdk:11

    WORKDIR /app

    COPY transactional-portal /app

    RUN mvn clean package

    ENV ROOT_CERT_FILE=/app/root.crt

    COPY root.crt /app

    RUN keytool -import -file $ROOT_CERT_FILE -alias root -storetype JKS -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit -noprompt

    ENTRYPOINT ["java", "-jar", "/app/target/transactional-portal.jar"]

 This Dockerfile imports an OpenJDK 11 image as the base image, sets the working directory to /app, copies the transactional-portal application code to the Docker image, builds the application using Maven, sets an environment variable for the root certificate file path, copies the root certificate to the Docker image, installs the root certificate into the Java truststore, and sets the entry point to run the Spring Boot application.
 

## Why we need a Root certificate for java applications 

A root certificate is a digital certificate that is used to establish the trustworthiness of other certificates. It is issued by a trusted certificate authority (CA) and is used to sign the certificates of other entities, such as servers, applications, or users.

A keystore and a truststore are used to manage digital certificates in Java applications. A keystore is a file that contains private keys and the associated digital certificates. It is used to authenticate a Java application to other entities. A truststore, on the other hand, is a file that contains trusted digital certificates, including root certificates and other certificates that are used to establish trust with other entities.

The relationship between a root certificate, keystore, and truststore is as follows:

A root certificate is used to sign other certificates, including the certificates stored in a keystore and truststore.

A keystore contains private keys and digital certificates, including those that are signed by a root certificate. The private key is used to authenticate the Java application to other entities.

A truststore contains digital certificates, including root certificates, that are used to establish trust with other entities. When a Java application makes an SSL/TLS connection to another entity, the entity presents its digital certificate. The Java application checks the digital certificate against the certificates in the truststore. If the certificate is trusted, the SSL/TLS connection is established. If the certificate is not trusted, the SSL/TLS connection fails.

Therefore, to establish trust between a Java application and other entities, it is necessary to have a truststore that contains the trusted digital certificates, including root certificates, of the entities with which the Java application will communicate. Additionally, if the Java application needs to authenticate itself to other entities, it will need to have a keystore that contains the digital certificate and private key used for authentication.


# Example to illustrate the relationship between a root certificate, keystore, and truststore:

Obtain a root certificate: Assume that we have obtained a root certificate from a trusted certificate authority (CA).

Create a keystore: We create a keystore that contains a digital certificate that is signed by the root certificate. This keystore will be used by our Java application to authenticate itself to other entities.

We can use the keytool utility to create the keystore and generate a digital certificate. Here's an example command to create a keystore with a self-signed certificate:

    keytool -genkeypair -alias myapp -keyalg RSA -keysize 2048 -validity 365 -keystore myapp.keystore
    
This command generates a keystore named myapp.keystore with an alias of myapp. It uses the RSA algorithm with a key size of 2048 bits and a validity of 365 days.   

Create a truststore: We create a truststore that contains the root certificate and any other certificates that are required to establish trust with other entities. This truststore will be used by our Java application to verify the digital certificates presented by other entities.


We can use the keytool utility to create the truststore and import the root certificate. Here's an example command to import the root certificate into a truststore named mytruststore:
    keytool -import -alias myca -file myca.crt -keystore mytruststore
This command imports the root certificate from a file named myca.crt into a truststore named mytruststore. The alias for the certificate is set to myca.

 Use the keystore and truststore in the Java application: In our Java application, we can specify the keystore and truststore using system properties. For example:
 
        System.setProperty("javax.net.ssl.keyStore", "/path/to/myapp.keystore");
        System.setProperty("javax.net.ssl.keyStorePassword", "mypass");
        System.setProperty("javax.net.ssl.trustStore", "/path/to/mytruststore");
        System.setProperty("javax.net.ssl.trustStorePassword", "mypass");
      
This code sets the system properties javax.net.ssl.keyStore and javax.net.ssl.keyStorePassword to the location and password of the keystore, and javax.net.ssl.trustStore and javax.net.ssl.trustStorePassword to the location and password of the truststore.


### A socket connection reset can occur due to several reasons such as network issues, application bugs, server overload, firewall restrictions, and many others.

To fix a socket connection reset, you can try the following steps:

 Check the network connectivity: Verify that both the client and server have a stable network connection. You can use tools like ping or traceroute to diagnose network problems.

 Check firewall settings: Make sure that the firewall is not blocking the socket connection. Adjust the firewall settings to allow the connection.

 Increase timeout settings: Increase the timeout settings on the client and server to give more time for the connection to establish.

 Check server resources: Ensure that the server has enough resources to handle the connection requests. If the server is overloaded, it may reset the connection.

 Check the code: Review the code that establishes the socket connection and ensure that it is free of bugs. You can also add error handling and retry logic to handle socket connection failures gracefully.

 Update software: Check for updates to the operating system, network drivers, and application software. Outdated software may cause socket connection issues


  
