# Process Mining Loader

## Overview
Process Mining Loader is a webMethods.io Integration accelerator that allows user to upload the data into ARIS Process Mining. It supports both single upload and batch upload based scenarios. It creates table definition if it doesn't exist already and then uploads data into ARIS Process Mining.

## Pre-requisites
* webMethods.io Integration tenant
* ARIS Process Mining tenant

### ARIS Process Mining connector configuration

1. Setup ARIS Process Mining

    1. Login to your ARIS Process Mining tenant. 
    2. Create a system integration for the data ingestion API. Please refer to section 1.2.1 in [documentation](https://documentation.softwareag.com/aris/Process_Mining/10-0sr22/ypm10-0sr22e/10-0sr22_ARIS_Process_Mining_Data_Ingestion_API.pdf).
    3. Create a connection in the data set. Please refer to section 1.2.2 in [documentation](https://documentation.softwareag.com/aris/Process_Mining/10-0sr22/ypm10-0sr22e/10-0sr22_ARIS_Process_Mining_Data_Ingestion_API.pdf).
 
2. Setup webMethods.io ARIS Process Mining connector

    1. Login to the webMethods.io tenant.
    2. Go to Projects tab, select an existing project or create new project.
    3. Go to Connectors tab and select ARIS Process Mining connector under Predefined Connectors list.
    4. Configure Add Account for ARIS Process Mining connector.
        1. ARIS Cloud URL as your ARIS Process Mining tenant url.
        2. Project Room as tenant name.
        3. Data Set Name of the data set that exists in ARIS Process Mining.
        4. Client id & Client secret.

## Installation & Configuration

### Workflow installation
1. Download latest release **Process_Mining.zip** from Releases section.
2. Login to webMethods.io Integration tenant.
3. Go to Projects and select the project.
4. Click Import and choose **Process_Mining.zip**.
5. Choose ARIS Process Mining connector in Connect to ARIS Process Mining drop-down.
4. Click Import.

## Execution
 This workflow can be executed by invoking Webhook URL using HTTP REST Client.
 1. Login to webMethods.io Integration tenant.
 2. Go to Projects and select project and open workflow.
 3. Click Settings icon by hovering over the Webhook step.
 4. Copy the Webhook URL.
 5. Go to your REST Client to invoke the workflow.
    1. Request URL: Webhook url
    2. Request Method: POST
    3. Content-Type Header: application/json
    4. Basic Authentication
    5. Request Payload: example payload below
		
### Example playload
```
{
  "RecordHeader": {
    "dataLoadKey": "dataKey001",
    "commit": "true"
  },
  "TableInput": {
    "TableName": "users",
    "Namespace": "demo",
    "ColumnData": [
      {
        "name": "demo",
        "id": 12345,
        "mail": "demo@softwareag.com",
      },
      {
        "name": "demo2",
        "id": 56789,
        "mail": "demo2@softwareag.com",
      }  
    ]
  }
}
```

##### Input Description
* **dataLoadKey** - Unique key set by user for the table. 
* **commit** - if it set to true, after uploading the data into ARIS data will be committed. false - used for batch update and data upload will not be commiited. 
* **TableName** - ARIS Table Name
* **Namespace** - ARIS Table Namespace
* **ColumnData** - Array of data that needs to be uploaded into ARIS
		
##### Notes
1. For datetime fields transform the date into format MM/dd/yyyy HH:mm:ss.SSS then only it will be considered as datetime field.
2. For initial upload make sure all the fields are present in the first record of ColumnData.

# Examples

## Process Mining Jira Loader

### Overview
Process Mining Jira Loader is a webMethods.io Integration accelerator that allows uploading the Jira issues and changes to ARIS Process Mining. It fetch the issues and issue history details from Jira and transform them into array then invokes Process Mining workflow using its Webhook url to upload the data into ARIS.

### Pre-requisites
* webMethods.io Integration tenant
	* Process Mining workflow uploaded
* Jira software
* ARIS Process Mining

### Connector configuration

1. setup webMethods.io Atlassian Jira Connector
    1. Login to the webMethods.io tenant.
    2. Go to Projects tab, select an existing project or create new project.
    3. Go to Connectors tab and select Atlassian Jira connector under Predefined Connectors list.
    4. Configure Add Account for Atlassian Jira connector
        1. Select Authorization Type as Connection from drop down list and click Next
        2. Jira server URL as your Jira server url
        3. Username as jira login username
        4. API key as Jira API token. Please refer to create an api token section in [Documentation](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/)
        5. Set Hostname Verifier to org.apache.http.conn.ssl.NoopHostnameVerifier.
        6. Set Enable Connection Pooling to false.
        7. Set Session management to none.

2. setup webmethods.io Hypertext Transfer Protocol connector
    1. Login to the webMethods.io tenant.
    2. Go to Projects tab, select an existing project or create new project.
    3. Go to Connectors tab and select Hypertext Transfer Protocol connector under Predefined Connectors list.
        1. URL as webhook URL of Process Mining workflow
        2. Authentication Type as Basic
        3. Username as tenant login username 
        4. Password as tenant login password 

### Installation & Configuration

1. Install flow services
    1. Download latest release sendJiraIssueDetails.zip from Releases section. This zip file contains all the required flowservices.
    2. Login to webMethods.io Integration tenant.
    3. Go to Projects and select the project.
    4. Click Import and choose sendJiraIssueDetails.zip.
    5. Choose Atlassian Jira and Hypertext Transfer Protocol  connector in respective drop-down.
    6. Click Import.

##### Flow Services Description
* **sendJiraIssueDetails** - This flow service fetches the issues details from jira and invokes issueMapping flow service for mapping transformation and then invokes sendDataToARIS flowservice to upload the data into ARIS.
* **createTableWithSampleData** - This service load sample datainto ARIS before fetching JIRA details, So it will create table structure with all the required fields. This service execution is controlled by project parameter "loadSampleData".
* **issueMapping** - This flow service transform the jira issue details into array. 
* **sendDataToARIS** - This flow service invokes Process Minining workflow to upload the data into ARIS using HTTP call.


2. Install workflow
    1. Download latest release Send_Issue_Details_to_ARIS.zip from Releases section.
    2. Login to webMethods.io Integration tenant.
    3. Go to Projects and select the project.
    4. Click Import and choose Send_Issue_Details_to_ARIS.zip.
    6. Click Import.
   

##### Workflow Description
* **Send Issue Details to ARIS** - This is the main workflow that invokes sendJiraIssueDetails flows service with configured project paramerters.


3. Configure Project Parameters

Go to Configurations > Workflow > Parameter and update project parameters 

* **jql** - jql query to filter the issue data 
* **expand** - include additional fields into results. This usecase **"changelog"** has to be passed as value. This will includes the jira issue history into results while fetching the data. 
* **tableNamespace** - Name of the table namespace.
* **issueTableName** - Table name for issues table. 
* **changesTableName** - Table name for changes table.
* **dataLoadKey_changes** - Unique key for loading the data into changes table. 
* **dataLoadKey_issue** - Unique key for loading the data into issues table. 
* **batchRecordSize** - Records size for batch upload.
* **loadSampleData** - Whether to invoke createTableWithSampleData flow service or not. if set to "true" during the workflow execution it will invoke the flow service.
* **changesSample**  - Sample data with all the fields for chages table. 
* **issueSample** -  Sample data with all the fields for issues table.
	
### Execution
This workflow can be executed by clicking the run workflow button.


______________________
These tools are provided as-is and without warranty or support. They do not constitute part of the webMethods product suite. Users are free to use, fork and modify them, subject to the license agreement. While we welcome contributions, we cannot guarantee to include every contribution in the master project.

