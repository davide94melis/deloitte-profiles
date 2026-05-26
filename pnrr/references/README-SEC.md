# pnrr-security-manager

This README pnrr security manager would normally document whatever steps are necessary to get your application up and
running.

## Local development ##

In order to run the application on your local machine, follow these steps:

1. Create an "application-local.yml" file in ./src/main/resources folder.

2. Copy the content of the property you want override from ./src/main/resources/config/application.yml and paste it into "application-local.yml" file.

3. Set the spring active profile in Intellij:
   Go to "Edit-Configuration" and put into "Environment variables" --> SPRING_PROFILES_ACTIVE=local.

4. Before each run it is necessary to build the project with: mvn clean install This is necessary because the MapStruct
   library will auto-generate in compiletime the implementation of the mapper interfaces.
   (If there is no differences on Jpa and DTOs you can skip this step after first time)

## CORS Configuration ##

In order to configure cors in each environment (local,dev or production) you must fill the orgin_path field of customer table.
For example if you are in local env, you should have a row in the customer table with the origin path: http://locahost:4200. 
Please include the prefix in the value.

A http request will be done by security manager to incentive manager in order to get customer origin paths.
Please configure the application-local.yaml file with the following
```javascript

application:
    incentive-service:
        base-path: http://localhost:<INCENTIVEMANAGER_LOCAL_PORT>
```

## Crypt/Decrypt feature on Local development ##

In order to use the Crypt/Decrypt feature on your local development fill your application-local.yaml file with the
follow configuration:

```javascript
security:
    aes:
        secret-key: "9T58p92CKQS4qvhU3AEHFKCxPRLwsyQv"
        init-vector: "M5GrAiEgXwQR3bni"
```

## Sample User Data  ##

Below some encrypted data to insert in your local database for first login on Local development without using registration flow.

Sample passw:
Sample123Del!oitte  ->  $2a$10$U5pB/14bc5Lbb3NagE9fbuq2dJNN92/6SVr9jd0X//qCHSYh3IPDO

Sample secret:
SEBV7JII6VYZFUL7B2YREARVLNJX3UDI  ->  YVHUTCd3ZTi6YeKobV1f0RdLsfZimt+hqLED2MgPEabtY4MuF56OSyfraDv1Kw4b