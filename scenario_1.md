# Scenario 1: Getting started with Zowe

1. [Overview](#overview)
2. [Logging in to the Zowe Desktop](#logging-in-to-the-zowe-desktop)
3. [Querying JES jobs and viewing related status in JES Explorer](#querying-jes-jobs-and-viewing-related-status-in-jes-explorer)
4. [Using TN3270 in Zowe Desktop to view the job](#using-tn3270-in-zowe-desktop-to-view-the-job)
5. [Editing a data set in MVS Explorer](#editing-a-data-set-in-mvs-explorer)
6. [Using the Zowe CLI to edit a data set](#using-the-zowe-cli-to-edit-a-data-set)
7. [View the data set changes in MVS Explorer](#view-the-data-set-changes-in-mvs-explorer)
8. [Next Steps](#next-steps)
    - [Go deeper with Zowe](#go-deeper-with-zowe)
	- [Try the Extending Zowe scenario](#try-the-extending-zowe-scenario)

## Overview

Zowe offers modern interfaces that enables you to interact with z/OS in a way that is similar to what you experience on cloud platforms today. Like Mac or Windows, Zowe comes with a set of APIs and OS capabilities that applications build on and includes some applications out of the box.

<img src="./images/zowe-architecture.png" width="400">

This scenario walks you through the Zowe interfaces including the Zowe Desktop and Zowe CLI through several simple tasks to help you get familiar with Zowe.
- If you are new to Zowe, start with this scenario to explore the base Zowe features and functions.
- If you are already familiar with Zowe interfaces and capabilities, you can skip this scenario and directly go to the Extending Zowe scenario which guides you to extend Zowe by creating your own APIs and applications.

This scenario guides you through the steps in roughly 30 minutes. By the end of the session, you'll know how to:
- Log in to the Zowe Desktop
- View and edit data sets by using the MVS Explorer
- Query jobs with filters and view the related status by using the JES Explorer
- View jobs by using TN3270 in the Zowe Desktop
- Edit data sets and upload to mainframe by using Zowe Command Line Interface (CLI)

<!--If you haven't started the scenario, complete the following steps:
1. Double-click Z Trial Wizard.exe on the desktop.
1. In the Z Trial Program window, select the scenario that you want and click **Explore scenario**.
1. In the pop-up window, click **Start scenario**.-->

As an introductory scenario, no previous knowledge of Zowe is needed. Please wait a moment while your development environment loads (this takes a minute or so). When it loads, get started by logging in to the Zowe Desktop.

## Logging in to the Zowe Desktop
Access and navigate the Zowe Desktop to view the Zowe applications.

### About this task
You will use the Firefox browser to enter the provided URL to log in to Zowe Desktop.

### Procedure
1.	In the taskbar, click the icon of Firefox to start the browser.
1.	Enter the following URL in the address bar to access the login page:
`https://10.149.60.146:8544/ZLUX/plugins/org.zowe.zlux.bootstrap/web/index.html`
    >**Notes:** Need to test this URL in the zTrial environment or replace with an accessible one.
1.	On the login page of the Zowe Desktop, enter your mainframe credentials.
    - **Username**: `TSTRADM`
    - **Password**: `TSTRADM`
    <!--Should we prevision this for users? Any risk on providing the credentials to users?-->
    <img src="./images/2-4.png" width="200">
1.	Press Enter.

### Results
Upon authentication of your user name and password, the desktop opens.

Several applications are pinned to the task bar. Click the Start menu and you will see a list of applications that are installed by default. You can pin other applications to the task bar by right-clicking the on the application icon and selecting **Pin to taskbar**.

<img src="./images/zowe-desktop.png" width="400">

### What to do next
You will use the JES Explorer to query the jobs with filters and view the related status.


## Querying JES jobs and viewing related status in JES Explorer
Use the Job Entry Subsystem (JES) Explorer to query JES jobs with filters and view the related status.

### Procedure

1. Click the **Start** menu on Zowe Desktop.  
    <img src="images/2-1.png" width="250">

1. Scroll down to find the JES Explorer icon and right-click to open it.  The JES Explorer is displayed.

1. Click the **Job Filters** column to expand the filter criteria. You can filter jobs on various criteria by Owner, Prefix, Job ID, and Status.  
    <img src="images/2-2.png" width="200">

1. To query the jobs starting with _ZOWE_ and in an active status, enter _ZOWE*_ in the **Prefix** field and select **ACTIVE** from the **Status** drop-down list, and click **APPLY**.  
    **Note:** Wildcard is supported. Valid wildcard characters are asterisk (*), percent sign (%), and question mark (?).

    <img src="images/zowe-jes-filter.png" width="200">

1. From the job filter results, click the job **ZOWECRF:STC01414[ACTIVE]**.  <!--Will need to update the job name to match the name in the zTrial wizard when ready-->
    The data sets for this job are listed.

1. Click **JESJCL** to open the JESJCL data set. The contents of this data set are displayed. You can also select other data sets to view their contents.  
    **Tip**: You can hover over the text to display a hover help window.  

    <img src="images/2-3.png" width="400">


### Results
You used the JES Explorer to query the JES jobs with filters and viewed the related steps, files, and status.

## What to do next
Next, you'll use the TN3270 application plug-in in Zowe Desktop to view the same job that you viewed in this task.


## Using TN3270 in Zowe Desktop to view the job
You will use the TN3270 application plug-in to view the same job that you filtered out in the previous task.

### About this task
Zowe not only provides new modern applications to interact with z/OS but also integrates traditional tool TN3270 that you are familiar with. This TN3270 application plug-in provides a 3270 connection to the mainframe on which the Zowe Application Server runs.

### Procedure
1. From the taskbar at the bottom of the Zowe Desktop, click the TN3270 icon to open the TN3270 application plug-in.

    <img src="images/3-1.png" width="200">

    The TN3270 panel is displayed which offers selections to access various mainframe services.

    <img src="images/zowe-tn3270-welcome.png" width="200">


<!--1. To connect to the MVS3BTS MVS service, enter _MVS3BTSO_ and press **enter**.-->

1. Enter the following command and press enter to log on to TSO.

    ```LOGON TSTRADM```

1. On the **TSO/E LOGON** panel, enter the password `TSTRADM` in the **Password** field and press **enter**.  
    You successfully log on to TSO.

1. To start ISPF, enter _ISPF_ and press **enter**.  The ISPF Primary Option Menu is displayed.  

2. To use SDSF to view output from a job, type _S_ at the Option prompt and press **enter**.  

    <img src="pics/3-2.png" width="200">

3. To view the jobs in an active status, type _DA_ at the command input prompt and press **enter**.  
    The jobs that are running are displayed.

4. To query the jobs starting with `ZOWECRF`, type `PREFIX ZOWECRF` at the commond input prompt and press **enter**.  
    The jobs that are both in an active status and start with ZOWECRF are displayed.

5. To view the contents of the job, type `S` next to the Jobname ZOWECRF and press enter.

    <img src="pics/3-3.png" width="100">

### What to do next
In the next step, you will use the MVS Explorer to make changes to a data set.

## Editing a data set in MVS Explorer
Use the MVS Explorer to edit a data set member and save the changes.

### About this task
The MVS Explorer view allows you to browse the MVS file system by creating filters against data set names.

### Procedure
1. Click the Start menu on Zowe Desktop.
1. Scroll down to find the **MVS Explorer** icon. Right-click and select **Pin to taskbar** to pin this application to the desktop for later use.
1. Click the MVS Explorer icon on the task bar. The MVS Explorer opens. The **Filter** field is pre-filled with the filter string `TRTRADM`. All the data sets matching this filter are displayed. You can expand a data set name and see the members in it.

     <img src="images/zowe-mvs-filter.png" width="200">

1. Locate and click the data set member `APPLY1` in `TSTRADM.SMPE.JCL`.
    <!--The data set name need to be updated later to match what's actually on zTrial image.-->  

    <img src="images/zowe-mvs-dataset-locate.png" width="400">

1. Edit the data set by adding `NOTIFY=TSTRADM`.

    <img src="images/zowe-mvs-dataset-edit.png" width="400">

1. Click **SAVE** to save your edits.  

    <img src="./images/zowe-mvs-dataset-save.png" width="400">

### Results

Your edits are saved!

### What to do next

Next, you’ll use Zowe CLI to view and remove the changes you just made.

## Using the Zowe CLI to edit a data set
Use Zowe CLI to download the data set that you edited using MVS Explorer in the previous step, make changes to it, and upload the changes to mainframe.

### About this task

Zowe CLI is a command-line interface that allows you to interact with z/OS from a variety of other platforms, such as cloud or distributed systems, to submit jobs, issue TSO and z/OS console commands, integrate z/OS actions into scripts, and produce responses as JSON documents. With this extensible and scriptable interface, you can tie in mainframes to distributed DevOps pipelines and build in automation.

### Procedures

1. Launch the Command Prompt.
    1. Click the Windows logo in the bottom-left corner of the screen.
    2. Inside the search field, enter `command` or `cmd`. Then, click or tap on the Command Prompt result.
1. To list the data sets of _TSTRADM_, enter the following command:    <!--Need to update the data set name here when zTrial environemnt is set up. >
    ```
    zowe zos-files list data-set "TSTRADM.*"
    ```
    The following data sets are listed.
     >**Notes:** (to add a screencapture for the command result.)
1. To download all the data set members of _TSTRADM.SMPE.JCL_, enter the following command:    
    ```
    zowe zos-files download all-members "TSTRADM.SMPE.JCL"
    ```
    The message `Data set downloaded successfully` indicates that the data set members are downloaded.
    >**Notes:** [-- Need to add a screen capture here when zTrial environment is set up.]
1. Use the text editor to open the data set member named _APPLY1_ by entering the following command:
   ```code TSTRADM/SMPE/JCL/APPLY1.JCL```

   The file opens in an editor.

1. Remove the change `,NOTIFY=TSTRADM` that you added in a previous step and save your edits.
    >**Notes:** Is this a good story? This is to lead users to practice using different apps. <!--Need to define what changes users should make-->
1. Open the command prompt and upload your changes to mainframe by entering the following command:
    ```
    zowe zos-files upload file-to-data-set TSTRADM/SMPE/JCL/APPLY1.JCL "TSTRADM.SMPE.JCL"
    ```

    The message `Data set is uploaded successfully` indicates that you've sucessfully uploaded your changes.

    >**Notes:** Need to test the parameters when zTrial Windows image is up and running.

### Results
Congratulations! You’ve used the Zowe CLI to edit a data set member and upload the changes to mainframe.

### What to do next
You will open the MVS Explorer again to view the updates that you made to the data set in this procedure.

## View the data set changes in MVS Explorer
Use the MVS Explorer to view the data set changes in the previous step.

### Procedures
1. Click the MVS explorer application icon from the taskbar.
1. Locate and click the data set member `TSTRADM.SMPE.JCL` and check the changes you just made by using Zowe CLI.

>**Notes**: Need to add a screen capture here.

### What to do next
Congratulations! Your edits take effect.

## Next Steps
Ready to become a Zowe extender? Try to explore how you could extend Zowe to create your own APIs and applications.

### Go deeper with Zowe
Enjoyed this scenario? In under 30 minutes, you have used the MVS Explorer to and Zowe CLI to edit the same data set member, and used the JES Explorer and TN3270 to query the same JES job with filters, all without leaving Zowe. Now that you’re familiar with Zowe components, why not try to learn more about the project and the value it delivers. Zowe also contains a lot more application plug-ins in both Zowe Desktop and Zowe CLI. For more information, see the following resources:

- [Zowe on Open Mainframe Project](https://www.openmainframeproject.org/projects/zowe)
- [Zowe community site](https://www.openmainframeproject.org/projects/zowe)
- [Zowe documentation site](https://zowe.github.io/docs-site/latest/)

### Try the Extending Zowe scenario
You can add your own application plug-ins to Zowe. See how easy it is to extend Zowe by trying the Extending Zowe scenario.
