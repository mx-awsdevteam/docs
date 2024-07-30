---
title: "Mendix Data Loader"
url: /appstore/modules/snowflake/mendix-data-loader/
description: "Describes the configuration and usage of the Mendix Data Loader application from the Snowflake Marketplace."
weight: 20
#If moving or renaming this doc file, implement a temporary redirect and let the respective team know they should update the URL in the product. See Mapping to Products for more details. 
---

## 1 Introduction

The [Mendix Data Loader](https://app.snowflake.com/marketplace/listing/GZTDZHHIE0/mendix-mendix-data-loader) allows for seamless data ingestion from operational Mendix applications by using an exposed OData service into Snowflake, enhancing an organization's business intelligence and reporting capabilities.

### 1.1 Typical Use Cases

The Mendix Data Loader supports a range of data ingestion tasks, enabling organizations to leverage their operational data within Snowflake for analytical purposes. The key functionalities include ingesting data dynamically from Mendix applications (only needing an OData endpoint and credentials) to Snowflake. The ingested data is stored in the target schema of the target database specified by the user and created by the Mendix Data Loader application. This target schema in the target database serves as a staging area. The user should copy the tables of the target schema into a database and schema where they want to store the ingested data. This should be done after every ingestion.

### 1.2 Prerequisites {#prerequisites}

To use the Mendix Data Loader, you must have the following:

* A Mendix application with a [published OData service](https://docs.mendix.com/refguide/published-odata-services/) that includes exposed entities. 
* A Snowflake environment.

### 1.3 Licensing and Cost

The Mendix Data Loader is covered under the Mendix EULA. While the loader itself does not incur additional costs, operating within Snowflake may incur a usage cost. For more information, refer to the [Snowflake pricing documentation](https://www.snowflake.com/en/data-cloud/pricing-options/).

Depending on your use case, your deployment environment, and the type of app that you want to build, you may also need a license for your Mendix app. For more information, refer to [Licensing Apps](/developerportal/deploy/licensing-apps-outside-mxcloud/).

## 2 Installation

Follow instructions in [Install an app from a listing](https://other-docs.snowflake.com/en/native-apps/consumer-installing) to add the component to your Snowflake environment.

## 3 Configuration

Once the Mendix Data Loader is deployed, follow these steps to configure and use it:

1. View the README file which the application displays upon starting.

2. Click the **MENDIX_DATA_LOADER** tab in the header to open the application interface.

3. Grant the application the **USAGE** privilege on a warehouse. This step is required if you want to schedule the data ingestions.

4. Grant the application the **CREATE DATABASE** and **EXECUTE TASK** privileges. This step is required for the application to create the staging database for data ingestion and to execute tasks.

5. Configure the Mendix Data Loader by performing the following steps:

    1. Select either **Basic** or **OAuth** as the authentication type.
    2. Click **Submit**.
    3. Provide the required information, depending on the selected authentication type.
   
        * For Basic authentication, provide the following information:
       
            * **API endpoint** - The base endpoint for the OData resource in your Mendix application, for example, `https://yourmendixapp.mendixcloud.com/odata/snowflakedata/v1/`
            * **Username** - The username for the basic authentication into the OData resource in your Mendix application
            * **Password** - The password for the basic authentication into the OData resource in your Mendix application
            * **Target database name** - The name of the database to which you want to ingest the data
            * **Target schema name** - The target schema name where all the data will be ingested

        * For OAuth authentication, provide the following information:
       
            * **API endpoint** - The base endpoint for the OData resource in your Mendix application
            * **Client ID** - The client ID from your OAuth provider
            * **Client secret** - The secret associated with the client from your OAuth provider
            * **Token endpoint** - The endpoint at which the token will be validated from your OAuth provider
            * **Allowed scopes** - The allowed and custom scopes configured on the client from your OAuth provider
            * **Access token validity** - The amount of time (in seconds) the access token is valid
            * **Target database name** - The name of the database to which you want to ingest the data
            * **Target schema name** - The target schema name where all the data will be ingested
     
    4. Click **Submit**.

6. Click **Generate access script**.
7. Copy the script, navigate to an SQL worksheet, and execute the entire script.

8. Ingest the data by choosing one of the following options:
 
    * If you want to do a one-time ingestion from a Mendix app into Snowflake, navigate to the main tab and click **Ingest Data**. All ingested data is stored in [transient tables](https://docs.snowflake.com/en/user-guide/tables-temp-transient#transient-tables).
    * If you want to create a schedule for data ingestion, navigate to the **Schedule Task** tab and specify the following configuration options:
        * **When should the ingestion task run?** - Provide the schedule using the CRON format. In this drop-down, you can choose between providing a custom CRON expression, Every day at 00:00 AM UTC, Every Monday at 00:00 AM UTC, or Every first day of the month at 00:00 AM UTC.
        * **Custom CRON expression** - This field should only be used when the user chooses to provide a custom CRON expression.
        * **Time out** - This is an optional setting that can be used to change after how much time a timeout exception should happen. 
        * **Number of retry attempts** - This setting sets how many retries should be performed if an ingestion job fails.
        * **Suspend task after number of failures** - This setting sets the number of times a task is allowed to consecutively fail before suspending the task.

          After configuring the above, click **Schedule Ingestion Task**. You can view details of the created task on the **Task Management** tab where you can also view its performed ingestion jobs, suspend or enable the task, and drop the task. At present, one ingestion task can exist at a time.

9. To view the ingested data, access the schema that was specified inside the specified target database.

The ingested data is stored in the target schema of the target database specified by the user and created by the Mendix Data Loader application. This target schema in the target database serves as a staging area. Because of that, you should copy the tables of the target schema into a database and schema that you want to use to store the ingested data. This should be done after every ingestion.

## 4 Verifying the Access Token

When using OAuth authentication with the Mendix Data Loader, it is crucial to verify the access token received by your Mendix application. This verification process ensures the token's authenticity and integrity, protecting your application from unauthorized access attempts.

{{% alert color="info" %}} If your organization already has an established solution for token verification, you can implement the Mendix Data Loader using OAuth as the authentication method without additional steps. However, if you do not have an existing solution, note that the platform-supported [OIDC SSO module](https://marketplace.mendix.com/link/component/120371) has this feature on its roadmap. {{% /alert %}}

### 4.1 Token Verification Process

To verify an access token, follow these high-level steps:

1. Decode the access token using the JSON Web Key Set (JWKS) from your OAuth provider.
2. Extract the claims from the token's payload.
3. Verify the payload by checking specific properties.

### 4.2 Verifying the Payload

When verifying the payload, typically you should check the following properties:

* `iss` (Issuer): Ensure it matches the expected OAuth provider's URL.
* `aud` (Audience): Confirm it corresponds to your application's client ID.
* `exp` (Expiration Time): Verify the token hasn't expired.
* `iat` (Issued At): Check if the token was issued at a reasonable time.

Additionally, you may need to verify custom claims specific to your OAuth provider or application requirements.

{{% alert color="info" %}} The exact properties and verification process may vary depending on your OAuth provider and security requirements. Always refer to your provider's documentation and your organization's security policies when implementing token verification. {{% /alert %}}

## 5 Technical Reference

### 5.1 Current Limitations

* Exposing an association in an Odata service is as a link is not supported yet by the Mendix Data Loader. Instead, choose the **As an associated object id** option in your Odata settings. This option will store the associated object ID in the table, but not explicitly as foreign key.
* The Mendix Data Loader supports single endpoint (OData) ingestion. If you want to ingest data from multiple endpoint, you can do this by ingesting the data from each endpoint separately one by one. Make sure to assign a different staging schema for every ingestion you do, or the previous ingestions will be overwritten. The ability to ingest data from multiple endpoints in one go will be added in a future release.
* The Mendix Data Loader always ingests all the data exposed by the OData published by your Mendix application. If you do not want to ingest all of the data inside the exposed entities, you must filter the data at the Mendix/OData side. 

### 5.2 Troubleshooting

If you encounter any issues while using the Mendix Data Loader, use the following troubleshooting tips to help you solve them.

For any additional troubleshooting, contact the [development team](mailto:sa_dev_team@mendix.com).

#### 5.2.1 Error Parsing JSON: Document Is Too Large

When ingesting data, the Mendix Data Loader shows an error similar to the following: `net.snowflake.client.jdbc.SnowflakeSQLException: Error parsing JSON: document is too large, max size 16777216 bytes`.

##### 5.2.1.1 Cause

The amount of data being ingested is so large that the JSON file has become too large to parse.

##### 5.2.1.2 Solution

To solve this issue, configure the exposed OData entities to have pagination. For the best performance, make the pages as large as possible while still ensuring that the JSON does not become too large to parse. 

#### 5.2.2 No Response from my Mendix Application when Pagination is Enabled on Mendix Studio Pro 10.10

In the process of ingesting data, the Mendix application may not return any values if pagination is enabled for the published OData service and if the Mendix Studio Pro version is 10.10.

##### 5.2.2.1 Cause

A bug in the published OData service resource in Mendix Studio Pro 10.10 where the application root url is set incorrectly causes no data to be returned.

##### 5.2.1.2 Solution

This issue is resolved in Mendix version 10.12.0 and up. If you wish to work around this issue, you can set the ApplicationRootUrl of the application so that it has a trailing slash "/", e.g., **https://mymendixapp.mendixcloud.com/**. This resolution is the same as setting a custom domain as described in the [Custom Domains Mendix Documentation](https://docs.mendix.com/developerportal/deploy/custom-domains/#use-custom-url).

### 5.3 Contact Information

For support or queries regarding the Mendix Data Loader, email the development team at [SA_Dev_Team@mendix.com](mailto:sa_dev_team@mendix.com).
