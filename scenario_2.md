# Scenario 2: Extending Zowe

1. [Overview](#overview)
2. [Extend the Zowe API](#step-1-extend-the-zowe-api)
3. [Extending Zowe Application Framework](#step-2-extending-zowe-application-framework)
4. [Extending Zowe CLI](#step-3-extending-zowe-cli)
5. [Next Steps](#next-steps)
6. [Go deeper with Zowe](#go-deeper-with-zowe)

## Overview

In this scenario, you will learn how to extend Zowe by adding new features in the sample Zowe API and applications and deploying the change. This scenario guides you through the steps in roughly 30 minutes. By the end of the session, you'll know how to:

- Extend the API by adding a new feature to a REST API in the API Mediation Layer
- Extend Zowe Application Framework by adding a new feature to an application plug-in on the Zowe Desktop
- Extend Zowe CLI by creating a Zowe CLI plug-in to access the API

No previous knowledge of Zowe is needed but some knowledge of API and command line will be helpful. Please wait a moment while your development environment loads (this takes a minute or so). When it loads, get started by extending the API.

## Step 1: Extending the Zowe API

In this step, you will add a new feature in an API in the Zowe API Medidation Layer and then access your API service endpoints to ensure that it works.

The sample API used in this step is a Node.js API for finding cars and accounts for a dealership. This API will be running in the API Catalog of the API Mediation Layer. You will view the current API, add the code for a new feature, redeploy the API, and then test that the API service endpoint works.

### Procedure

1. Develop custom API.
    1. Open the sample API project in Visual Studio Code (VSCode).
       1. Click on the Files Explorer icon in the taskbar to open the Files Explorer, and go to this folder `C:\Users\Administrator\Documents\zowe-trial-scenario-2`. 
       2. Right click the folder named sample-node-api and select **Open with Code**.
       
           <img src="./images/scenario2-sample-open.png" width="300">
    1. Run the sample API project in the VSCode terminal.
       1. In VSCode, from the Menu Bar, open the terminal by using the **View** > **Terminal** menu command.
          
          <img src="./images/scenario2-vscode-terminal.png" width="300">

          Below the editor region, the terminal panel is started in the current working directory.
       1. In the termianl panel, change the current working directory to _sample-node-api_ by issuing the following command in the terminal panel:
           ```
           cd sample-node-api
           ```
       2. Issue the `npm install` command and press Enter to install the sample application.

           <img src="./images/scenario2-npm-install.png" width="600">
       3. Issue the `npm start` command and press Enter to start the sample application on the node server. 

          <img src="./images/scenario2-npm-start.png" width="600">
    1. Access local URLs.
       1. From the Taskbar in the Desktop, click the Firefox icon to open Firefox.
       1. Enter the following URL in the address field to get the information about all accounts.
          `https://localhost:18000/accounts`
          
           All the accounts 0, 1, 2, 3, and 4 are displayed. 
       
           <img src="./images/scenario2-api-test-accounts.png" width="500">
           
       1. To get the detailed information about one specific account, for example, account 0, enter the following URL in the address field.
          `https://localhost:18000/accounts/0'
          
           <img src="./images/scenario2-api-test-account.png" width="500">
           
       1. To get the information about all the cars owned by one specific account, for example, by account 0, enter the following URL in the address field.
          `https://localhost:18000/accounts/0/cars`
          The following error message is displayed, which indicates that this API endpoint is not working.
           
           <img src="./images/scenario2-api-test-cars-fail.png" width="500">
           
    1. Back to the terminal panel in VSCode, press `Ctrl+c` to stop the running application.  
    1. Enter `npm test` in the VSCode terminal. You will see that three tests fail.

        <img src="./images/scenario2-api-test-fail.png" width="200">
        
       This is caused by a piece of missing code in the configuration file. Next, let's locate the file to add the missing code.

    1. Add the missing feature into the sample API node project.
       1. Open the Explorer tab of VSCode and then click **SAMPLE-NODE-API** > **server** > **routes** > **accountsCars.route.js**. The contents of the **accountsCars.route.js** file is displayed.
       
            <img src="./images/scenario2-api-folder-locate.png" width="300">

            You will see that the code for two routes is commented out in command line 11 - 15.

            <img src="./images/scenario2-missing-code-file.png" width="500">

        2. Uncomment this code snippet by removing the **/*** and ***/** signs at the beginning and the end. Then, press Ctrl+S to save the change.
        
            <img src="./images/scenario2-missing-code-insert.png" width="500">

       3. Restart the project in the terminal.
          1. Press `ctrl+c` in the terminal to stop the running project.
          1. Run the `npm test` command to check that the failed tests are fixed.
          
              <img src="./images/scenario2-server-npm-test.png" width="500">
              
              You will see that all the tests are passed. 
          1. Run the `npm start` command to restart the project.

     1. Access the newly added API routes in the Firefox browser.

        Open Firefox and enter the following URL in the address field to get the information about all the cars owned by account 0.
           `https://localhost:18000/accounts/0/cars`

          <img src="./images/scenario2-api-working.png" width="300">
          
          You can see that account 0 owns two cars.
          
          To get the information about one specific car, for example, the car with id 1, enter the URL `https://localhost:18000/accounts/0/cars/1`.  

         <img src="./images/scenario2-api-car.png" width="500">

   Now you succefully add the missing feature locally. Next, you'll redeploy this API and see the changes on the Zowe server.

1. Redeploy this API to the Zowe server and view the changes.
    1. Open Firefox and access the same API running on the zowe server.

       Open the Firefox browser and enter the following URL in the address field to get the information about all the cars owned by account 0.
       
       `https://10.149.60.146:7554/api/v1/sample-node-api/accounts/0/cars`

         <img src="./images/scenario2-server-api-not-working.png" width="500">

        You can see that the new routes you added locally are not deployed to the Zowe server yet.

    1. Redeploy the updated sample API node files to the Zowe server.
       1. In the VSCode terminal panel, press `ctrl+c` to stop the running project.
       2. Enter the following command to upload the updated sample files to the Zowe server.
         `scp -P 2022 -r server tstradm@10.149.60.146::/zaas1/zowe/1.0.0/sample-node-api`
         
         <img src="./images/scenario2-server-api-update.png" width="500">
         
       3. Enter password **TSTRADM**.
    1. Restart the sample API on the Zowe server.
       1. Enter the following ssh command and press Enter to log in to the Zowe server.  
          
          `ssh -p 2022 tstradm@10.149.60.146`
      
           <img src="./images/scenario2-server-login.png" width="500">
       
        1. Enter password **TSTRADM**.
        1. Go to the sample application scripts folder by running the following command.
        
           `cd /zaas1/zowe/1.0.0/sample-node-api/scripts`
           
        1. Restart the sample application scripts by running the following command.
        
           `Restart-sample-node-api.sh`
           
           Wait for about one minute for the process to complete. When you see the following command prompt, the sample API on the Zowe server is restarted. 
           
           <img src="./images/scenario2-server-api-restarted.png" width="400">
           
     1. Access the redeployed API again in the Firefox browser. Enter the following URL in the address field:
        `https://10.149.60.146:7554/api/v1/sample-node-api/cars`
         
         <img src="./images/scenario2-server-cars-correct.png" width="400">
        
        It works now and you can see the information about all cars as the same as you see locally. You can aslo try this URL `https://10.149.60.146:7554/api/v1/sample-node-api/accounts/0/cars/1` to get the information about car 1 owned by account 0.
        
        <img src="./images/scenario2-server-car-correct.png" width="400">

### Results
You successfully added the missing feature to the sample node API and redeployed to the Zowe server.

### Next step
In the next step, a sample application that uses this sample API is deployed on the Zowe Desktop. Similar to what you did in this step, you will add some missing features to make that application work to get experience with Zowe Web UI development.

## Step 2: Extending Zowe Application Framework

In this step, you will add some provided code snippets with the existing code to build a Trial Sample application that works fully on the Zowe Desktop.

### Procedure

1. Build and test the Trial Sample application in Zowe Desktop.

    1. Open the sample Zowe Application Framework project in VS Code.

        1. Click on the File Explorer icon in the Windows taskbar to open the Windows Explorer.

           <img src="./images/scenario2-open-windows-explorer.png" width="100">

        1.  Go to folder `C:\Users\Administrator\Documents\zowe-trial-scenario-2` which contains the source code for the sample project.

         <img src="./images/scenario2-ui-locate-folder.png" width="400">

        1. Right click the folder named **part-02-UI** and select **Open with Code** to open the folder in VS Code.
           
           <img src="./images/scenario2-zlux-open.png" width="200">

    1. Run the sample project in the VS Code terminal.

        1. In VS Code, from the Menu bar, click **View** > **Terminal** to open the terminal.

           <img src="./images/scenario2-vscode-terminal.png" width="200">   

           Below the editor region, the terminal panel is started in the current working directory.

           <img src="./images/scenario2-ui-terminal-opened.png" width="500">

        1. Change the current working directory to _webClient_ by issuing the following command in the terminal panel and press Enter:
           ```
           cd zlux/sample-trial-app/webClient
           ```
           <img src="./images/scenario2-zlux-webclient.png" width="500">

        1. Issue the `npm install` command and press Enter to install the sample project. Wait for about one minute for the process to complete.

           <img src="./images/scenario2-zlux-npm-install.png" width="800"> 
       
           When complete, you will see messages similar to the following ones. No action is required here.
       
           <img src="./images/scenario2-ui-install-complete.png" width="800">

       1. Enter the `npm run build` command to run the build. The build completes within seconds. 

          <img src="./images/scenario2-npm-run-build.png" width="500">

          A folder named _web_ is created in the root directory _sample-trial-app_.
       
          <img src="./images/scenario2-zlux-web.png" width="150">

1. Transfer the files from the _web_ folder to the Trial Application folder on the Zowe server. 
   1. Enter the following command:

      ```
      scp -P 2022 -r ../web tstradm@10.149.60.146:/zaas1/zowe/1.0.0/sample-trial-app
      ```
            
      <img src="./images/scenario2-zlux-scp.png" width="500">

   2. Enter the password. The password is **tstradm**.

      The files _icon.png_, _mian.js_, and _main.js.map_ are securely copied to the folder _sample-trial-app_ on the Zowe server.

      <img src="./images/scenario2-enter-password.png" width="500">

1. Open and test the Trial Sample application in the Zowe Desktop.

    1. Start Firefox and enter the following URL to access the Zowe Desktop in the address field.
        ```https://10.149.60.146:8544/ZLUX/plugins/org.zowe.zlux.bootstrap/web/index.html```
    1. Enter the following username and password to log in.
       - User name: **TSTRADM**
       - Password: **TSTRADM**

       The Zowe Desktop opens.

    1. In the Zowe desktop, click the Start menu and locate the _Trial Sample_ application. Right-click on the **Trial Sample** application and select **Pin to taskbar** for later use.
        
        <img src="./images/scenario2-zlux-sample-icon.png" width="200">
    1. Click to open the **Trial Sample** application from the taskbar.

       In this application, click **Accounts** and you will see that the values of the **Name** column are missing. This is because of some missing values in the configuration file of this application.

       <img src="./images/scenario2-zlux-sample-missing.png" width="500">

1. Add the missing code snippet and redeploy the changes.
    1. Uncomment the missing code snippet in the _Accountlist.js_ file.
        1. In VS Code Explorer, click **zlux** > **sample-trial-app** > **webClient** > **src** > **Accountlist.js**. This file _Accountlist.js_ contains the missing values.

           <img src="./images/scenario2-ui-config-file-locate.png" width="200">

           You will see that the code for the user name details is commented out in command line 77 - 79.
           <img src="./images/scenario2-zlux-comment-out.png" width="600">

        1. Uncomment this code snippet by removing the **/*** and ***/** signs at the beginning and the end. Then, press **Ctrl+S** to save the change.
           
           <img src="./images/scenario2-zlux-uncomment.png" width="600">
           
    1. Enter `npm run build` to run the build.
       The folder named _web_ is updated in the root directory _sample-trial-app_.
    1. Transfer the updated files from the _web_ folder to the Trial Application folder on the Zowe server.
        1. Enter the following command:
           ```
           scp -P 2022 -r ../web tstradm@10.149.60.146:/zaas1/zowe/1.0.0/sample-trial-app
           ```
        2. Enter the password. The password is **tstradm**.
           The files _icon.png_, _mian.js_, and _main.js.map_ are securely copied to the folder _sample-trial-app_ on the Zowe server.

1. Verify that the Trial Sample application works correctly now. 
    1. In the Firefox browser, press `F5` to refresh the Zowe Desktop page.
       A pop-up box is opened to ask for your confirmation to leave. Click **Leave Page** to refresh the Zowe Desktop. When prompted for the password to log in to Zowe Desktop, enter **TSTRADM**.
     
       <img src="./images/scenario2-zlux-leave-page.png" width="400">

    1. In the Zowe Desktop, click to reopen the **Trial Sample** application from the taskbar.
    1. In this application, click **Accounts** and you will see that the values of the **Name** column are displayed.
     
        <img src="./images/scenario2-zlux-sample-display.png" width="500">

        You can also click on any name to get its detailed information.
     
        <img src="./images/scenario2-zlux-sample-name-detail.png" width="500">

### Result
Congratulations! You added the missing values to the Trial Sample application, deployed the changes, and verified that this application works correctly.

### Next step

In the next step, You will work on a Zowe CLI plug-in based on the same Node.js API.

## Step 3: Extending Zowe CLI

You will extend an existing Zowe CLI plug-in by introducing the Node.js programmatic API in scenario 1.

<!--Requirements on the client system: -->

### **Procedure**
1. Open Visual Studio Code from the desktop.
1. From the **Menu Bar**, open the terminal by using the **View** > **Terminal** menu command.  
    Below the editor region, the terminal panel is started in the current working directory, **ZTRIAL-CLI**.
1. Run the following CLI command to check whether the data set _average-horse-power_ can be accessed.  
    `zowe ztrial-plugin cars average-horse-power`  
    You'll see that a command syntax error is prompted.
1. Enter `npm test` to execute the tests. You will get the following output, which says that one test fails.  
    <img src="./images/scenario2-cli-test-fail.png" width="300">  
    This is because there are missing codes in the configuration file. Next, let's locate the file to add the missing codes.
1. From the Workspace **ZTRIAL-CLI**, click **src** > **api** > **Car.ts (temp name)** to open the **Car.ts** typescript file. The contents of the **Car.ts** is displayed in the editor region.   
    You will see that the code for a feature is missing in this file.
1. Download the missing code and insert to the **Car.ts** file.
    1. To get the missing code snippet, use the Zowe CLI plugin to download. In the terminal panel, enter the following command:  
         >**To be done:the command to be added`  

        The dataset is successfully downloaded.

    2. View the content of the dataset and find the missing code block, and then insert it into the **Car.ts** file and press `Ctrl+S` to save the changes.
1. Redeploy the API.
    1. In the terminal panel, enter `npm run build` to build a new package.
    >**To be done:** more details.
    2. Enter `zowe plugins install ./` to install the plug-in.
    >**To be done:** the name of the plug-in.  
1. Verify that this API works correctly now.
    1. In the terminal panel, run the following CLI command to check whether the data set _average-horse-power_ can be accessed:  
    `zowe ztrial-plugin cars average-horse-power-for-account 4`  
    You'll see an error-free result with no command syntax error.
    2. Enter `npm test` to execute the tests. You will see the following all-pass output.  
        >**To be done:** more details and screenshots to be added.

### **Results**  

Congratulations! You added the missing feature of the API, redeployed the API, and verified that the CLI plug-in works correctly with this API.

# Next Steps
Thanks for your time in exploring the Zowe scenarios!
# Go deeper with Zowe
Zowe is an open source project that is created to host technologies that benefit the Z platform from all members of the Z community, including Integrated Software Vendors, System Integrators, and z/OS consumers.

Zowe, like Mac or Windows, comes with a set of APIs and OS capabilities that applications build on and also includes some applications out of the box.

If you have any interest, visit Zowe (https://zowe.github.io) on Open Mainframe Project to learn more about the capabilities of Zowe and the value it delivers.
