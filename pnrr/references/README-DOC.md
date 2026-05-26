# PNRR-document-manager
Manage the document storage flow for PNRR app

## Local development ##

In order to run the application on your local machine, follow these steps:

1. Create an "application-local.yml" file in ./src/main/resources folder.

2. Copy the content of the property you want override from ./src/main/resources/config/application.yml and paste it into "application-local.yml" file.

3. Create a new local database downloading it from mongo database official site

4. Set the spring active profile in Intellij:
   Go to "Edit-Configuration" and put into "Environment variables" --> SPRING_PROFILES_ACTIVE=local.




## Connecting to Local MongoDB ##
You can download MongoDatabase from official site, install also mongoDB Compass as client for the database.

You have to use your application-local.yaml with your local configuration
Most of the time you don't need credentials part if you are in local mongo database :

```javascript
spring:
    data:
        mongodb:
            uri: "mongodb://my-user:my-passw@localhost:27017/my-db?retryWrites=false"
```



## Connecting to Dev MongoDB ##
If you have problem connecting local mongo database, you can use DEV ISP mongo db.
Copy and paste in your application-local.yaml the follow configuration:

```javascript
spring:
    data:
        mongodb:
            uri: "mongodb://pnrr-dev:zsXzTLtCmkMWjCF@localhost:27017/PNRRDev?retryWrites=false"
```

You need also a port forwarding using the jenkins ec2 machine, so you can connect to jenkins machine with Bitvise (credentials in excel file ISP DEV) and configure the "C2S" panel with:
Destination host: 

```javascript
"mongo-cluster-pnrr-dev-ecobonus-pnrr.cluster-c9k0gsiof0tp.eu-west-1.docdb.amazonaws.com"
```
