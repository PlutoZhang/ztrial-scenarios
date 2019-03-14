# Scenario 2: Extending Zowe

## Overview 

In this scenario, you will learn how to extend Zowe to add your own API or application. This scenario guides you through the steps in roughly 30 minutes. By the end of the session, you'll know how to:

- Extend the API by adding a REST API to the API Mediation Layer
- Extend Zowe Web UI by creating and deploying an application plug-in on Zowe Desktop
- Extend Zowe CLI by creating a Zowe CLI plug-in to access the API

No previous knowledge of Zowe or API design is needed but some knowledge of API and Command Line will be helpful. Please wait a moment while your development environment loads (this takes a minute or so). When it loads, get started by extending the API.

## Step 1: Extend the Zowe API

In this step, you will add some missing code to expose an API in API Medidation Layer and then access your API service endpoints to ensure that it works. 

The sample API used in this step is a Node.js API for finding cars and accounts for a dealership. This API which has some missing features will be running in the API Catalog of the API Mediation Layer. You will view the current API, add the missing code for the feature, redeploy it and then test that the API service endpoint works.

<!--Requirements on the client system: VSCode, npm-->

### **Procedure**

1. Visit the Swagger doc. 

    >**Questions:** Is it the doc in the API Catalog? How can users access that, by logging in to the API Catalog and click the API Catalog app? Will this Node.js API be prebuilt in zTrial so users can access? Which Swagger doc should users open?

1. Open Firefox and enter the following URL in the address field.  
    >**Questions:** What's the purpose of doing this? To test if the endpoint works?

    ```http://localhost:3000/accounts/1/cars```

    The following error message is displayed, which indicates that the API edpoint is not working. 

    <img src="./images/scenario2-api-test-error.png" width="200">

1. Open Visual Studio Code from the desktop. 
    >**Questions:** How can users open VSCode in the zTrial Windows image? Think that will need to be preinstalled. We need to clarify the path to access VSCode in zTrial here.

1. Open the terminal by using the **View** > **Terminal** menu command. The terminal is opened at the bottom of the editor window.
1. Enter `npm start`.
1. Enter `npm test`. You will see that one test fails. 
    
    <img src="./images/scenario2-api-test-fail.png" width="200">

    This is because there are missing code in the configuration file. Next, let's locate the file to add the missing code.
    
1. Open the Explorer tab of VSCode and then click **SAMPLE-NODE-API** > **server** > **routes** > **accountsCars.route.js**. The contents of the **accountsCars.route.js** file is displayed. 

    <img src="./images/scenario2-api-folder-locate.png" width="200">

    You will see that the code for a feature is missing in this file.

    <img src="./images/scenario2-missing-code-file.png" width="300">

1. Insert the following code to the file and press `Ctrl+S` to save the changes. 
    >**Questions:** Where should users find the missing code? Currently we just put it here so users can copy and paste. This might also be the quickest way to get the code.

    ```
    router.route('/cars')
    .get(accountsCarsController.getAll);

    router.route('/cars/:_id')
    .get(accountsCarsController.get);
    ```
    <img src="./images/scenario2-missing-code-insert.png" width="300">

1. Redeploy the API. 
    >**Questions:** How to do this? Need more clarification here.

1. In the TERMINAL panel, enter the `npm start` command. 
1. Open Firefox again and enter the following URL in the address field.

    ```http://localhost:3000/accounts/1/cars```

    You should get the following response, which indicates that you can access the API endpoints.

    <img src="./images/scenario2-api-test-success.png" width="350">

    >**Questions:** Should users also go to API Catalog to verify that it's working?

### **Next step**


## Step 2: Creating and deploying an application on Zowe Desktop

You will complete a sample application and deploy the application on Zowe Desktop. 

In this step, you will combine some provided code snippets with the skeleton code to build a complete application that displays on the Zowe Desktop.

A react app which hooks up to the API will be available but won't have the new endpoint working yet. Now the API has been updated we can find the code block using the MVS explorer and then redeploying it.

<!--Requirements on the client system: -->

### **Procedure**


**Next step**


## Step 3: Extending a Zowe CLI plug-in

You will extend an existing Zowe CLI plug-in by introducing the Node.js programmatic API in scenario 1, and create a command definition with a handler. 

<!--Requirements on the client system: -->

**Procedure**

1. Defining command syntax
1. Defining command handler
1.
1.

**Next step**