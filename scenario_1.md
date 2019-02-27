# Scenario 1: Getting started with Zowe

1. [Overview](#overview)
2. [Logging in to the Zowe Desktop](#logging-in-to-the-zowe-desktop)
3. [Editing a data set in MVS Explorer](#editing-a-data-set-in-mvs-explorer)
4. [Querying JES jobs and viewing related status in JES Explorer](#querying-jes-jobs-and-viewing-related-status-in-jes-explorer)
5. [Using TN3270 in Zowe Desktop to view the job](#using-tn3270-in-zowe-desktop-to-view-the-job)
6. [Using the Zowe CLI to edit JCL and upload to mainframe](#using-the-zowe-cli-to-edit-jcl-and-upload-to-mainframe)
7. [Using the MVS Explorer to view the data sets](#using-the-mvs-explorer-to-view-the-data-sets)
8. [Next Steps](#next-steps)
- [Try the Extending Zowe scenario](#try-the-extending-zowe-scenario)
- [Go deeper with Zowe](#go-deeper-with-zowe)

<!-- /TOC -->

## Overview

Zowe offers modern interfaces that enables you to interact with z/OS in a way that is similar to what you experience on cloud platforms today. Like Mac or Windows, Zowe comes with a set of APIs and OS capabilities that applications build on and includes some applications out of the box.

<img src="./images/zowe-architecture.png" width="400">

This scenario walks you through the Zowe interfaces including the Zowe Desktop and Zowe CLI through several simple tasks to help you get familiar with Zowe.
- If you are new to Zowe, start with this scenario to explore the base Zowe features and functions.
- If you are familiar with Zowe, you can skip this scenario and directly go to **Scenario 2** which guides you to extend Zowe by creating your own APIs and applications.

This scenario guides you through the steps in roughly 30 minutes. By the end of the session, you'll know how to:
- Log in to the Zowe Desktop
- View and edit data sets by using the MVS Explorer
- Query jobs with filters and view the related status by using the JES Explorer
- View jobs by using TN3270 in the Zowe Desktop
- Create data sets and upload to mainframe by using Zowe Command Line Interface (CLI)

<!--If you haven't started the scenario, complete the following steps:
1. Double-click Z Trial Wizard.exe on the desktop.
1. In the Z Trial Program window, select the scenario that you want and click **Explore scenario**.
1. In the pop-up window, click **Start scenario**.-->

As an introductory scenario, no previous knowledge of Zowe is needed. Please wait a moment while your development environment loads (this takes a minute or so). When it loads, get started by logging in to the Zowe Desktop.

## Logging in to the Zowe Desktop
Access and navigate the Zowe Desktop to view the Zowe applications.

### Procedure
1.	In the taskbar, click the icon of Firefox to start the browser.
1.	Enter the following URL in the address bar to access the login page:
`https://myhost:httpsPort/ZLUX/plugins/org.zowe.zlux.bootstrap/web/index.html` (URL to be replaced with an accessible one)
1.	On the login page of the Zowe Desktop, enter your mainframe credentials.
    - **Username**: TBD
    - **Password**: TBD
    <!--Should we prevision this for users? Any risk on providing the credentials to users?-->
    <img src="./images/zowe-desktop-login.jpg" width="200">
1.	Press Enter.

### Results
Upon authentication of your user name and password, the desktop opens.

Several applications are pinned to the task bar. Click the Start menu and you will see a list of applications that are installed by default. You can pin other applications to the task bar by right-clicking the on the application icon and selecting **Pin to taskbar**.

<img src="./images/zowe-desktop.png" width="400">

### What to do next
The next step is to use the MVS Explorer to make changes to data sets.

## Editing a data set in MVS Explorer
Use the MVS Explorer to edit a data set and save the changes.

The MVS Explorer view allows you to browse the MVS file system by creating filters against data set names.

### Procedure
1. Click the Start menu on Zowe Desktop.
2. Scroll down to find the **MVS Explorer** icon and click to open it. You can also right-click and select **Open** from the menu. The file opens in the editor.
3. Locate and click the data set member `CRAIG.TEST.DATA`.  
    <!--The data set name need to be updated later to match what's actually on zTrial image.-->  

    <img src="./images/zowe-mvs-dataset-locate.png" width="400">
4. Edit the data set.
5. Click **SAVE** to save your edits.  
    <img src="./images/zowe-mvs-dataset-save.png" width="400">

### Results

Your edits are saved!

### What to do next

Next, you'll use the JES Explorer to query the jobs with filters and view the related status.

## Querying JES jobs and viewing related status in JES Explorer
Use the Job Entry Subsystem (JES) Explorer to query JES jobs with filters and view the related status.

### Procedure

1. Click the **Start** menu on Zowe Desktop.  
    <img src="pics/2-1.png" width="250">

1. Scroll down to find the JES Explorer icon and right-click to open it.  The JES Explorer is displayed.

1. Click the **Job Filters** column to expand the filter criteria. You can filter jobs on various criteria by Owner, Prefix, Job ID, and Status.  
    <img src="pics/2-2.png" width="200">

1. To query the jobs starting with _Zowe_ and in an active status, enter _Zowe*_ in the **Prefix** field and select **ACTIVE** from the **Status** drop-down list, and click **APPLY**.  
    **Note:** Wildcard is supported. Valid wildcard characters are asterisk (*), percent sign (%), and question mark (?).

1. From the job filter results, click the job **ZOWECRF:STC29575[ACTIVE]** that you
want.  <!--Will need to update the job name to match the name in the zTrial wizard when ready-->
    The output data sets for this job are listed.

1. Click **JESJCL** to open the JESJCL data set. The contents of this data set are displayed. You can also select other data sets to view their contents.  
    **Tip**: You can hover over the text to display a hover help window.  
    <img src="pics/2-3.png" width="400">


### Results
You've used the JES Explorer to query the JES jobs with filters and viewed the related steps, files, and status.

## What to do next
Next, you'll use the TN3270 application plug-in in Zowe Desktop to view the same job that you viewed in this task.

## Using TN3270 in Zowe Desktop to view the job
You'll use the TN3270 application plug-in to view the same job that you filtered out in the previous task. This TN3270 plug-in provides a 3270 connection to the mainframe on which the Zowe Application Server runs.

## Procedure
1. From the taskbar at the bottom of the Zowe Desktop, click the TN3270 icon to open the TN3270 application plug-in.
    <img src="pics/3-1.png" width="400">

    The TN3270 panel is displayed which offers selections to access various mainframe services.

1. To connect to the MVS3BTS MVS service, enter _MVS3BTSO_ and press **enter**.

1. Enter the user ID that you use to log on to TSO. Enter _craigf_ in the USERID field and press **enter**.

1. On the TSO/E LOGON panel, enter the password in the **Password** field and press **enter**.  
    You successfully log on to TSO.

1. To start ISPF, enter _ISPF_ and press **enter**.  The ISPF Primary Option Menu is displayed.  

2. To use SDSF to view output from a job, type _S_ at the Option prompt and press **enter**.  
    <img src="pics/3-2.png" width="200">

3. To view the jobs in an active status, type _DA_ at the command input prompt and press **enter**.  
    The jobs that are running are displayed.

4. To query the jobs starting with ZOWECRF, type `PREFIX ZOWECRF` at the commond input prompt and press **enter**.  
    The jobs that are both in an active status and start with ZOWECRF are displayed.

5. To view the contents of the job, type `S` next to the Jobname ZOWECRF and press enter.   
    <img src="pics/3-3.png" width="200">

### What to do next
In the next step, you will explore Zowe CLI.

## Using the Zowe CLI to edit JCL and upload to mainframe
Use Zowe CLI to download a data set, edit the JCL, and upload the changes to mainframe.

Zowe CLI is a command-line interface that allows you to interact with z/OS from a variety of other platforms, such as cloud or distributed systems, to submit jobs, issue TSO and z/OS console commands, integrate z/OS actions into scripts, and produce responses as JSON documents. With this extensible and scriptable interface, you can tie in mainframes to distributed DevOps pipelines and build in automation.

### Procedures

1. Launch Zowe CLI from a bash terminal.
1. To list the data sets of _CRAIG.DEMO_, enter the following command:    
    ```
    zowe zos-files list data-set "CRAIG.DEMO.*"
    ```
    The following data sets are listed. (to add a screencapture for the command result.)
1. To download all the data set members of _CRAIG.DEMO.JCL_, enter the following command:    
    ```
    zowe zos-files download  all-members "CRAIG.DEMO.JCL"
    ```
    The message `Data set downloaded successfully` indicates that the data set members are downloaded.
1. Locate the data set member named _ztrial_ and use the text editor to open this member.
1. Make the required edits and click **SAVE** to save your edits.<!--Need to define what changes users should make-->
1. Open the terminal and upload your changes to mainframe by entering the following command:
    ```
    zowe zos-files upload file-to-data-set CRAIG/DEMO/JCL/ztrial.txt "CRAIG.DEMO.JCL"
    ```
### Results
You've used the Zowe CLI to edit JCL and upload to mainframe.
### What to do next
You'll use the MVS Explorer to view the JCL that you updated in this procedure.

## Using the MVS Explorer to view the data sets

### About this task
You'll use the MVS Explorer to view the data sets.

### Procedures
1. In the bottom application draw of Zowe Desktop, you can see several explorer server icons. Click the MVS Explorer icon, the MVS explorer is displayed.
1. Use the high-level qualifier filter to browse the MVS file system (data set and members) and...
1. ...


### What to do next
Congratulations! You've successfully got hands-on experience with Zowe! ......


## Next Steps
Ready to become a Zowe extender? Try to explore how you could extend Zowe to create your own APIs and applications.

## Try the Extending Zowe scenario
...... link to the next scenario to be pasted here

## Go deeper with Zowe
Enjoyed this scenario? Now that you're familiar with each Zowe component, why not try to learn more about the product information and the value it delivers. For more information, see Zowe on Open Mainframe Project (https://www.openmainframeproject.org/projects/zowe).
