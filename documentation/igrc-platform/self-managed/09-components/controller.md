---
title: Controller
description: Documentation for controller usage and configuration
---

# Controller

## Home page

![Controller home page](images/controller_home.png "Controller home page")

This dashboard shows:  

- the current number of files present in the import files
- the number of uploads received
- the number of connectors declared in the portal
- the number of past execution plans

Moreover, you will see if an execution plan has been submitted or is currently running.  

## Import files page

![Import files](images/controller_importfiles.png "Import files")

This page lists all the files that will be part of the next execution plan.  

You can use the ![Download](images/download.png "Download") button to download the corresponding file and the ![Delete](images/trashcan.png "Delete") button to delete it.  

> Note that deleting a file only deletes that particular instance of the file.  
> If a file with the same name (and path) as a previously deleted one is included in a later upload, it will become part of the current list of files to be imported.  

## Uploads page

![Upload](images/controller_uploads.png "Upload")

This page lists all the received uploads. It also indicates, for each upload, the number of file it contained and the number of files still present in the import files.  

### Upload detail

![Upload details](images/controller_upload_detail.png "Upload details")

This shows all the files contained in a particular upload.  

The leftmost column will show a ![Current](images/file_current.png "Current") icon if the file in this upload is still present in the import files or a ![Obsolete](images/file_obsolete.png "Obsolete") if it has been superseded by a later upload (or manually deleted in the import files page).  

## Connectors page

![Connectors](images/controller_connectors.png "Connectors")

Shows the connectors declared via the portal.  

You can see, for each connector, if it is enabled or not, the date it was last executed (if any), the next date it will be executed and the number of available test files.  

> Note that the next execution date can be in the past if the orchestrator has not yet requested an execution of this connector.  

### Connector detail

![Connector details](images/controller_connector_detail.png "Connector details")

Here you can see the details of a declared connector.  

You can also request a test execution of this connector. It will run the connector and collect the resulting zip file but not merging it with the import files.  

In the test results table, you can use the ![Download](images/download.png "Download") button to download the test zip file and the ![Delete](images/trashcan.png "Delete") button to remove it.  

## Execution plans page

![Execution plan](images/controller_execplans.png "Execution plan")

Lists all the execution plans.  

An execution plan can be in one of the following states:  

- Submitted ![Submitted](images/execplan_submitted.png) (the batch process has not started yet)
- Running ![Running](images/execplan_running.png) (the batch process is running)
- Completed successfully ![Completed](images/execplan_completed.png)
- Finished with errors ![Error](images/execplan_error.png)

## Configuration

### Recipients of notification emails

![Controller notification configuration](images/controller_config_notifs.png)

Here you can configure the recipients of notifications for:  

- Execution plan success
- Execution plan failure
- Extraction failure

### Email server configuration

#### SMTP configuration

![](images/controller_config_smtp.png)

For SMTP emails you must configure:
- 'SMTP relay': the host name of the STMP relay server
- 'Port': the port the relay listen to (defaults to 25 if SSL mode is disabled or 465 otherwise)
- 'Authentication': authentication mode, can be 'If available', 'Always' or 'Never'
- 'User name': the connection name when authentication is used
- 'Password secret key': the key of the authentication password as defined in the configuration application
- 'SSL mode': whether secure SSL connection is used or not, can be 'Yes' or 'No'
- 'TLS': whether TLS is used or not, can be 'If available', 'Always' or 'Never'
- 'Sender email': the From address of sent emails

You will also find a section to send a test email to an address using the configuration defined above.  

#### Global SMTP

It is possible to use the SMTP relay parameters defined in the configuration UI for the controller. To do so select the "Global SMTP" option in the Email server configuration tab.  

#### Disabling notification emails

![](images/controller_config_none.png)

You can disable sending emails by selecting 'No email' in the 'Server type' combo box.  
