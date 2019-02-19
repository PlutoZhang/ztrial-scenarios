# Scenario 1: Getting started with Zowe

## Overview

Zowe offers modern interfaces that enables you to interact with z/OS in a way that is similar to what you experience on cloud platforms today. Like Mac or Windows, Zowe comes with a set of APIs and OS capabilities that applications build on and includes some applications out of the box.

This scenario walks you through the Zowe interfaces including the Zowe Desktop and Zowe CLI through several simple tasks to help you get familiar with Zowe.
- If you are new to Zowe, start with this scenario to explore the base Zowe features and functions.
- If you are familiar with Zowe, you can skip this scenario and directly go to **Scenario 2** which guides you to extend Zowe by creating your own APIs and applications.

This scenario guides you through the steps in roughly 30 minutes. By the end of the session, you'll know how to:
- Log in to the Zowe Desktop
- Use JES explorer to query jobs with filters, and view the related status
- Use TN3270 in the Zowe Desktop to view the job and (purge the job)
- Use the Zowe CLI to create data sets and upload to mainframe
- Use the MVS Explorer to view the data sets

<!--If you haven't started the scenario, complete the following steps:
1. Double-click Z Trial Wizard.exe on the desktop.
1. In the Z Trial Program window, select the scenario that you want and click **Explore scenario**.
1. In the pop-up window, click **Start scenario**.-->

As an introductory scenario, no previous knowledge of Zowe is needed. Please wait a moment while your development environment loads (this takes a minute or so). When it loads, get started by logging in to the Zowe Desktop.

## Logging in to the Zowe Desktop

### About this task
On the login page of the Zowe Desktop, enter your username and password and you will see the desktop with Zowe applications.

### Procedure
1.	In the taskbar, click the icon of Firefox to start the browser.
1.	In Firefox, enter the following URL in the address bar to access the login page:
https://myhost:httpsPort/ZLUX/plugins/org.zowe.zlux.bootstrap/web/index.html (URL to be replaced with an accessible one)
1.	On the login page of the Zowe Desktop, enter XXX in the Username field.
1.	Enter XXX in the Password field.
1.	Press Enter.    
The Zowe Desktop is opened.

### Results


### What to do next
The next step is to use the JES Explorer to query the jobs with filters and view its related status.

## Querying the JES job and viewing its related status in JES explorer

### About this task
You will use Job Entry Subsystem (JES) explorer to query the JES job with filters and view its related status.

### Procedure

1. Click the **Start** menu. <img src="pics/2-1.png" width="400">
1. Scroll down to find the JES Explorer and right-click to open it.  
The JES Explorer is displayed.
1. Click the Job Filters column to expand the filter criteria. You can filter jobs on various criteria by Owner, Prefix, Job ID, and Status. <img src="pics/2-2.png" width="400">
1. To query the jobs starting with _Zowe_ and in an active status, enter _Zowe*_ in the Prefix field and select ACTIVE from the Status drop-down field, and click **APPLY**.  
**Note:** Wildcard is supported. Valid wildcard characters are asterisk (*), percent sign (%), and question mark (?).
1. From the job filter results, click the job **ZOWECRF:STC29575[ACTIVE]** that you want.  
The output data sets for this job are listed.
1. Click **JESJCL** to open the JESJCL data set. The contents of this data set are displayed. You can also select other data sets to view their contents.  
**Tip**: You can hover over the text to display a hover help window.<img src="pics/2-3.png" width="400">


### Results
You've used the JES Explorer to query the JES jobs with filters and viewed the related steps, files, and status.

## What to do next
Next, you'll use the TN3270 application plug-in in Zowe Desktop to view the same job that you viewed in this task.

# Using TN3270 in Zowe desktop to view the job

## About this task
You'll use the TN3270 application plug-in to view the same job that you filtered out in the previous task. This TN3270 plug-in provides a 3270 connection to the mainframe on which the Zowe Application Server runs.

## Procedures
1. From the taskbar at the bottom of the Zowe Desktop, click the TN3270 icon to open the TN3270 application plug-in. <img src="pics/3-1.png" width="400">
The TN3270 panel is displayed which offers selections to access various mainframe services.
1. To connect to the MVS3BTS MVS service, enter _MVS3BTSO_ and press **enter**.  
1. Enter the user ID that you use to log on to TSO. Enter _craigf_ in the USERID field and press **enter**.
1. On the TSO/E LOGON panel, enter the password in the **Password** field and press **enter**.  
You successfully log on to TSO.
1. To start ISPF, enter _ISPF_ and press **enter**.  
The ISPF Primary Option Menu is displayed.  
2. To use SDSF to view output from a job, type _S_ at the Option prompt and press **enter**. 
<img src="pics/3-2.png" width="200">
3. To view the jobs in an active status, type _DA_ at the command input prompt and press **enter**.  
The jobs that are running are displayed.
4. To query the jobs starting with ZOWECRF, type `PREFIX ZOWECRF` at the commond input prompt and press **enter**.  
The jobs that are both in an active status and start with ZOWECRF are displayed.
9. To view the contents of the job, type `S` next to the Jobname ZOWECRF and press enter. <img src="pics/3-3.png" width="200">

## What to do next


# Use the Zowe CLI to create data sets and upload to mainframe

## About this task

Zowe Command Line Interface (Zowe CLI) is a command-line interface that allows users to interact with z/OS from a variety of other platforms, such as cloud or distributed systems, to submit jobs, issue TSO and z/OS console commands, integrate z/OS actions into scripts, and produce responses as JSON documents. With this extensible and scriptable interface, you can tie in mainframes to distributed DevOps pipelines and build in automation.

You'll use Zowe CLI to create data sets and upload to mainframe...

## Procedures

Note: Zowe CLI contains a help system that is embedded directly into the command-line interface. When you want help with Zowe CLI, you can issue the following help command that provides you with information about the product, syntax, and usage:

`zowe - - help`

1. xxx
1. xxx
1. xxx

## What to do next

# Using the MVS Explorer to view the data sets

## About this task
You'll use the MVS Explorer to view the data sets.

## Procedures
1. In the bottom application draw of Zowe Desktop, you can see several explorer server icons. Click the MVS Explorer icon, the MVS explorer is displayed.
1. Use the high-level qualifier filter to browse the MVS file system (data set and members) and...
1. ...


## What to do next
Congratulations! You've successfully got hands-on experience with Zowe! ......


# Next Steps
Ready to become a Zowe extender? Try to explore how you could extend Zowe to create your own APIs and applications.

# Try the Extending Zowe scenario
...... link to the next scenario to be pasted here

# Go deeper with Zowe
Enjoyed this scenario? Now that you're familiar with each Zowe component, why not try to learn more about the product information and the value it delivers. For more information, see Zowe on Open Mainframe Project (https://www.openmainframeproject.org/projects/zowe).
