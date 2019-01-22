
# Zowe scenario 1: Expose a Spring Boot application as a RESTful API

## Overview

You can expose an established z/OS application as a RESTful API so that it can be exploited by different applications and organizations.

As the example product we are using a simple Spring Boot sample app that can be downloaded here: spring-boot-jzos-sample. This sample uses JZOS to provision a Rest API for accessing Java environmental variables.

This scenario guides you through the steps in roughly 30 minutes. By the end of the session, you’ll know how to:

- Create a Rest API application from scratch

You should be familiar with the following technologies:
- Java
- Git and GitHub
- Maven

Knowledge of the following development technologies is beneficial:
- Spring Boot
- Eclipse/z/OS Explorer
- Rest API
- zSystems development

Please wait a moment while your development environment loads (this takes a minute or so). When it loads, get started by creating the API project.

## Task: 

### Prerequisite
- Build requires Maven installed on local machine

### Steps

- Download the sample to your local machine.
- Download IBMJZOS from the host machine running Zowe. Create a lib folder off the project root and create the ibmjzos-1.0.jar file for compilation purposes
- Build the sample and transfer the file /jzos/target/jzos-0.0.1-SNAPSHOT.jar to a USS folder on the Zowe server.
- Transfer the jzos.sh file from the project root folder to the same USS folder on the Zowe server.
- Modify the shell script as necessary for the environment

### Keystore

As this sample runs using HTTPS you need to reference a key store or create your own. Although creating your own is relatively straight forward I have reused the one used by the API mediation layer. It's worth noting that at the time of writing this Readme I have used a relative reference to the keystore and you should update based upon the location of the Zowe keystore used.

### IBMJZOS.jar

The app has a dependency on IBMJZOS so for compiling you need to download from your z/OS installation. Simply copy the file from your Java installation. $JAVA_HOME/lib/ext/ibmjzos.jar call it ibmjzos-1.0.jar and save in the /libs folder of this project.

### Shell Script

The shell script builds the required to specify the classpath and set environmental properties. Please check the valiidity on your system of:

- Location of JAVA_HOME
- key_store location and if an alternative keystore is used the type and passwords.
-  An available port number


# Zowe scenario 2: Onboard the RESTful API in the Zowe API Mediation Layer

## Overview

The API Mediation Layer provides a single point of access for mainframe service REST APIs. The layer offers enterprise, cloud-like features such as high-availability, scalability, dynamic API discovery, consistent security, a single sign-on experience, and documentation. The API Mediation Layer facilitates secure communication across loosely coupled microservices through the API Gateway. The API Mediation Layer includes an API Catalog that provides an interface to view all discovered microservices, their associated APIs, and Swagger documentation in a user-friendly manner. The Discovery Service makes it possible to determine the location and status of microservice instances running inside the ecosystem.

This scenario guides you through the steps in roughly 30 minutes. By the end of the session, you will know how to:
- Add Zowe API enabler annotations to your service code and update the build scripts. 
- Update your service configuration file to include MFaaS API Layer specific settings.
- Setup key store with the service certificate.
- Externalize the API Layer site-specific configuration settings.
- Test that your API instance is working and is discoverable.

No previous knowledge of API Mediation Layer is needed, but some awareness of API terminology might help.

Please wait a moment while your development environment loads (it takes a minute or so).

## Add Zowe API enablers to your service
The first step to onboard a REST API with the Zowe ecosystem is to add enabler annotations to your service code. Enablers prepare your service for discovery and swagger documentation retrieval.

1. Add the following annotations to the main class of your Spring Boot, or add these annotations to an extra Spring configuration class:

    *  `@ComponentScan({"com.ca.mfaas.enable", "com.ca.mfaas.product"})`
    *  `@EnableApiDiscovery`

    **Example:**   

    ```
     package com.ca.mfaas.DiscoverableClientSampleApplication;
     ..
     import com.ca.mfaas.enable.EnableApiDiscovery;
     import org.springframework.context.annotation.ComponentScan;
     ..
     @EnableApiDiscovery
     @ComponentScan({"com.ca.mfaas.enable", "com.ca.mfaas.product"})
     ...
     public class DiscoverableClientSampleApplication {...
     ```
2. Add the Zowe Artifactory repository definition to the list of repositories in Gradle or Maven build systems. Use the code block that corresponds to your build system.
    * In a Gradle build system, add the following code to the `build.gradle` file into the `repositories` block.

      **Note:** Valid Zowe Artifactory credentials must be used.  

        ```
      maven {
          url 'https://gizaartifactory.jfrog.io/gizaartifactory/libs-release'
          credentials {
              username 'apilayer-build'
              password 'lHj7sjJmAxL5k7obuf80Of+tCLQYZPMVpDob5oJG1NI='
          }            
      }
        ```
      **Note:** You can define `gradle.properties` file where you can set your username, password and the
      read-only repo URL for access to the Zowe Artifactory.This way, you do not need to hardcode the username,
      password, and read-only repo URL in your `gradle.build` file.

      **Example:**
      ```
         # Artifactory repositories for builds
         artifactoryMavenRepo=https://gizaartifactory.jfrog.io/gizaartifactory/libs-release

         # Artifactory credentials for builds (not publishing):
         mavenUser=apilayer-build
         mavenPassword=lHj7sjJmAxL5k7obuf80Of+tCLQYZPMVpDob5oJG1NI=
      ```

    * In a Maven build system, follow these steps:

        a) Add the following code to the `pom.xml` file:

        ```
        <repository>
               <id>Gizaartificatory</id>
               <url>https://gizaartifactory.jfrog.io/gizaartifactory/libs-release</url>
        </repository>
        ```

        b) Create a `settings.xml` file and copy the following XML code block which defines the
        login credentials for the Zowe  Artifactory. Use valid credentials.  

        ```
        <?xml version="1.0" encoding="UTF-8"?>

        <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
        <servers>
           <server>
               <id>Gizaartificatory</id>
               <username>{artifactoryUser}</username>
               <password>{artifactoryPassword}</password>
           </server>
        </servers>
        </settings>
        ```  

        c) Copy the `settings.xml` file inside `${user.home}/.m2/` directory.

3. Add a JAR package to the list of dependencies in Gradle or Maven build systems. Zowe API Mediation Layer supports Spring Boot versions 1.5.9 and 2.0.4.

    * If you use Spring Boot release 1.5.x in a Gradle build system, add the following code to the build.gradle file into the `dependencies` block:

    ```
        compile group: 'com.ca.mfaas.sdk', name: 'mfaas-integration-enabler-spring-v1-springboot-1.5.9.RELEASE', version: '0.3.0-SNAPSHOT'
    ```
     * If you use Spring Boot release 1.5.x in a Maven build system, add the following code to the `pom.xml` file:

    ```
        <dependency>
              <groupId>com.ca.mfaas.sdk</groupId>
              <artifactId>mfaas-integration-enabler-spring-v1-springboot-1.5.9.RELEASE</artifactId>
              <version>0.3.0-SNAPSHOT</version>
        </dependency>
    ```
     * If you use the Spring Boot release 2.0.x in a Gradle build system, add the following code to the `build.gradle` file into the `dependencies` block:   
        ```
        compile group: 'com.ca.mfaas.sdk', name: 'mfaas-integration-enabler-spring-v2-springboot-2.0.4.RELEASE', version: '0.3.0-SNAPSHOT'
        ```

     * If you use the Spring Boot release 2.0.x in a Maven build system, add the following code to the `pom.xml` file:  
        ```
        <dependency>
               <groupId>com.ca.mfaas.sdk</groupId>
               <artifactId>mfaas-integration-enabler-spring-v2-springboot-2.0.4.RELEASE</artifactId>
               <version>0.3.0-SNAPSHOT</version>
        </dependency>
        ```  

Congratulations! You have successfully added enabler annotations to your service code! In the next task, you will build your service to include the code pieces that make it discoverable in the API Mediation Layer and to add Swagger documentation.


## Add API Layer onboarding configuration
 As an API service developer, you set multiple configuration settings in your application.yml that correspond to the API Layer. These settings enable an API to be discoverable and included in the API catalog. Some of the settings in the application.yml are internal and are set by the API service developer. Some settings are externalized and set by the customer system administrator. Those external settings are service parameters and are in the format: ${environment.*}.

**Follow these steps:**

1. Add the following #MFAAS configuration section in your `application.yml`:

    ```
      ##############################################################################################
      # MFAAS configuration section
      ##############################################################################################
      mfaas:
          discovery:
              serviceId: ${environment.serviceId}
              locations: ${environment.discoveryLocations}
              enabled: ${environment.discoveryEnabled:true}
              endpoints:
                  statusPage: ${mfaas.server.scheme}://${mfaas.service.hostname}:${mfaas.server.port}${mfaas.server.contextPath}/application/info
                  healthPage: ${mfaas.server.scheme}://${mfaas.service.hostname}:${mfaas.server.port}${mfaas.server.contextPath}/application/health
                  homePage: ${mfaas.server.scheme}://${mfaas.service.hostname}:${mfaas.server.port}${mfaas.server.contextPath}/
              info:
                  serviceTitle:  ${environment.serviceTitle}
                  description:  ${environment.serviceDescription}
                  # swaggerLocation: resource_location_of_your_static_swagger_doc.json
              fetchRegistry: false
              region: default
          service:
              hostname: ${environment.hostname}
              ipAddress: ${environment.ipAddress}
          catalog-ui-tile:
              id: yourProductFamilyId
              title: Your API service product family title in the API catalog dashboard tile
              description: Your API service product family description in the API catalog dashboard tile
              version:  1.0.0
          server:
              scheme: http
              port: ${environment.port}
              contextPath: /yourServiceUrlPrefix

      eureka:
          instance:
              appname: ${mfaas.discovery.serviceId}
              hostname: ${mfaas.service.hostname}
              statusPageUrlPath: ${mfaas.discovery.endpoints.statusPage}
              healthCheckUrl: ${mfaas.discovery.endpoints.healthPage}
              homePageUrl: ${mfaas.discovery.endpoints.homePage}
              metadata-map:
                  routed-services:
                      api_v1:
                          gateway-url: "api/v1"
                          service-url: ${mfaas.server.contextPath}

                      api-doc:
                          gateway-url: "api/v1/api-doc"
                          service-url: ${mfaas.server.contextPath}/api-doc
                  mfaas:
                      api-info:
                          apiVersionProperties:
                              v1:
                                  title: Your API title for swagger JSON which is displayed in API Catalog / service / API Information
                                  description: Your API description for swagger JSON
                                  version: 1.0.0
                                  basePackage: your.service.base.package.for.swagger.annotated.controllers
                                  # apiPattern: /v1/.*  # alternative to basePackage for exposing endpoints which match the regex pattern to swagger JSON
                      discovery:
                          catalogUiTile:
                              id: ${mfaas.catalog-ui-tile.id}
                              title:  ${mfaas.catalog-ui-tile.title}
                              description: ${mfaas.catalog-ui-tile.description}
                              version: ${mfaas.catalog-ui-tile.version}
                          enableApiDoc: ${mfaas.discovery.info.enableApiDoc:true}
                          service:
                              title: ${mfaas.discovery.info.serviceTitle}
                              description: ${mfaas.discovery.info.description}
          client:
              enabled: ${mfaas.discovery.enabled}
              healthcheck:
                  enabled: true
              serviceUrl:
                  defaultZone: ${mfaas.discovery.locations}
              fetchRegistry:  ${mfaas.discovery.fetchRegistry}
              region: ${mfaas.discovery.region}

      ##############################################################################################
      # Application configuration section
      ##############################################################################################
      server:
          # address: ${mfaas.service.ipAddress}
          port: ${mfaas.server.port}
          servlet:
              contextPath: ${mfaas.server.contextPath}

      spring:
          application:
              name: ${mfaas.discovery.serviceId}      

2. Change the MFaaS parameters to correspond with your API service specifications. Most of these internal parameters contain "your service" text.

    **Note:**  `${mfaas.*}` variables are used throughout the `application.yml` sample to reduce the number of required changes.

    **Tip:** When existing parameters set by the system administrator are already present in your configuration file (for example, `hostname, address, contextPath,` and `port`), we recommend that you replace them with the corresponding MFaaS properties.

    a. **Discovery Parameters**

    * **mfaas.discovery.serviceId**

         Specifies the service instance identifier to register in the API Layer installation. The service ID is used in the URL for routing to the API service through the gateway. The service ID uniquely identifies instances of a micro service in the API mediation layer. The system administrator at the customer site defines this parameter.  

        **Important!**  Ensure that the service ID is set properly with the following considerations:

         * When two API services use the same service ID, the API gateway considers the services to be clones. An incoming API request can be routed to either of them.
         * The same service ID should be set for only multiple API service instances for API scalability.
         * The service ID value must contain only lowercase alphanumeric characters.
         * The service ID cannot contain more than 40 characters.
         * The service ID is linked to security resources. Changes to the service ID require an update of security resources.

         **Examples:**

         * If the customer system administrator sets the service ID to `sysviewlpr1`, the API URL in the API Gateway appears as the following URL:
            ```
            https://gateway:port/api/v1/sysviewlpr1/endpoint1/...
            ```
         * If customer system administrator sets the service ID to vantageprod1, the API URL in the API Gateway appears as the following URL:             
            ```
            http://gateway:port/api/v1/vantageprod1/endpoint1/...
            ```

    * **mfaas.discovery.locations**

        Specifies the public URL of the Discovery Service (eureka). The system administrator at the customer site defines this parameter.

        **Example:**
         ```
         http://eureka:password@141.202.65.33:10311/eureka/
         ```
    * **mfaas.discovery.enabled**

        Specifies whether the API service instance is to be discovered in the API Layer. The system administrator at the customer site defines this parameter. Set this parameter to true if the API Layer is installed and configured. Otherwise, you can set this parameter to `false` to exclude an API service instances from the API Layer.    
    * **mfaas.discovery.fetchRegistry**

        Specifies whether the API service is to receive regular update notifications from the discovery service. Under most circumstances, you can accept the default value of `false` for the parameter.

    * **mfaas.discovery.region**

        Specifies the geographical region. This parameter is required by the Eureka client. Under most circumstances you can accept the value `default` for the parameter.          

    b. **Service and Server Parameters**
    * **mfaas.service.hostname**

        Specifies the hostname of the system where the API service instance runs. This parameter is externalized and is set by the customer system administrator. The administrator ensures the hostname can be resolved by DSN to the IP address that is accessible by applications running on their z/OS systems.   
    * **mfaas.service.ipAddress**

        Specifies the local IP address of the system where the API service instance runs. This IP address may or may not be a public IP address. This parameter is externalized and set by the customer system administrator.
    * **mfaas.server.scheme**

       Specifies whether the API service is using the HTTPS protocol. This value can be set to https or http  depending on whether your service is using SSL.
    * **mfaas.server.port**

       Specifies the port that is used by the API service instance. This parameter is externalized and set by the customer system administrator.
    * **mfaas.server.contextPath**

       Specifies the prefix that is used within your API service URL path.

       **Examples:**
       * If your API service does not use an extra prefix in the URL (for example, `http://host:port/endpoint1/`), set this value to /.
       * If your API service uses an extra URL prefix set the parameter to that prefix value.
         For the URL: `http://host:port/filemaster/endpoint1/`, set this parameter to `/filemaster`.  
       * In both examples, the API service URL appears as the following URL when routed through the gateway:
            ```
            http://gateway:port/serviceId/endpoint1/
            ```

    c. **API Catalog Parameters**

      These parameters are used to populate API Catalog. The API Catalog contains information about every registered API service. The catalog also groups related APIs. Each API group has its own name and description. Catalog groups are constructed in real-time based on information that is provided by the API services. Each group is displayed as a "tile" in the API Catalog UI dashboard.
      * **mfaas.catalog-ui-tile.id**

        Specifies the unique identifier for the API services product family. This is the grouping value used by the API Layer to group multiple API services together into "tiles". Each unique identifier represents a single API Catalog UI dashboard tile. Specify a value that does not interfere with API services from other products.

      * **mfaas.catalog-ui-tile.title**

        Specifies the title of the API services product family. This value is displayed in the API Catalog UI dashboard as the tile title

      * **mfaas.catalog-ui-tile.description**

        Specifies the detailed description of the API services product family. This value is displayed in the API Catalog UI dashboard as the tile description

      * **mfaas.catalog-ui-tile.version**

        Specifies the semantic version of this API Catalog tile. Increase the version when you introduce new changes to the API services product family details (title and description).

      * **mfaas.discovery.info.serviceTitle**

        Specifies the human readable name of the API service instance (for example, "Endevor Prod" or "Sysview LPAR1"). This value is displayed in the API Catalog when a specific API service instance is selected. This parameter is externalized and set by the customer system administrator.

         ![Service Status](../../images/api-mediation/Service-Status.png)

         **Tip:** We recommend that you provide a good default value or give good naming examples to the customers.
      * **mfaas.discovery.info.description**

          Specifies a short description of the API service.

          **Example:** "CA Endevor SCM - Production Instance" or "CA SYSVIEW running on LPAR1".
          This value is displayed in the API Catalog when a specific API service instance is selected. This parameter is externalized and set by the customer system administrator.  

        **Tip:** We recommend that you provide a good default value or give good naming examples to the customers. Describe the service so that the end user knows the function of the service.
      * **mfaas.discovery.info.swaggerLocation**

        Specifies the location of a static swagger document. The JSON document contained in this file is displayed instead of the automatically generated API documentation. The JSON file must contain a valid OpenAPI 2.x Specification document. This value is optional and commented out by default.

        **Note:** Specifying a `swaggerLocation` value disables the automated JSON API documentation generation with the SpringFox library. By disabling auto-generation, you need to keep the contents of the manual swagger definition consistent with your endpoints. We recommend to use auto-generation to prevent incorrect endpoint definitions in the static swagger documentation.  

    d. **Metadata Parameters**

      The routing rules can be modified with parameters in the metadata configuration code block.  

      **Note:** If your REST API does not conform to MFaaS REST API Building codes, configure routing to transform your actual endpoints (serviceUrl) to gatewayUrl format. For more information see: [REST API Building Codes](https://docops.ca.com/display/IWM/Guidelines+for+Building+a+New+API)
      * `eureka.instance.metadata-map.routed-services.<prefix>`

        Specifies a name for routing rules group. This parameter is only for logical grouping of further parameters. You can specify an arbitrary value but it is a good development practice to mention the group purpose in the name.

        **Examples:**
        ```
        api-doc
        api_v1
        api_v2
        ```
      * `eureka.instance.metadata-map.routed-services.<prefix>.gatewayUrl`

           Both gateway-url and service-url parameters specify how the API service endpoints are mapped to the API gateway endpoints. The gateway-url parameter sets the target endpoint on the gateway.
      * `metadata-map.routed-services.<prefix>.serviceUrl`

          Both gateway-url and service-url parameters specify how the API service endpoints are mapped to the API gateway endpoints. The service-url parameter points to the target endpoint on the gateway.

        **Important!** Ensure that each of the values for gatewayUrl parameter are unique in the configuration. Duplicate gatewayUrl values may cause requests to be routed to the wrong service URL.

        **Note:** The endpoint `/api-doc` returns the API service Swagger JSON. This endpoint is introduced by the `@EnableMfaasInfo` annotation and is utilized by the API Catalog.

    e. **Swagger Api-Doc Parameters**

      Configures API Version Header Information, specifically the [InfoObject](https://swagger.io/specification/#infoObject) section, and adjusts Swagger documentation that your API service returns. Use the following format:

      ```
    api-info:
       apiVersionProperties:
          v1:
              title: Your API title for swagger JSON which is displayed in API Catalog / service / API Information
              description: Your API description for swagger JSON
              version: 1.0.0
              basePackage: your.service.base.package.for.swagger.annotated.controllers
              # apiPattern: /v1/.*  # alternative to basePackage for exposing endpoints which match the regex pattern to swagger JSON
      ```   

    The following parameters describe the function of the specific version of an API. This information is included in the swagger JSON and displayed in the API Catalog:

    ![API information detail](../../images/api-mediation/Service-detail.png)


   * **v1**

     Specifies the major version of your service API: `v1, v2,` etc.
   * **title**

        Specifies the title of your service API.
   * **description**             

        Specifies the high-level function description of your service API.
   * **version**

        Specifies the actual version of the API in semantic format.
   * **basePackage**

        Specifies the package where the API is located. This option only exposes endpoints that are defined in a specified java package. The parameters `basePackage` and `apiPattern` are mutually exclusive. Specify only one of them and remove or comment out the second one.
   * **apiPattern**

        This option exposes any endpoints that match a specified regular expression. The parameters `basePackage` and `apiPattern` are mutually exclusive. Specify just one of them and remove or comment out the second one.

        **Tip:** You have three options to make your endpoints discoverable and exposed: `basePackage`, `apiPattern`, or none (if you do not specify a parameter). If `basePackage` or `apiPattern` are not defined, all endpoints in the Spring Boot app are exposed.

## Setup key store with the service certificate

To register with the API Mediation Layer, a service is required to have a certificate that is trusted by API Mediation Layer.

**Follow these steps:**

1. Follow instructions at [Generating certificate for a new service on localhost](https://github.com/zowe/api-layer/tree/master/keystore#generating-certificate-for-a-new-service-on-localhost)

    When a service is running on localhost, the command can have the following format:

       <api-layer-repository>/scripts/apiml_cm.sh --action new-service --service-alias localhost --service-ext SAN=dns:localhost.localdomain,dns:localhost --service-keystore keystore/localhost.keystore.p12 --service-truststore keystore/localhost.truststore.p12 --service-dname "CN=Sample REST API Service, OU=Mainframe, O=Zowe, L=Prague, S=Prague, C=Czechia" --service-password password --service-validity 365 --local-ca-filename <api-layer-repository>/keystore/local_ca/localca    

    Alternatively, for the purpose of local development, copy or use the `<api-layer-repository>/keystore/localhost.truststore.p12` in your service without generating a new certificate.

2. Update the configuration of your service `application.yml` to contain the HTTPS configuration by adding the following code:

        server:
            ssl:
                protocol: TLSv1.2
                ciphers: TLS_RSA_WITH_AES_128_CBC_SHA,TLS_DHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384,TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384,TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_EMPTY_RENEGOTIATION_INFO_SCSV
                keyAlias: localhost
                keyPassword: password
                keyStore: keystore/localhost.keystore.p12
                keyStoreType: PKCS12
                keyStorePassword: password
                trustStore: keystore/localhost.truststore.p12
                trustStoreType: PKCS12
                trustStorePassword: password

**Note:** You need to define both key store and trust store even if your server is not using HTTPS port.


## Externalize API Layer configuration parameters

The following list summarizes the API Layer parameters that are set by the customer system administrator:

   * `mfaas.discovery.enabled: ${environment.discoveryEnabled:true}`
   * `mfaas.discovery.locations: ${environment.discoveryLocations}`
   * `mfaas.discovery.serviceID: ${environment.serviceId}`
   * `mfaas.discovery.info.serviceTitle: ${environment.serviceTitle}`
   * `mfaas.discovery.info.description: ${environment.serviceDescription}`
   * `mfaas.service.hostname: ${environment.hostname}`
   * `mfaas.service.ipAddress: ${environment.ipAddress}`
   * `mfaas.server.port: ${environment.port}`

**Tip:** Spring Boot applications are configured in the `application.yml` and `bootstrap.yml` files that are located in the USS file system. However, system administrators prefer to provide configuration through the mainframe sequential data set (or PDS member). To override Java values, use Spring Boot with an external YML file, environment variables, and Java System properties. For Mainframe as a Service applications, we recommend that you use Java System properties.    

Java System properties are defined using `-D` options for Java. Java System properties can override any configuration. Those properties that are likely to change are defined as `${environment.variableName}:`     

```
IJO="$IJO -Denvironment.discoveryEnabled=.."
IJO="$IJO -Denvironment.discoveryLocations=.."

IJO="$IJO -Denvironment.serviceId=.."
IJO="$IJO -Denvironment.serviceTitle=.."
IJO="$IJO -Denvironment.serviceDescription=.."
IJO="$IJO -Denvironment.hostname=.."
IJO="$IJO -Denvironment.ipAddress=.."
IJO="$IJO -Denvironment.port=.."
```     
The `discoveryLocations` (public URL of the discovery service) value is found in the API Meditation Layer configuration, in the `*.PARMLIB(MASxPRM)` member and assigned to the MFS_EUREKA variable.

**Example:**
```
MFS_EUREKA="http://eureka:password@141.202.65.33:10011/eureka/")
```      

## Test your service

To test that your API instance is working and is discoverable, use the following validation tests:

### Validate that your API instance is still working

**Follow these steps:**

 1. Disable discovery by setting `discoveryEnabled=false` in your API service instance configuration.
 2. Run your tests to check that they are working as before.

### Validate that your API instance is discoverable

**Follow these steps:**
 1. Point your configuration of API instance to use the following discovery service:
    ```
    http://eureka:password@localhost:10011/eureka
    ```
 2. Start up the API service instance.
 3. Check that your API service instance and each of its endpoints are displayed in the API Catalog
    ```
    https://localhost:10010/ui/v1/caapicatalog/
    ```

 4. Check that you can access your API service endpoints through the gateway.

   **Example:**
   ```
   https://localhost:10010/api/v1/
   ```

 5. Check that you can still access your API service endpoints directly outside of the gateway.

# Zowe scenario 3: Part 1 -  Stand up a local version of the Example Zowe Application Server

## Overview

This scenario guides you through the process of configuring a simple Zowe Application Server with a few applications included. By the end of this scenario, you will know how to:

- Acquire the source code
- Acquire external components
- Set the server configuration
- Build application plug-ins
- Deploy server configuration files
- Run the server
- Connect in a browser!

No previous knowledge of Zowe WebUI is needed.

Please wait a moment while your development environment loads (it takes a minute or so).

## 1. Acquire the source code

To get started, first clone or download the github capstone repository, https://github.com/zowe/zlux
Because we will be configuring ZSS on z/OS's USS, and the Zowe Application Server on a LUW host, you will need to place the contents on both systems.
If you are using git, use the following commands:

```
git clone --recursive git@github.com:zowe/zlux.git
cd zlux
git submodule foreach "git checkout master"
cd zlux-build
```

At this point, you have the latest code from each repository on your system.
Continue from within `zlux-example-server`.

## 2. Acquire external components

Application plug-ins and external servers can require contents that are not in the Zowe github repositories. In the case of the `zlux-example-server`, there is a a ZSS binary component which cannot be found in the repositories. To obtain the ZSS binary component, contact Rocket.

After you obtain the ZSS binary component, you should receive _zssServer_.
This must be placed within _zlux-build/externals/Rocket_, on the z/OS host.
For example:

```
mkdir externals
mkdir externals/Rocket

//(on z/OS only)
mv zssServer externals/Rocket
```

## 3. Set the server configuration

Read the [Configuration](https://github.com/zowe/zlux/wiki/Configuration-for-zLUX-Proxy-Server-&-ZSS) wiki page for a detailed explanation of the primary items that you will want to configure for your first server.

In short, ensure that within the `config/zluxserver.json` file, **node.http.port** or **node.https.port** and the other HTTPS parameters are set to your liking on the LUW host, and that **zssPort** is set on the z/OS host.

Before you continue, if you intend to use the terminal, at this time (temporarily) it must be pre-configured to know the destination host.
Edit _../tn3270-ng2/\_defaultTN3270.json_ to set _host_ and _port_ to a valid TN3270 server telnet host and port and then save the file.
Edit _../vt-ng2/\_defaultVT.json_ to set _host_ and _port_ to a valid ssh host and port and then save the file.

## 4. Build application plug-ins

**Note:** NPM is used when building application plug-ins. The version of NPM needed for the build to succeed should be at least 5.4. You can update NPM by executing `npm install -g npm`

Application plug-ins can contain server and web components. The web components must be built, as webpack is involved in optimized packaging. Server components are also likely to need building if they require external dependencies from NPM, use native code, or are written in typescript.

This example server only needs transpilation and packaging of web components, and therefore we do not need any special build steps for the host running ZSS.

Instead, on the host running the Zowe Application Server, run the script that will automatically build all included application plug-ins.
Under `zlux-build` run,

```
//Windows
build.bat

//Otherwise
build.sh
```

This will take some time to complete.

_Note: You will need to have `ant` and `ant-contrib` installed_

## 5. Deploy server configuration files

If you are running the Zowe Application Server separate from ZSS, ensure the ZSS installation configuration is deployed. You can accomplish this by navigating to `zlux-build` and running the following:

```
ant deploy
```

On the other hand, if you are running ZSS and the Zowe Application Server on the same host, _build.sh_ and _build.bat_ execute _deploy_ and therefore this task was accomplished in step 4.

However, if you need to change the server configuration files or if you want to add more application plug-ins to be included at startup, you must update the deploy content to reflect this. Simply running `deploy.bat` or `deploy.sh` will accomplish this, but files such as `zluxserver.json` are only read at startup, so a reload of the Zowe Application Server and ZSS would be required.

## 6. Run the server

At this point, all server files have been configured and the application plug-ins built, so ZSS and the Zowe Application Server are ready to run.
First, from the z/OS system, start ZSS.

```
cd ../zlux-example-server/bin
./zssServer.sh
```

If the zssServer server did not start, two common sources of error are:

1. The _zssPort_ chosen is already occupied. To fix this, edit _config/zluxserver.json_ to choose a new one, and re-run _build/deploy.sh_ to make the change take effect.
2. The zssServer binary does not have the APF bit set. Because this server is meant for secure services, it is required. To fix this, execute `extattr +a zssServer`. Note that you might need to alter the execute permissions of `zssServer.sh` in the event that the previous command is not satisfactory (for example: chmod +x zssServer.sh)

Second, from the system with the Zowe Application Server, start it with a few parameters to hook it to ZSS.

```
cd ../zlux-example-server/bin

// Windows:
nodeServer.bat <parameters>

// Others:
nodeServer.sh <parameters>
```

Valid parameters for nodeServer are as follows:

- _-h_: Specifies the hostname where ZSS can be found. Use as `-h \<hostname\>`
- _-P_: Specifies the port where ZSS can be found. Use as `-P \<port\>`. This overrides _zssPort_ from the configuration file.
- _-p_: Specifies the HTTP port to be used by the Zowe Application Server. Use as `-p <port>`. This overrides _node.http.port_ from the configuration file.
- _-s_: Specifies the HTTPS port to be used by the Zowe Application Server. Use as `-s <port>`. This overrides _node.https.port_ from the configuration file.
- _--noChild_: If specified, tells the server to ignore and skip spawning of child processes defined as _node.childProcesses_ in the configuration file.

In the example where we run ZSS on a host named `mainframe.zowe.com`, running on zssPort = 19997, the Zowe Application Server running on Windows could be started with the following:

`nodeServer.bat -h mainframe.zowe.com -P 19997 -p 19998`

After which we would be able to connect to the Zowe Application Server at port 19998.

**NOTE:** the parameter parsing is provided by [argumentParser.js](https://github.com/zowe/zlux-proxy-server/blob/master/js/argumentParser.js), which allows for a few variations of input, depending on preference. For example, the following are all valid ways to specify the ZSS host:

- **-h myhost.com**
- **-h=myhost.com**
- **--hostServer myhost.com**
- **--hostServer=myhost.com**

When the Zowe Application Server has started, one of the last messages you will see as bootstrapping completes is that the server is listening on the HTTP/s port. At this time, you should be able to use the server.

## 7. Connect in a browser

Now that ZSS and the Zowe Application Server are both started, you can access this instance by pointing your web browser to the Zowe Application Server.
In this example, the address you will want to go to first is the location of the window management application - Zowe Desktop. The URL is:

`http(s)://\<zLUX Proxy Server\>:\<node.http(s).port\>/ZLUX/plugins/com.rs.mvd/web/index.html`

Once here, a Login window is presented with a few example application plug-ins in the taskbar at the bottom of the window. To try the application plug-ins to see how they interact with the framework, can login with your mainframe credentials.

- tn3270-ng2: This application communicates with the Zowe Application Server to enable a TN3270 connection in the browser.
- subsystems: This application shows various z/OS subsystems installed on the host the ZSS runs on. This is accomplished through discovery of these services by the application's portion running in the ZSS context.
- sample-app: A simple app that shows how a Zowe Application Framework application frontend (Angular) component can communicate with an application backend (REST) component.

### Deploy example

```
// All paths relative to zlux-example-server/js or zlux-example-server/bin
// In real installations, these values will be configured during the install.
  "rootDir":"../deploy",
  "productDir":"../deploy/product",
  "siteDir":"../deploy/site",
  "instanceDir":"../deploy/instance",
  "groupsDir":"../deploy/instance/groups",
  "usersDir":"../deploy/instance/users"
```

## Application plug-in configuration

This section does not cover dynamic runtime inclusion of application plug-ins, but rather application plug-ins that are defined in advance.
In the configuration file, a directory can be specified which contains JSON files that tell the server what application plug-in to include and where to find it on disk. The backend of these application plug-ins use the Server's Plugin structure, so much of the server-side references to application plug-ins use the term "Plugin".

To include application plug-ins, be sure to define the location of the `Plugins` directory in the configuration file, through the top-level attribute _pluginsDir_

**NOTE:** In this repository, the directory for these JSON files is `/plugins`. To separate configuration files from runtime files, the `zlux-example-server` repository copies the contents of this folder into `/deploy/instance/ZLUX/plugins`. So, the example configuration file uses the latter directory.

### Plugins directory example

```
// All paths relative to zlux-example-server/js or zlux-example-server/bin
// In real installations, these values will be configured during the install.
//...
  "pluginsDir":"../deploy/instance/ZLUX/plugins",
```

## ZSS Configuration

Running ZSS requires a JSON configuration file that is similar (or the same as) the one used for the Zowe Application Server. The attributes that are needed for ZSS, at minimum, are:_rootDir_, _productDir_, _siteDir_, _instanceDir_, _groupsDir_, _usersDir_, _pluginsDir_ and **zssPort**. All of these attributes have the same meaning as described above for the Zowe Application Server, but if the Zowe Application Server and ZSS are not run from the same location, then these directories can be different.

The **zssPort** attribute is specific to ZSS. This is the TCP port on which ZSS will listen to be contacted by the Zowe Application Server. Define this port in the configuration file as a value between 1024-65535.

### Connecting Zowe Application Server to ZSS

When running the Zowe Application Server, simply specify a few flags to declare which ZSS instance the Zowe Application Framework will proxy ZSS requests to:

- _-h_: Declares the host where ZSS can be found. Use as `-h \<hostname\>`
- _-P_: Declares the port at which ZSS is listening. Use as `-P \<port\>`





# Zowe scenario 3: Part 2 - Add a new application plug-in to the Zowe WebUI

This scenario walks you through the process of adding new apps to the Zowe WebUI, and teaches you how to communicate with other parts of Zowe. By the end of this scenario, you will know how to:

- Create an app that shows up on the Desktop
- Create a dataservice that implements a simple REST API

No previous knowledge of Zowe WebUI is needed.

Please wait a moment while your development environment loads (it takes a minute or so).

## Constructing an application skeleton

Download the skeleton code from the [project repository](https://github.com/zowe/workshop-user-browser-app). Next, move the project into the `zlux` source folder created in the prerequisite tutorial.

If you look within this repository, you'll see that a few boilerplate files already exist to help you get your first application plug-in running quickly. The structure of this repository follows the guidelines for Zowe application plug-in filesystem layout, which you can read more about [on the wiki](https://github.com/zowe/zlux/wiki/ZLUX-App-filesystem-structure).

### Defining your first plug-in

Where do you start when making an application plug-in? In the Zowe Application Framework, an application plug-in is a plug-in of type "Application". Every plug-in is bound by their `pluginDefinition.json` file, which describes its properties.
Let's start by creating this file.

Create a file, `pluginDefinition.json`, at the root of the `workshop-user-browser-app` folder.
The file should contain the following:

```json
{
  "identifier": "org.openmainframe.zowe.workshop-user-browser",
  "apiVersion": "1.0.0",
  "pluginVersion": "0.0.1",
  "pluginType": "application",
  "webContent": {
    "framework": "angular2",
    "launchDefinition": {
      "pluginShortNameKey": "userBrowser",
      "pluginShortNameDefault": "User Browser",
      "imageSrc": "assets/icon.png"
    },
    "descriptionKey": "userBrowserDescription",
    "descriptionDefault": "Browse Employees in System",
    "isSingleWindowApp": true,
    "defaultWindowStyle": {
      "width": 1300,
      "height": 500
    }
  }
}
```

A description of the particular values that are placed into this file can be found [on the wiki](https://github.com/zowe/zlux/wiki/Zlux-Plugin-Definition-&-Structure).

Note the following attributes:

- Our application has the unique identifier of `org.openmainframe.zowe.workshop-user-browser`, which can be used to refer to it when running Zowe.
- The application has a `webContent` attribute, because it will have a UI component that is visible in a browser.
  - The `webContent` section states that the application's code will conform to Zowe's Angular application structure, due to it stating `"framework": "angular2"`
  - The application plug-in has certain characteristics that the user will see, such as:
    - The default window size (`defaultWindowStyle`),
    - An application plug-in icon that we provided in `workshop-user-browser-app/webClient/src/assets/icon.png`,
    - That we should see it in the browser as an application plug-in named `User Browser`, the value of `pluginShortNameDefault`.

### Constructing a Simple Angular UI

Angular application plug-ins for Zowe are structured such that the source code exists within `webClient/src/app`. In here, you can create modules, components, templates and services in whatever hierarchy desired. For the application plug-in we are making here however, we will add three files:

- userbrowser.module.ts
- userbrowser-component.html
- userbrowser-component.ts

At first, let's just build a shell of an application plug-in that can display some simple content.
Fill in each file with the following content.

**userbrowser.module.ts**

```typescript
import { NgModule } from '@angular/core'
import { CommonModule } from '@angular/common'
import { FormsModule, ReactiveFormsModule } from '@angular/forms'
import { HttpModule } from '@angular/http'

import { UserBrowserComponent } from './userbrowser-component'

@NgModule({
  imports: [FormsModule, ReactiveFormsModule, CommonModule],
  declarations: [UserBrowserComponent],
  exports: [UserBrowserComponent],
  entryComponents: [UserBrowserComponent]
})
export class UserBrowserModule {}
```

**userbrowser-component.html**

```html
<div class="parent col-11" id="userbrowserPluginUI">
{{simpleText}}
</div>

<div class="userbrowser-spinner-position">
  <i class="fa fa-spinner fa-spin fa-3x" *ngIf="resultNotReady"></i>
</div>
```

**userbrowser-component.ts**

```typescript
import {
  Component,
  ViewChild,
  ElementRef,
  OnInit,
  AfterViewInit,
  Inject,
  SimpleChange
} from '@angular/core'
import { Observable } from 'rxjs/Observable'
import { Http, Response } from '@angular/http'
import 'rxjs/add/operator/catch'
import 'rxjs/add/operator/map'
import 'rxjs/add/operator/debounceTime'

import {
  Angular2InjectionTokens,
  Angular2PluginWindowActions,
  Angular2PluginWindowEvents
} from 'pluginlib/inject-resources'

@Component({
  selector: 'userbrowser',
  templateUrl: 'userbrowser-component.html',
  styleUrls: ['userbrowser-component.css']
})
export class UserBrowserComponent implements OnInit, AfterViewInit {
  private simpleText: string
  private resultNotReady: boolean = false

  constructor(
    private element: ElementRef,
    private http: Http,
    @Inject(Angular2InjectionTokens.LOGGER) private log: ZLUX.ComponentLogger,
    @Inject(Angular2InjectionTokens.PLUGIN_DEFINITION)
    private pluginDefinition: ZLUX.ContainerPluginDefinition,
    @Inject(Angular2InjectionTokens.WINDOW_ACTIONS)
    private windowAction: Angular2PluginWindowActions,
    @Inject(Angular2InjectionTokens.WINDOW_EVENTS)
    private windowEvents: Angular2PluginWindowEvents
  ) {
    this.log.info(`User Browser constructor called`)
  }

  ngOnInit(): void {
    this.simpleText = `Hello World!`
    this.log.info(`App has initialized`)
  }

  ngAfterViewInit(): void {}
}
```

### Packaging Your Web application plug-in

At this time, we've made the source for a Zowe application plug-in that should open in the Zowe Desktop with a greeting to the planet.
Before we're ready to use it however, we must transpile the typescript and package the application plug-in. This will require a few build tools first. We'll make an NPM package in order to facilitate this.

Let's create a `package.json` file within `workshop-user-browser-app/webClient`.
While a package.json can be created through other means such as `npm init` and packages can be added through commands such as `npm install --save-dev typescript@2.9.0`, we'll opt to save time by just pasting these contents in:

```json
{
  "name": "workshop-user-browser",
  "version": "0.0.1",
  "scripts": {
    "start": "webpack --progress --colors --watch",
    "build": "webpack --progress --colors",
    "lint": "tslint -c tslint.json \"src/**/*.ts\""
  },
  "private": true,
  "dependencies": {},
  "devDependencies": {
    "@angular/animations": "~6.0.9",
    "@angular/common": "~6.0.9",
    "@angular/compiler": "~6.0.9",
    "@angular/core": "~6.0.9",
    "@angular/forms": "~6.0.9",
    "@angular/http": "~6.0.9",
    "@angular/platform-browser": "~6.0.9",
    "@angular/platform-browser-dynamic": "~6.0.9",
    "@angular/router": "~6.0.9",
    "@zlux/grid": "git+https://github.com/zowe/zlux-grid.git",
    "@zlux/widgets": "git+https://github.com/zowe/zlux-widgets.git",
    "angular2-template-loader": "~0.6.2",
    "copy-webpack-plugin": "~4.5.2",
    "core-js": "~2.5.7",
    "css-loader": "~1.0.0",
    "exports-loader": "~0.7.0",
    "file-loader": "~1.1.11",
    "html-loader": "~0.5.5",
    "rxjs": "~6.2.2",
    "rxjs-compat": "~6.2.2",
    "source-map-loader": "~0.2.3",
    "ts-loader": "~4.4.2",
    "tslint": "~5.10.0",
    "typescript": "~2.9.0",
    "webpack": "~4.0.0",
    "webpack-cli": "~3.0.0",
    "webpack-config": "~7.5.0",
    "zone.js": "~0.8.26"
  }
}
```

Before we can build, we first need to tell our system where our example server is located. While we could provide the explicit path to our server in our project, creating an environmental variable with this location will speed up future projects.

To add an environmental variable on a Unix based machine:

1. `cd ~`
2. `nano .bash_profile`
3. Add `export MVD_DESKTOP_DIR=/Users/<user-name>/path/to/zlux/zlux-app-manager/virtual-desktop/`
4. Save and exit
5. `source ~/.bash_profile`

Now we are ready to build.
Let's set up our system to automatically perform these steps every time we make updates to the application plug-in.

1. Open a command prompt to `workshop-user-browser-app/webClient`
1. Execute `npm install`
1. Execute `npm run-script start`

After the first execution of the transpilation and packaging concludes, you should have `workshop-user-browser-app/web` populated with files that can be served by the Zowe Application Server.

### Adding Your application plug-in to the Zowe Desktop

At this point, your workshop-user-browser-app folder contains files for an application plug-in that could be added to a Zowe instance. We will add this to our own Zowe instance. First, ensure that the Zowe Application Server is not running. Then, navigate to the instance's root folder, `/zlux-example-server`.

Within, you'll see a folder, `plugins`. Take a look at one of the files within the folder. You can see that these are JSON files with the attributes **identifier** and **pluginLocation**. These files are what we call **Plugin Locators**, since they point to a plug-in to be included into the server.

Let's make one ourselves. Make a file `/zlux-example-server/plugins/org.openmainframe.zowe.workshop-user-browser.json`, with these contents:

```json
{
  "identifier": "org.openmainframe.zowe.workshop-user-browser",
  "pluginLocation": "../../workshop-user-browser-app"
}
```

When the server runs, it will check for these types of files in its `pluginsDir`, a location known to the server through its specification in the [server configuration file](https://github.com/zowe/zlux/wiki/Configuration-for-zLUX-Proxy-Server-&-ZSS#app-configuration). In our case, this is `/zlux-example-server/deploy/instance/ZLUX/plugins/`.

You could place the JSON directly into that location, but the recommended way to place content into the deploy area is through running the server deployment process.
Simply:

1. Open up a (second) command prompt to `zlux-build`
1. `ant deploy`

Now you're ready to run the server and see your application plug-in.

1. `cd /zlux-example-server/bin`
1. `./nodeServer.sh`
1. Open your browser to `https://hostname:port`
1. Login with your credentials
1. Open the application plug-in on the bottom of the page with the green 'U' icon.

Do you see the Hello World message from [this earlier step?](#constructing-a-simple-angular-ui). If so, you're in good shape! Now, let's add some content to the application plug-in.

## Building your first dataservice

An application plug-in can have one or more [Dataservices](https://github.com/zowe/zlux/wiki/ZLUX-Dataservices). A Dataservice is a REST or Websocket endpoint that can be added to the Zowe Application Server.

To demonstrate the use of a Dataservice, we'll add one to this application plug-in. The application plug-in needs to display a list of users, filtered by some value. Ordinarily, this sort of data would be contained within a database, where you can obtain rows in bulk and filter them in some manner. Retrieval of database contents, likewise, is a task that is easily representable through a REST API, so let's make one.

1. Create a file, `workshop-user-browser-app/nodeServer/ts/tablehandler.ts`
   Add the following contents:

```typescript
import { Response, Request } from 'express'
import * as table from './usertable'
import { Router } from 'express-serve-static-core'

const express = require('express')
const Promise = require('bluebird')

class UserTableDataservice {
  private context: any
  private router: Router

  constructor(context: any) {
    this.context = context
    let router = express.Router()

    router.use(function noteRequest(req: Request, res: Response, next: any) {
      context.logger.info('Saw request, method=' + req.method)
      next()
    })

    router.get('/', function(req: Request, res: Response) {
      res.status(200).json({ greeting: 'hello' })
    })

    this.router = router
  }

  getRouter(): Router {
    return this.router
  }
}

exports.tableRouter = function(context): Router {
  return new Promise(function(resolve, reject) {
    let dataservice = new UserTableDataservice(context)
    resolve(dataservice.getRouter())
  })
}
```

This is boilerplate for making a Dataservice. We lightly wrap ExpressJS Routers in a Promise-based structure where we can associate a Router with a particular URL space, which we will see later. If you were to attach this to the server, and do a GET on the root URL associated, you'd receive the {"greeting":"hello"} message.

### Working with ExpressJS

Let's move beyond hello world, and access this user table.

1. Within `workshop-user-browser-app/nodeServer/ts/tablehandler.ts`, add a function for returning the rows of the user table.

```typescript
const MY_VERSION = '0.0.1'
const METADATA_SCHEMA_VERSION = '1.0'
function respondWithRows(rows: Array<Array<string>>, res: Response): void {
  let rowObjects = rows.map(row => {
    return {
      firstname: row[table.columns.firstname],
      mi: row[table.columns.mi],
      lastname: row[table.columns.lastname],
      email: row[table.columns.email],
      location: row[table.columns.location],
      department: row[table.columns.department]
    }
  })

  let responseBody = {
    _docType: 'org.openmainframe.zowe.workshop-user-browser.user-table',
    _metaDataVersion: MY_VERSION,
    metadata: table.metadata,
    resultMetaDataSchemaVersion: '1.0',
    rows: rowObjects
  }
  res.status(200).json(responseBody)
}
```

Because we reference the usertable file through import, we are able to refer to its **metadata** and **columns** attributes here.
This **`respondWithRows`** function expects an array of rows, so we'll improve the Router to call this function with some rows so that we can present them back to the user.

2. Update the **UserTableDataservice** constructor, modifying and expanding upon the Router

```typescript
  constructor(context: any){
    this.context = context;
    let router = express.Router();
    router.use(function noteRequest(req: Request,res: Response,next: any) {
      context.logger.info('Saw request, method='+req.method);
      next();
    });
    router.get('/',function(req: Request,res: Response) {
      respondWithRows(table.rows,res);
    });

    router.get('/:filter/:filterValue',function(req: Request,res: Response) {
      let column = table.columns[req.params.filter];
      if (column===undefined) {
        res.status(400).json({"error":"Invalid filter specified"});
        return;
      }
      let matches = table.rows.filter(row=> row[column] == req.params.filterValue);
      respondWithRows(matches,res);
    });

    this.router = router;
  }
```

Zowe's use of ExpressJS Routers allows you to quickly assign functions to HTTP calls such as GET, PUT, POST, DELETE, or even websockets, and provides you with easy parsing and filtering of the HTTP requests so that there is very little involved in making a good API for users.

This REST API now allows for two GET calls to be made: one to root /, and the other to /_filter_/_value_. The behavior here is as is defined in [ExpressJS documentation](https://expressjs.com/en/guide/routing.html#route-parameters) for routers, where the URL is parameterized to give us arguments that we can feed into our function for filtering the user table rows before giving the result to **respondWithRows** for sending back to the caller.

### Adding your Dataservice to the Plugin Definition

Now that the Dataservice is made, add it to our Plugin's definition so that the server is aware of it, and then build it so that the server can run it.

1. Open a (third) command prompt to `workshop-user-browser-app/nodeServer`
1. Install dependencies, `npm install`
1. Invoke the NPM build process, `npm run-script start`
   1. If there are errors, go back to [building the dataservice](#building-your-first-dataservice) and make sure the files look correct.
1. Edit `workshop-user-browser-app/pluginDefinition.json`, adding a new attribute which declares Dataservices.

```json
"dataServices": [
    {
      "type": "router",
      "name": "table",
      "serviceLookupMethod": "external",
      "fileName": "tablehandler.js",
      "routerFactory": "tableRouter",
      "dependenciesIncluded": true
    }
],
```

Your full pluginDefinition.json should now be:

```json
{
  "identifier": "org.openmainframe.zowe.workshop-user-browser",
  "apiVersion": "1.0.0",
  "pluginVersion": "0.0.1",
  "pluginType": "application",
  "dataServices": [
    {
      "type": "router",
      "name": "table",
      "serviceLookupMethod": "external",
      "fileName": "tablehandler.js",
      "routerFactory": "tableRouter",
      "dependenciesIncluded": true
    }
  ],
  "webContent": {
    "framework": "angular2",
    "launchDefinition": {
      "pluginShortNameKey": "userBrowser",
      "pluginShortNameDefault": "User Browser",
      "imageSrc": "assets/icon.png"
    },
    "descriptionKey": "userBrowserDescription",
    "descriptionDefault": "Browse Employees in System",
    "isSingleWindowApp": true,
    "defaultWindowStyle": {
      "width": 1300,
      "height": 500
    }
  }
}
```

There's a few interesting attributes about the Dataservice we have specified here. First is that it is listed as `type: router`, which is because there are different types of Dataservices that can be made to suit the need. Second, the **name** is **table**, which determines both the name seen in logs but also the URL this can be accessed at. Finally, **fileName** and **routerFactory** point to the file within `workshop-user-browser-app/lib` where the code can be invoked, and the function that returns the ExpressJS Router, respectively.

4. [Restart the server](#adding-your-app-to-the-desktop) (as was done when adding the App initially) to load this new Dataservice. This is not always needed but done here for educational purposes.
5. Access `https://host:port/ZLUX/plugins/org.openmainframe.zowe.workshop-user-browser/services/table/` to see the Dataservice in action. It should return all of the rows in the user table, as you did a GET to the root / URL that we just coded.

## Adding your first Widget

Now that you can get this data from the server's new REST API, we need to make improvements to the web content of the application plug-in to visualize this. This means not only calling this API from the application plug-in, but presenting it in a way that is easy to read and extract information from.

### Adding your Dataservice to the application plug-in

Let's make some edits to **userbrowser-component.ts**, replacing the **UserBrowserComponent** Class's **ngOnInit** method with a call to get the user table, and defining **ngAfterViewInit**:

```typescript
  ngOnInit(): void {
    this.resultNotReady = true;
    this.log.info(`Calling own dataservice to get user listing for filter=${JSON.stringify(this.filter)}`);
    let uri = this.filter ? ZoweZLUX.uriBroker.pluginRESTUri(this.pluginDefinition.getBasePlugin(), 'table', `${this.filter.type}/${this.filter.value}`) : ZoweZLUX.uriBroker.pluginRESTUri(this.pluginDefinition.getBasePlugin(), 'table',null);
    setTimeout(()=> {
    this.log.info(`Sending GET request to ${uri}`);
    this.http.get(uri).map(res=>res.json()).subscribe(
      data=>{
        this.log.info(`Successful GET, data=${JSON.stringify(data)}`);
        this.columnMetaData = data.metadata;
        this.unfilteredRows = data.rows.map(x=>Object.assign({},x));
        this.rows = this.unfilteredRows;
        this.showGrid = true;
        this.resultNotReady = false;
      },
      error=>{
        this.log.warn(`Error from GET. error=${error}`);
        this.error_msg = error;
        this.resultNotReady = false;
      }
    );
    },100);
  }

  ngAfterViewInit(): void {
    // the flex table div is not on the dom at this point
    // have to calculate the height for the table by subtracting all
    // the height of all fixed items from their container
    let fixedElems = this.element.nativeElement.querySelectorAll('div.include-in-calculation');
    let height = 0;
    fixedElems.forEach(function (elem, i) {
      height += elem.clientHeight;
    });
    this.windowEvents.resized.subscribe(() => {
      if (this.grid) {
        this.grid.updateRowsPerPage();
      }
    });
  }
```

You might notice that we are referring to several instance variables that we have not declared yet. Let's add those within the **UserBrowserComponent** Class too, above the constructor.

```typescript
  private showGrid: boolean = false;
  private columnMetaData: any = null;
  private unfilteredRows: any = null;
  private rows: any = null;
  private selectedRows: any[];
  private query: string;
  private error_msg: any;
  private url: string;
  private filter:any;
```

Hopefully you are still running the command in the first command prompt, `npm run-script start`, which will rebuild your web content for the application whenever you make changes. You might see some errors, which we will resolve by adding the next portion of the application.

### Introducing ZLUX Grid

When **ngOnInit** runs, it will call out to the REST Dataservice and put the table row results into our cache, but we haven't yet visualized this in any way. We need to improve our HTML a bit to do that, and rather than reinvent the wheel, we have a table visualization library we can rely on - **ZLUX Grid**.

If you inspect `package.json` in the **webClient** folder, you'll see that we've already included @zlux/grid as a dependency (as a link to one of the Zowe github repositories) so it should have been pulled into the **node_modules** folder during the `npm install` operation. We just need to include it in the Angular code to make use of it. To do so, complete these steps:

1. Edit **webClient/src/app/userbrowser.module.ts**, adding import statements for the zlux widgets above and within the @NgModule statement:

```typescript
import { ZluxGridModule } from '@zlux/grid';
import { ZluxPopupWindowModule, ZluxButtonModule } from '@zlux/widgets'
//...
@NgModule({
imports: [FormsModule, HttpModule, ReactiveFormsModule, CommonModule, ZluxGridModule, ZluxPopupWindowModule, ZluxButtonModule],
//...
```

The full file should now be:

```typescript
*
  This Angular module definition will pull all of your Angular files together to form a coherent App
*/

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';
import { ZluxGridModule } from '@zlux/grid';
import { ZluxPopupWindowModule, ZluxButtonModule } from '@zlux/widgets'

import { UserBrowserComponent } from './userbrowser-component';

@NgModule({
  imports: [FormsModule, HttpModule, ReactiveFormsModule, CommonModule, ZluxGridModule, ZluxPopupWindowModule, ZluxButtonModule],
  declarations: [UserBrowserComponent],
  exports: [UserBrowserComponent],
  entryComponents: [UserBrowserComponent]
})
export class UserBrowserModule { }
```

2. Edit **userbrowser-component.html** within the same folder. Previously, it was just meant for presenting a Hello World message, so we should add some style to accommodate the zlux-grid element that we will also add to this template through a tag.

```html
<!-- In this HTML file, an Angular Template should be placed that will work together with your Angular Component to make a dynamic, modern UI -->

<div class="parent col-11" id="userbrowserPluginUI">
  <div class="fixed-height-child include-in-calculation">
      <button type="button" class="wide-button btn btn-default" value="Send">
        Submit Selected Users
      </button>
  </div>
  <div class="fixed-height-child height-40" *ngIf="!showGrid && !viewConfig">
    <div class="">
      <p class="alert-danger">{{error_msg}}</p>
    </div>
  </div>
  <div class="container variable-height-child" *ngIf="showGrid">
    <zlux-grid [columns]="columnMetaData | zluxTableMetadataToColumns"
    [rows]="rows"
    [paginator]="true"
    selectionMode="multiple"
    selectionWay="checkbox"
    [scrollableHorizontal]="true"
    (selectionChange)="onTableSelectionChange($event)"
    #grid></zlux-grid>
  </div>
  <div class="fixed-height-child include-in-calculation" style="height: 20px; order: 3"></div>
</div>

<div class="userbrowser-spinner-position">
  <i class="fa fa-spinner fa-spin fa-3x" *ngIf="resultNotReady"></i>
</div>
```

Note the key functions of this template:

- There's a button which when clicked will submit selected users (from the grid). We will implement this ability later.
- We show or hide the grid based on a variable `ngIf="showGrid"` so that we can wait to show the grid until there is data to present
- The zlux-grid tag pulls the Zowe Application Framework Grid widget into our application, and it has many variables that can be set for visualization, as well as functions and modes.
  - We allow the columns, rows, and metadata to be set dynamically by using the square bracket [ ] template syntax, and allow our code to be informed when the user selection of rows changes through `(selectionChange)="onTableSelectionChange($event)"`

3. Small modification to **userbrowser-component.ts** to add the grid variable, and set up the aforementioned table selection event listener, both within the **UserBrowserComponent** Class:

```js
@ViewChild('grid') grid; //above the constructor

onTableSelectionChange(rows: any[]):void{
    this.selectedRows = rows;
}
```

The previous section, [Adding your Dataservice to the application](#adding-your-dataservice-to-the-app) set the variables that are fed into the Zowe Application Framework Grid widget, so at this point the application should be updated with the ability to present a list of users in a grid.

If you are still running `npm run-script start` in a command prompt, it should now show that the application has been successfully built, and that means we are ready to see the results. Reload your browser's webpage and open the user browser application once more. Do you see the list of users in columns and rows that can be sorted and selected? If so, great, you've built a simple yet useful application within Zowe! Let's move on to the last portion of the application tutorial where we hook the Starter application and the User Browser application together to accomplish a task.

## Adding Zowe App-to-App Communication

Applications in Zowe can be useful and provide insight all by themselves, but a big advantage to using the Zowe Desktop is that applications can track and share context by user interaction. By having the foreground application request the application best suited for a task, the requested application can perform the task with context regarding the task data and purpose and you can accomplish a complex task by simple and intuitive means.

In the case of this tutorial, we're not only trying find a list of employees in a company (as was shown in the last step where the Grid was added and populated with the REST API), but to filter that list to find those employees who are best suited to the task we need to accomplish. So, our user browser application needs to be enhanced with two new abilities:

- Filter the user list to show only those users that meet the filter
- Send the subset of users selected in the list back to the App that requested a user list.

How do we do either task? Application-to-application communication! Applications can communicate with other applications in a few ways, but can be categorized into two interaction groups:

1. Launching an App with a context of what it should do
1. Messaging an App that's already open to a request or alert it of something

In either case, the application framework provides Actions as the objects to perform the communication. Actions not only define what form of communication should happen, but between which Apps. Actions are issued from one application, and are fulfilled by a target application. But, because there might be more than one instance or window of an application open, there are Target Modes:

- Open a new App window, where the message context is delivered in the form of a Launch Context
- Message a particular, or any of the currently open instances of the target App

### Adding the Starter application

In order to facilitate app-to-app communication, we need another application with which to communicate. A 'starter' application is provided which can be [found on github](https://github.com/zowe/workshop-starter-app).

As we did previously in the [Adding Your application to the Desktop](#adding-your-app-to-the-desktop) section, we need to move the application files to a location where they can be included in our `zlux-example-server`. We then need to add to the `plugins` folder in the example server and re-deploy.

1. Clone or download the starter app under the `zlux` folder

- `git clone https://github.com/zowe/workshop-starter-app.git`

2. Navigate to starter app and build it as before

- Install packages with `cd webClient` and then `npm install`
- Build the project using `npm start`

2. Next navigate to the `zlux-example-server`:

- create a new file under `/zlux-example-server/plugins/org.openmainframe.zowe.workshop-starter.json`
- Edit the file to contain:

```json
{
  "identifier": "org.openmainframe.zowe.workshop-starter",
  "pluginLocation": "../../workshop-starter-app"
}
```

3. Make sure the ./nodeServer is stopped before running `ant deploy` under `zlux-build`
4. Restart the ./nodeServer under `zlux-example-server/bin` with the appropriate parameters passed in.
5. Refresh the browser and verify that the app with a **Green S** is present in zLUX.

### Enabling Communication

We've already done the work of setting up the application's HTML and Angular definitions, so in order to make our application compatible with application-to-application communication, it only needs to listen for, act upon, and issue Zowe application Actions. Let's edit the typescript component to do that. Edit the **UserBrowserComponent** Class's constructor within **userbrowser-component.ts** to listen for the launch context:

```typescript
  constructor(
    private element: ElementRef,
    private http: Http,
    @Inject(Angular2InjectionTokens.LOGGER) private log: ZLUX.ComponentLogger,
    @Inject(Angular2InjectionTokens.PLUGIN_DEFINITION) private pluginDefinition: ZLUX.ContainerPluginDefinition,
    @Inject(Angular2InjectionTokens.WINDOW_ACTIONS) private windowAction: Angular2PluginWindowActions,
    @Inject(Angular2InjectionTokens.WINDOW_EVENTS) private windowEvents: Angular2PluginWindowEvents,
    //Now, if this is not null, we're provided with some context of what to do on launch.
    @Inject(Angular2InjectionTokens.LAUNCH_METADATA) private launchMetadata: any,
  ) {
    this.log.info(`User Browser constructor called`);

    //NOW: if provided with some startup context, act upon it... otherwise just load all.
    //Step: after making the grid... we add this to show that we can instruct an app to narrow its scope on open
    this.log.info(`Launch metadata provided=${JSON.stringify(launchMetadata)}`);
    if (launchMetadata != null && launchMetadata.data) {
    /* The message will always be an Object, but format can be specific. The format we are using here is in the Starter App:
      https://github.com/zowe/workshop-starter-app/blob/master/webClient/src/app/workshopstarter-component.ts#L177
    */
      switch (launchMetadata.data.type) {
      case 'load':
        if (launchMetadata.data.filter) {
          this.filter = launchMetadata.data.filter;
        }
        break;
      default:
        this.log.warn(`Unknown launchMetadata type`);
      }
    } else {
      this.log.info(`Skipping launching in a context due to missing or malformed launchMetadata object`);
    }
}
```

Then, add a new method on the Class, **provideZLUXDispatcherCallbacks**, which is a web-framework-independent way to allow the Zowe applications to register for event listening of Actions.

```typescript
  /*
  I expect a JSON here, but the format can be specific depending on the Action - see the Starter App to see the format that is sent for the Workshop:
  https://github.com/zowe/workshop-starter-app/blob/master/webClient/src/app/workshopstarter-component.ts#L225
  */
  zluxOnMessage(eventContext: any): Promise<any> {
    return new Promise((resolve,reject)=> {
      if (!eventContext || !eventContext.data) {
        return reject('Event context missing or malformed');
      }
      switch (eventContext.data.type) {
      case 'filter':
        let filterParms = eventContext.data.parameters;
        this.log.info(`Messaged to filter table by column=${filterParms.column}, value=${filterParms.value}`);

        for (let i = 0; i < this.columnMetaData.columnMetaData.length; i++) {
          if (this.columnMetaData.columnMetaData[i].columnIdentifier == filterParms.column) {
            //ensure it is a valid column
            this.rows = this.unfilteredRows.filter((row)=> {
              if (row[filterParms.column]===filterParms.value) {
                return true;
              } else {
                return false;
              }
            });
            break;
          }
        }
        resolve();
        break;
      default:
        reject('Event context missing or unknown data.type');
      };
    });
  }


  provideZLUXDispatcherCallbacks(): ZLUX.ApplicationCallbacks {
    return {
      onMessage: (eventContext: any): Promise<any> => {
        return this.zluxOnMessage(eventContext);
      }
    }
}
```

At this point, the application should build successfully and upon reloading the Zowe page in your browser, you should see that if you open the Starter application (the application with the green S), that clicking the **Find Users from Lookup Directory** button should open the User Browser application with a smaller, filtered list of employees rather than the unfiltered list we see if opening the application manually.

We can also see that once this application has been opened, the Starter application's button, **Filter Results to Those Nearby**, becomes enabled and we can click it to see the open User Browser application's listing become filtered even more, this time using the browsers [Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/Using_geolocation) to instruct the User Browser application to filter the list to those employees who are closest to you!

### Calling back to the Starter application

We're almost finished. The application can visualize data from a REST API, and can be instructed by other applications to filter that data according to the situation. But, to complete this tutorial, we need the application communication to go in the other direction - inform the Starter application which employees you have chosen in the table!

This time, we will edit **provideZLUXDispatcherCallbacks** to issue Actions rather than to listen for them. We need to target the Starter application, since it is the application that expects to receive a message about which employees should be assigned a task. If that application is given an employee listing that contains employees with the wrong job titles, the operation will be rejected as invalid, so we can ensure that we get the correct result through a combination of filtering and sending a subset of the filtered users back to the starter application.

Add a private instance variable to the **UserBrowserComponent** Class.

```typescript
 private submitSelectionAction: ZLUX.Action;
```

Then, create the Action template within the constructor

```typescript
this.submitSelectionAction = ZoweZLUX.dispatcher.makeAction(
  'org.openmainframe.zowe.workshop-user-browser.actions.submitselections',
  'Sorts user table in App which has it',
  ZoweZLUX.dispatcher.constants.ActionTargetMode.PluginFindAnyOrCreate,
  ZoweZLUX.dispatcher.constants.ActionType.Message,
  'org.openmainframe.zowe.workshop-starter',
  { data: { op: 'deref', source: 'event', path: ['data'] } }
)
```

So, we've made an Action which targets an open window of the Starter application, and provides it with an Object with a data attribute.
We'll populate this object for the message to send to the application by getting the results from Zowe Application Framework Grid (`this.selectedRows` will be populated from `this.onTableSelectionChange`).

For the final change to this file, add a new method to the Class:

```typescript
  submitSelectedUsers() {
    let plugin = ZoweZLUX.PluginManager.getPlugin("org.openmainframe.zowe.workshop-starter");
    if (!plugin) {
      this.log.warn(`Cannot request Workshop Starter App... It was not in the current environment!`);
      return;
    }

    ZoweZLUX.dispatcher.invokeAction(this.submitSelectionAction,
      {'data':{
         'type':'loadusers',
         'value':this.selectedRows
      }}
    );
}
```

And we'll invoke this through a button click action, which we will add into the Angular template, **userbrowser-component.html**, by changing the button tag for "Submit Selected Users" to:

```html
<button type="button" class="wide-button btn btn-default" (click)="submitSelectedUsers()" value="Send">
```

Check that the application builds successfully, and if so, you've built the application for the tutorial! Try it out:

1. Open the Starter application.
1. Click the "Find Users from Lookup Directory" button.
   1. You should see a filtered list of users in your user application.
1. Click the "Filter Results to Those Nearby" button on the Starter application.
   1. You should now see the list be filtered further to include only one geography.
1. Select some users to send back to the Starter application.
1. Click the "Submit Selected Users" button on the User Browser application.

   1. The Starter application should print a confirmation message that indicates success.

And that's it! Looking back at the beginning of this document, you should notice that we've covered all aspects of application building - REST APIs, persistent settings storage, Creating Angular applications and using Widgets within them, as well as having one application communicate with another. Hopefully you have learned a lot about application building from this experience, but if you have questions or want to learn more, please reach out to those in the Foundation so that we can assist.


# Zowe scenario 4: Add a new plug-in to Zowe CLI

## Overview
This senario demonstrates how to create a brand new Zowe CLI plug-in that uses Zowe CLI Node.js programmatic APIs.

At the end of this scenario, you will have created a data set diff utility plug-in for Zowe CLI, from which you can pipe
your plugin's output to a third-party utility for a side-by-side diff of data set member contents.

![Side by Side Diff](../../images/guides/CLI/htmlDiff.png)

Completed source for this tutorial can be found on the `develop-a-plugin` branch of the zowe-cli-sample-plugin repository.

### Cloning the sample plug-in source
 Clone the sample repo, delete the irrelevant source, and create a brand new plug-in. Follow these steps:

1. `cd` into your `zowe-tutorial` folder
2. `git clone https://github.com/zowe/zowe-cli-sample-plugin files-util`
3. `cd files-util`
4. Delete the `.git` (hidden) folder.
5. Delete all content within the `src/api`, `src/cli`, and `docs` folders.
6. Delete all content within the `__tests__/__system__/api`, `__tests__/__system__/cli`, `__tests__/api`, and `__tests__/cli` folders
7. `git init`
8. `git add .`
9. `git commit -m "initial"`

### Changing package.json
Use a unique `npm` name for your plugin. Change `package.json` name field as follows:

```typescript
  "name": "@brightside/files-util",
```

Issue the command `npm install` against the local repository.

### Adjusting Imperative CLI Framework configuration
Change `imperative.ts` to contain the following:
```typescript
import { IImperativeConfig } from "@brightside/imperative";

const config: IImperativeConfig = {
    commandModuleGlobs: ["**/cli/*/*.definition!(.d).*s"],
    rootCommandDescription: "Files utilty plugin for Zowe CLI",
    envVariablePrefix: "FILES_UTIL_PLUGIN",
    defaultHome: "~/.files_util_plugin",
    productDisplayName: "Files Util Plugin",
    name: "files-util"
};

export = config;
```
Here we adjusted the description and other fields in the `imperative` JSON configuration to be relevant to this plug-in.

### Adding third-party packages

We'll use the following packages to create a programmatic API:

- `npm install --save diff`
- `npm install -D @types/diff`


### Creating a Node.js programmatic API
In `files-util/src/api`, create a file named `DataSetDiff.ts`. The content of `DataSetDiff.ts` should be the following:
```typescript
import { AbstractSession } from "@brightside/imperative";
import { Download, IDownloadOptions, IZosFilesResponse } from "@brightside/core";
import * as diff from "diff";
import { readFileSync } from "fs";

export class DataSetDiff {

    public static async diff(session: AbstractSession, oldDataSet: string, newDataSet: string) {

        let error;
        let response: IZosFilesResponse;

        const options: IDownloadOptions = {
            extension: "dat",
        };

        try {
            response = await Download.dataSet(session, oldDataSet, options);
        } catch (err) {
            error = "oldDataSet: " + err;
            throw error;
        }

        try {
            response = await Download.dataSet(session, newDataSet, options);
        } catch (err) {
            error = "newDataSet: " + err;
            throw error;
        }

        const regex = /\.|\(/gi; // Replace . and ( with /
        const regex2 = /\)/gi;   // Replace ) with .

        // convert the old data set name to use as a path/file
        let file = oldDataSet.replace(regex, "/");
        file = file.replace(regex2, ".") + "dat";
        // Load the downloaded contents of 'oldDataSet'
        const oldContent = readFileSync(`${file}`).toString();

        // convert the new data set name to use as a path/file
        file = newDataSet.replace(regex, "/");
        file = file.replace(regex2, ".") + "dat";
        // Load the downloaded contents of 'oldDataSet'
        const newContent = readFileSync(`${file}`).toString();

        return diff.createTwoFilesPatch(oldDataSet, newDataSet, oldContent, newContent, "Old", "New");
    }
}
```

### Exporting your API
In `files-util/src`, change `index.ts` to contain the following:
```typescript
export * from "./api/DataSetDiff";
```

## Checkpoint
At this point, you should be able to rebuild the plug-in without errors via `npm run build`. You included third party dependencies, created a programmatic API, and customized this new plug-in project. Next, you'll define the command to invoke your programmatic API.

### Defining commands
In `files-util/src/cli`, create a folder named `diff`. Within the `diff` folder, create a file `Diff.definition.ts`. Its content should be as follows:
```typescript
import { ICommandDefinition } from "@brightside/imperative";
import { DataSetsDefinition } from "./data-sets/DataSets.definition";
const IssueDefinition: ICommandDefinition = {
    name: "diff",
    summary: "Diff two data sets content",
    description: "Uses open source diff packages to diff two data sets content",
    type: "group",
    children: [DataSetsDefinition]
};

export = IssueDefinition;
```

Also within the `diff` folder, create a folder named `data-sets`. Within the `data-sets` folder create `DataSets.definition.ts` and `DataSets.handler.ts`.

`DataSets.definition.ts` should contain:
```typescript
import { ICommandDefinition } from "@brightside/imperative";

export const DataSetsDefinition: ICommandDefinition = {
    name: "data-sets",
    aliases: ["ds"],
    summary: "data sets to diff",
    description: "diff the first data set with the second",
    type: "command",
    handler: __dirname + "/DataSets.handler",
    positionals: [
        {
            name: "oldDataSet",
            description: "The old data set",
            type: "string"
        },
        {
            name: "newDataSet",
            description: "The new data set",
            type: "string"
        }
    ],
    profile: {
        required: ["zosmf"]
    }
};
```

`DataSets.handler.ts` should contain the following:
```typescript
import { ICommandHandler, IHandlerParameters, TextUtils, Session } from "@brightside/imperative";
import { DataSetDiff } from "../../../api/DataSetDiff";

export default class DataSetsDiffHandler implements ICommandHandler {
    public async process(params: IHandlerParameters): Promise<void> {

        const profile = params.profiles.get("zosmf");
        const session = new Session({
            type: "basic",
            hostname: profile.host,
            port: profile.port,
            user: profile.user,
            password: profile.pass,
            base64EncodedAuth: profile.auth,
            rejectUnauthorized: profile.rejectUnauthorized,
        });
        const resp = await DataSetDiff.diff(session, params.arguments.oldDataSet, params.arguments.newDataSet);
        params.response.console.log(resp);
    }
}

```

## Trying your command
Be sure to build your plug-in via `npm run build`.

Install your plug-in into Zowe CLI via `zowe plugins install`.

Issue the following command. Replace the data set names with valid mainframe data set names on your system:

![Pipe Output](../../images/guides/CLI/pipeOutput.png)

The raw diff output is displayed as a command response:

![Raw Diff Output](../../images/guides/CLI/diffOutput.png)

## Bringing together new tools!
The advantage of Zowe CLI and of the CLI approach in mainframe development is that it allows for combining different developer tools for new and interesting uses.

[diff2html](https://diff2html.xyz/) is a free tool to generate HTML side-by-side diffs to help see actual differences in diff output.

Install the `diff2html` CLI via `npm install -g diff2html-cli`. Then, pipe your Zowe CL plugin's output into `diff2html` to generate diff HTML and launch a web browser that contains the content in the screen shot at the [top of this file](#overview).

- `zowe files-util diff data-sets "kelda16.work.jcl(iefbr14)" "kelda16.work.jcl(iefbr15)" | diff2html -i stdin`

## Next steps
Try the [Implementing profiles in a plug-in](cli-implement-profiles.md) tutorial to learn about using profiles with your plug-in.