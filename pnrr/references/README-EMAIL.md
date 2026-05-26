# PNRR-email-manager

This README pnrr email-manager would normally document whatever steps are necessary to get your application up and
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

## Send Email on Local development using Aws SES ##

In order to use to Send email feature on your local development and quickly start the project you can use sample account SES AWS of Dev enviroment (ISP).
Fill your application-local.yaml file with the follow configuration:

```javascript
spring:
    mail:
        host: "email-smtp.eu-west-1.amazonaws.com"
        port: 587
        username: "AKIARLAYPWWL2KAE74OA"
        password: "BMNXOZUUMTF9eYkbonMxL0Hx5mo9wCQmqKPV3CKjUVHd"
```
