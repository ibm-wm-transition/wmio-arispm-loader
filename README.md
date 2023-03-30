# Process Mining Loader

Process Mining Loader is a webMethods.io integration accelerator that allows user to upload the data into ARIS Process Mining. It supports both single upload and batch upload based scenarios. It creates table definition out of data and creates table, create data upload cycle, uploads data into ARIS and commit it.


## Pre-requisites
* webMethods.io Integration tenant
* ARIS Process Mining tenant

## Workflows and Flow Services
* **Process Mining** - This workflow uploads data into ARIS Process Mining. For initial upload it creates table definition out of first record of the data and creates table in ARIS. It supports batch upload, so user can send data into multiple batches and finally commits data.

## 1. Install workflow and flow services 
1. Download the latest release of workflow and flow services from Releases section.
2. Login to Software AG Cloud webMethods.io tenant.
3. Go to Projects and select/create project and click import. 
4. Choose **Process_Mining.zip** and import it.

## 2. Add New Account "ProcessMining" for ARIS Process Mining connector. Following information is required
1. ARIS Cloud URL.
2. Project and Data set name.
3. client id & client secret (Refer Administration > System Integration section of ARIS).
4. Set UMC Session Timeout and Response Timeout to default value.

## Inputs
* **dataLoadKey** - Unique key set by user for the table. 
* **commit** - if it set to true, after uploading the data into ARIS data will be committed. false - used for batch update and data upload will not be commiited. 
* **TableName** - ARIS Table Name
* **Namespace** - ARIS Table Namespace
* **ColumnData** - Array of data that needs to be uploaded into ARIS

## Sample Input
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
## Notes
1. For datetime fields transform the date into format MM/dd/yyyy HH:mm:ss.SSS then only it will be considered as datetime field.
2. For initial upload make sure all the fields are present in the first record of ColumnData

# Examples

## Process Mining Jira Loader

Process Mining Jira Loader is a webMethods.io integration accelerator that allows uploading the jira issues and changes data into ARIS Process Mining. It fetches the issue and issue history details from Jira and transform them into array then invokes Process Mining workflow[Link to Process Mining workflow] to upload the data into ARIS.

### Pre-requisites
* webMethods.io Integration tenant
	* Process Mining workflow to upload the data into ARIS 
* Jira software
* ARIS Process Mining

### Workflows and Flow Services
1. Send Issue Details to ARIS - This is the main workflow passes project paramerters to sendJiraIssueDetails flows service.
2. sendJiraIssueDetails - This flow service fetches the issues details from jira and invokes issueMapping flow service for mapping transformation and invokes sendDataToARIS flowservice to upload the data into ARIS.
3. createTableWithSampleData - This service load sample datainto ARIS before fetching JIRA details, So it will create table structure with all the necessary fields. For first time load we can enable this by setting loadSampleData parameter set to true.
4. issueMapping - This flow service contains mapping transformation for issues and changes
5. sendDataToARIS - This flow service invokes process minining API endpoint to upload the data into ARIS using HTTP call.

### 1. Install workflow and flow services 
1. Download the latest release of workflow and flow services from Releases section.
2. Login to Software AG cloud webmethods.io tenant .
3. Go to Projects and select project and click import. ( Order of import - sendJiraIssueDetails.zip, Send_Issue_Details_to_ARIS.zip)
4. Choose workflow/flow service zip and import it.

### 2. Add New Account "AtlassianJira" for Atlassian Jira Connector. Following information is required
1. Jira server URL.
2. Username and API key
3. Set Hostname Verifier to org.apache.http.conn.ssl.NoopHostnameVerifier.
4. Set Enable Connection Pooling to false.
5. Set Session management to none.

### 3. Add "New Account" for "HTTP_ProcessMining"  HTTP Connector. Following information is required
1. URL - API Endpoint/webhook URL of Process Mining workflow
2. Username and Password

### 4. Configure Project Parameters
Go to Configurations > Workflow > Parameter and update project parameters

* **jql** - jql squery to filter the issue data 
* **expand** - include additional fields into results. This usecase **"changelog"** has to be passed as value. This will includes the jira issue history into results while fetching the data. 
* **tableNamespace** - Table Namespace.
* **issueTableName** - Table name for issues table. 
* **changesTableName** - Table name for changes table.
* **dataLoadKey_changes** - Unique data load key for changes table. 
* **dataLoadKey_issue** - Unique data load key for issues table. 
* **batchRecordSize** - Records size for batch upload.
* **loadSampleData** - To identify createTableWithSampleData service needs to be invoked or not. 
* **changesSample**  - Sample changes JSON data with all the fields. 
* **issueSample** -  Sample issues JSON data with all the fields.
	

______________________
These tools are provided as-is and without warranty or support. They do not constitute part of the Software AG product suite. Users are free to use, fork and modify them, subject to the license agreement. While Software AG welcomes contributions, we cannot guarantee to include every contribution in the master project.
