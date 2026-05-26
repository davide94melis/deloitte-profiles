# PNRR-incentive-manager

This README pnrr incentive-manager would normally document whatever steps are necessary to get your application up and
running.

## Local development ##

In order to run the application on your local machine, follow these steps:

1. Create an "application-local.yml" file in ./src/main/resources folder.

2. Copy the content of the property you want override from ./src/main/resources/config/application.yml and paste it into "application-local.yml" file.

3. Create a new local database called "pnrr".

4. Set the spring active profile in Intellij:
   Go to "Edit-Configuration" and put into "Environment variables" --> SPRING_PROFILES_ACTIVE=local.

5. Before each run it is necessary to build the project with: mvn clean install This is necessary because the MapStruct
   library will auto-generate in compiletime the implementation of the mapper interfaces.
   (If there is no differences on Jpa and DTOs you can skip this step after first time)




## Crypt/Decrypt feature on Local development ##

In order to use the Crypt/Decrypt feature on your local development fill your application-local.yaml file with the
follow configuration:

```javascript
security:
    aes:
        secret-key: "9T58p92CKQS4qvhU3AEHFKCxPRLwsyQv"
        init-vector: "M5GrAiEgXwQR3bni"
```


## Creating secured APIs (@Controller) ##

In this project we have many kind of APIs: 

1 Secured API with Authentication and Authorization strategies.

2 Simple Public API for every user, without the authentication requirements.

For Public API we only need to give special path starting with "/api/public/...". With this strategy we ensure the API is public thanks to applied configuration in security-manager project.

For Private API we need only avoid the above path and use "/api/..". With this strategy we ensure the API is called only from authorized (logged) users.
If the Private API needs a special Role/Authorization or if we only want to ensure the api is called only from one type of role we need to follow the annotation strategy of spring-security: @RolesAllowed

## Logging best practices ##

Everyone will contribute the project have to follow these main guidelines for logging framework:


**INFORMATIVE LOGS**: At least for each method of @Service class we have to log information when we ENTER end EXIT of method with details about the request/response object.

```javascript
log.info("Entering method registrate user with request username {}", request.getUserame());
```

**ERROR LOGS**: Just before throw a custom exception is a best practice log information about the error. We don't need log the stack trace because the GlobalExceptionHandler will do for us.


```javascript
log.error("Error request the resource with id {} from username {}",request.id, request.getUserame());
throw new MyCustomException("Detailed message");
```

**WARNING LOGS**: Occasionally we need log a warning for a strange situation that should not appear, but it's not an error case.

```javascript
log.warn("Something is wrong from username {}",request.getUserame());
```

At the end, just remember to avoid logs of sensitive information for example the password or secret. 

For more information you can read the follow article: https://sematext.com/blog/java-logging-best-practices/


## JPA best practices ##

Avoid problem with DTO/Entity Mapping and "mappedBy" Relationship.
From an Entity view in a lot of cases we don't need the relationship called "mappedBy".
From a DTO view in a lot of cases we don't need (and we should avoid)the object that map above relationship.

If we add both mappedBy relationship in the entity and correlated field in the DTO class we will incur in large amount of automatic queries,
for retrieve data that maybe we don't neither need in the FE page.

In order to solve these problems we can adapt two strategy:

1: Just don't insert the "mappedBy" relationship if we don't need that (in most of the cases we will not need it)

2: Add the mappedBy relationship but use a specific DTO RESPONSE that can map or not this relationship (in most of the cases we will not need it)

Also, I was remembered you as best practice and as normal strategy of querying data, if we want some data derived starting from a JOIN relationship,
we should query the main owner of the relationship (owner of foreign key).
Another best practice to avoid overload of network and useless data on the FE(and the jpa queries on the BE),
is the use of ad HOC ResponseDTO for each dashboard PAGE(we can use "response" package on the BE).

## Work with DTO and [Mapstruct](https://mapstruct.org/) ##

Please use DTO only for return the data already processed or for Specific Request Data.
Avoid doing business logic on a DTO class when you can do the same thing with less logic using the entity class.
Use the Mapper only in the @Service class and possibly at the end of the method.
Remember that you can create multiple Dto for more http requests and response, as well as mapper interfaces.

## Use [Lombok](https://projectlombok.org/). ##

Check out the demo video on the website and see what it can do for you.

## A nice to have IntelliJ Extension: SonarLint ##

SonarLint reports issues on the files you're editing.
No configuration required, just install SonarLint from your IDE marketplace and continue your coding.
SonarLint precisely pinpoints where the problem is, and gives you recommendations on how to fix it.
1. Open IntelliJ -> File -> Settings -> Plugins -> Search and Install "SonarLint"

2. Before commit each modified file, check quality using SonarLint Button beneath IntelliJ.

3. Click "Run" button, check and fix the suggestion.

## Configure Cribis Client avoid SSL problem: ##

### Passw config ###
For password field you can use 'Deloitte01!' . 
Just insert it in your application-local yaml as: 
```javascript
application:
    cribis:
        password: "Deloitte01!"
```


### Avoid SSL problem: ###
Add the cer file "cribis_cer.cer" to your jvm using the following guide:
https://stackoverflow.com/questions/21076179/pkix-path-building-failed-and-unable-to-find-valid-certification-path-to-requ

keytool -import -alias cribis_cer -keystore  "C:\jdk-11.0.10\lib\security\cacerts" -file cribis_cer.cer

## Other best practices: ##

### @Controller and @Service class ###
We should follow a well-defined standard when we need some new feature in the project.
In most cases, if there is no @Controller class for our feature/domain we can create it. 
Avoid business logic in controller methods and use the related @Service to achieve that.
it's suggested have java method of @Service that match the same name method into @Controller class.

### Work with Dates ###
In order to get a good standard when working with dates, we always use LocalDate or LocalDatetime (also if we need a request param in a Controller class).

### Java Naming convention for method,variables and others ###
https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html