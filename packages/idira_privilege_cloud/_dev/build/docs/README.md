{{- generatedHeader }}
{{/*
This template can be used as a starting point for writing documentation for your new integration. For each section, fill in the details
described in the comments.

Find more detailed documentation guidelines in https://www.elastic.co/docs/extend/integrations/documentation-guidelines
*/}}
# Idira Privilege Cloud Integration for Elastic

## Overview
{{/* Complete this section with a short summary of what data this integration collects and the service it integrates with. */}}
The Idira Privilege Cloud integration for Elastic collects audit activity from [Idira Privilege Cloud](https://docs.cyberark.com/manage/latest/en/content/privilege-cloud/privilege%20cloud/privcloud-introduction.htm) (formerly CyberArk Privilege Cloud), the SaaS privileged access management service on the Idira Identity Security Platform. Audit records cover privileged account management, password and CPM operations, account access, approval workflows, Safe and member administration, user logins, master policy changes, and more.

### Compatibility
{{/* Complete this section with information on what 3rd party software or hardware versions this integration is compatible with */}}
This integration collects data from the Idira Identity Security Platform Shared Services (ISPSS) **Audit Export to SIEM** API, filtered to the Privilege Cloud service (application code `PAM`). Self-hosted CyberArk Privileged Access Security (PAS) deployments are not supported by this integration; use the [CyberArk Privileged Access Security](https://www.elastic.co/docs/reference/integrations/cyberarkpas) integration for those.

### How it works
{{/* Add a high level overview on how this integration works. For example, does it collect data from API calls or receiving data from a network or file.*/}}
Elastic Agent uses the CEL input to poll the Audit Export to SIEM API. It authenticates with an OAuth 2.0 access token from the tenant's Identity service, using the client credentials grant, and requests a new token when the current one expires. On each poll it creates a time-bounded audit query restricted to Privilege Cloud events and pages through the results with the cursor that the API returns. Each query window ends a short delay before the current time, so that recent audit records have time to become available. The end of the last completed time window is stored so that the next poll continues from where the previous one stopped.

## What data does this integration collect?
{{/* Complete this section with information on what types of data the integration collects, and link to reference documentation if available
*/}}
The Idira Privilege Cloud integration collects log messages of the following types:
* `audit`: Privilege Cloud audit activities, as described in the [Privilege Cloud audit events](https://docs.cyberark.com/audit-and-reports/latest/en/content/audit/product-audits/isp-privcloud-audit.htm) reference.

### Supported use cases
{{/* Add details on the use cases that can be enabled by using this integration. Explain why a user would want to install and use this integration. */}}
Use this integration to monitor privileged access in Idira Privilege Cloud from Elastic Security. For example, you can detect failed logins and unauthorized access attempts, track password retrievals and CPM password changes and failures, audit changes to Safes, Safe members, users, groups, and the master policy, and correlate privileged activity with events from other sources.

## What do I need to use this integration?
{{/* List any Elastic or vendor-specific prerequisites needed before starting to install the integration. For example, Elastic self-managed or cloud deployment, or a vendor-specific credentials or accounts */}}
- An Idira Privilege Cloud tenant on the Identity Security Platform Shared Services.
- An administrator with the **Show Export to SIEM in Administration space** right (granted to the Global Auditor role by default).
- An OAuth2 Server web app and an OAuth confidential service user in Idira Identity, configured as described below.
- An **Export to SIEM** integration, which provides the Audit API base URL and API key.
- Outbound HTTPS access from the Elastic Agent to the tenant's Identity host and to the Audit API host.

## How do I deploy this integration?

### Agent-based deployment

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](https://www.elastic.co/guide/en/fleet/current/elastic-agent-installation.html). You can install only one Elastic Agent per host.

Elastic Agent is required to poll the Idira Privilege Cloud Audit API and ship the data to Elastic, where the events will then be processed via the integration's ingest pipelines.

### Set up steps in Idira Privilege Cloud
{{/* List the steps that are required to set up the 3rd party system to send data to Elastic. 
This should be specific to the steps needed to set up the 3rd party system to send data to Elastic. It should not include generic information about how to install or set up the 3rd party system itself.
*/}}
1. Create an OAuth2 Server web app:
   1. Go to **Manage** > **Inventory** > **Targets** > **Applications**, and click **Add Web Apps**.
   2. On the **Custom** tab, locate the **OAuth2 Server** web app, and click **Add**.
   3. On the **Settings** tab, enter an **Application ID**. Note this value.
   4. On the **Tokens** tab, set **Token Type** to `jwtRS256`, and make sure that **Client Creds** is selected under **Auth methods**.
   5. On the **Scope** tab, add a scope named `isp.audit.events:read`.
   6. On the **Advanced** tab, add the following script, and click **Save**:
      ```
      setClaim('tenant_id', TenantData.Get("CybrTenantID"));
      setClaim('aud', 'cyberark.isp.audit');
      ```
2. Create a service user:
   1. Go to **Inventory** > **Identities** > **Users**, and click **Add User**.
   2. Enter the required fields, select **is OAuth confidential client**, and click **Create User**. The user's login name and password are the client ID and client secret for this integration.
   3. Open the user, select the **Application Settings** tab, click **Add**, and add the OAuth2 Server web app.
3. Go to **Inventory** > **Targets** > **Applications**, open the OAuth2 Server web app, and on the **Permissions** tab add the service user with the **Grant**, **View**, **Run**, and **Automatically Deploy** permissions.
4. Grant the service user the **Read Only System Administration** role right. Without this right, authentication succeeds but every audit query returns no records and no error.
5. In **Setup** > **Integrations** > **Export to SIEM**, create a third-party SIEM integration. Note the **API base URL** and the generated **API key**. A tenant can have at most two SIEM integrations, and their name and description cannot be edited after creation.
6. Note your tenant's **Identity ID**: the first label of the Identity host name, for example `ABC1234` in `ABC1234.id.cyberark.cloud`.

#### Vendor resources
{{/* Add vendor documentation links that are specific to the steps needed to set up the vendor system to send data to Elastic. Exclude this section if no vendor setup links are available. */}}
- [Export audit activities to SIEM applications](https://docs.cyberark.com/setup/latest/en/content/siem-integration/siem-export-3rd-party.htm)
- [Custom OAuth2 Server](https://docs.cyberark.com/manage/latest/en/content/identity/coreservices/authenticate/oauth2-server.htm)
- [Client credentials flow](https://docs.cyberark.com/identity/latest/en/content/developer/oauth/client-cred-flow.htm)

### Set up steps in Kibana
{{/* List the steps that are required to set up the integration in Kibana.
This includes how to add the integration, and how to configure the integration, with descriptions of each available configuration option.

If multiple input types are supported, add instructions for each in a subsection.
*/}}
1. In Kibana, go to **Management** > **Integrations**.
2. Search for **Idira Privilege Cloud**, and select it.
3. Click **Add Idira Privilege Cloud**.
4. Configure the integration:
   - **Audit API URL**: The API base URL of your Export to SIEM integration, without the `/api/audits/stream` path.
   - **SIEM Integration API Key**: The API key of your Export to SIEM integration.
   - **Identity ID**: Your Idira Identity tenant ID. It forms the token URL `https://<Identity ID>.id.cyberark.cloud/oauth2/token/<Application ID>`.
   - **OAuth2 Server Application ID**: The Application ID of the OAuth2 Server web app.
   - **Use Legacy Identity Domain**: Enable if your Identity URL uses the legacy `my.idaptive.app` domain.
   - **Client ID** and **Client Secret**: The login name, including its tenant suffix, and the password of the service user.
   - **Interval**: How often to poll the Audit API. The default is `5m`.
   - **Initial Interval**: How far back to collect events on the first run. The default is `24h`. The Audit API returns at most seven days of events.
5. Under **Advanced options**, you can set a **Token URL** that overrides the built-in token URL if your Identity host doesn't follow the standard pattern, set the **Query Delay** (default `5m`) that each query window ends before the current time, and configure the proxy, SSL, HTTP timeout, and the **Audit API Rate Limit**.
6. Click **Save and continue**.

### Validation
{{/* In this section, describe the actions required to validate that the integration is working properly, and data is flowing into Elasticsearch.
If required, list the steps needed in the vendor product to start sending events or trigger alerts.
Then list how to validate that the data is in Elasticsearch, using Kibana. This could be which indices to check in the Discover table, or which built in dashboards to look at to see the data.
*/}}
1. Generate audit activity in Idira Privilege Cloud, for example by signing in to the Privilege Cloud portal or viewing an account in a Safe.
2. Wait for the next polling interval. Audit records can take several minutes to become available through the API.
3. In Kibana, go to **Discover** and select the `logs-*` data view.
4. Filter on `data_stream.dataset : "idira_privilege_cloud.audit"` and confirm that events appear.

## Troubleshooting
{{/* The troubleshooting section should contain troubleshooting for common issues specific to this integration. Do not include generic troubleshooting information. Where appropriate, include details specific to each input type.
Whenever possible, link to the troubleshooting documentation provided by the third-party software. 

IMPORTANT: Use plain text for issue descriptions, NOT bold. Example:
- No data is being collected: Verify network connectivity and credentials
- TCP framing issues: Check that both sides use the same framing method
Do NOT use **bold** for issue names like "**No data is being collected**:".
*/}}
For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

- No events are collected, but there are no errors: Confirm that the service user has the **Read Only System Administration** right. Without it, the Audit API accepts the token but returns no records.
- Token requests fail with HTTP 400 or 401: Verify the Identity ID, the OAuth2 Server application ID, and the service user's credentials, and confirm that the service user is an OAuth confidential client with permissions on the OAuth2 Server web app.
- Audit API requests fail with HTTP 403: Verify the API key from the Export to SIEM integration.
- Audit API requests fail with HTTP 401: Confirm that the OAuth2 Server web app grants the `isp.audit.events:read` scope and sets the required audience and `tenant_id` claims.
- Older events are missing: The Audit Export to SIEM API returns only events from the previous seven days, so the initial lookback cannot exceed seven days.
- Audit API createQuery requests fail with HTTP 400: The integration sends `dateFrom` and `dateTo` in the `YYYY-MM-DD HH:MM:SS` format documented in the SIEM integration API reference, in UTC. Enable the request tracer, check the error body returned by `createQuery`, and confirm that your tenant accepts this format.
- Recent events are missing: Audit records can take several minutes to become available through the API, and a time window is not queried again after it has been collected. If records are missed, increase the **Query Delay** advanced option.
- Events arrive late or collection falls behind: The Audit API allows one call per minute and returns at most 500 events per call, so the integration can collect about 500 events per minute (about 720,000 per day). If the tenant produces more, collection falls behind. Once it is more than seven days behind, older events are skipped because the API only serves the previous seven days.

## Performance and scaling
{{/* Add any vendor specific performance and scaling information to this section.
Performance and scaling information should be specific to sending data to Elasticsearch. It should not include information about the vendor product itself or generic information about performance and scaling.
*/}}
The Audit Export to SIEM API is documented as accepting only one call per minute, and every results page is a separate call. At the default page size of 500, collection is limited to about 500 audit events per minute. Keep the page size at 500 for high-volume tenants. Do not configure more than one integration policy against the same Export to SIEM integration, and avoid polling intervals shorter than the default.

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### Inputs used
{{/* All inputs used by this package will be automatically listed here. Do not modify this section. */}}
{{ inputDocs }}

### API usage
{{/* For integrations that use APIs to collect data, document all the APIs that are used, and link to relevent information. For integrations that do not use APIs, do not include this section. */}}
These APIs are used with this integration:
* OAuth 2.0 token endpoint: `POST https://<identity-id>.id.cyberark.cloud/oauth2/token/<application-id>` ([documentation](https://docs.cyberark.com/identity/latest/en/content/developer/oauth/client-cred-flow.htm))
* Create audit query: `POST <audit-api-base-url>/api/audits/stream/createQuery` ([documentation](https://docs.cyberark.com/audit/latest/en/content/audit/isp_siem-integration-api.htm))
* Get audit query results: `POST <audit-api-base-url>/api/audits/stream/results` ([documentation](https://docs.cyberark.com/audit/latest/en/content/audit/isp_siem-integration-api.htm))

### Vendor documentation links
{{/* Add vendor documentation links which provide useful general information about the integration.*/}}
- [Introduction to Idira Privilege Cloud](https://docs.cyberark.com/manage/latest/en/content/privilege-cloud/privilege%20cloud/privcloud-introduction.htm)
- [Audit properties and service codes](https://docs.cyberark.com/audit-and-reports/latest/en/content/audit/isp_system-activities.htm)
- [Privilege Cloud audit events](https://docs.cyberark.com/audit-and-reports/latest/en/content/audit/product-audits/isp-privcloud-audit.htm)

### Data streams
{{/* Repeat all information in this section for each data stream the package collects.*/}}

#### audit

The `audit` data stream provides Privilege Cloud audit activities from the Audit Export to SIEM API, including privileged account management, account access, approval workflow, Safe and member management, user, group, and location management, application management, user login, master policy, report management, account discovery, component authentication, bulk operation, and IP allowlist events.

##### audit fields

{{ fields "audit" }}

##### audit sample event

{{ event "audit" }}

{{/* Export ILM Policies
     This accepts a list of data stream names as arguments and documents one policy
     per data stream. If no arguments are provided, data streams are auto-discovered by
     scanning for data_stream/<name>/elasticsearch/ilm/ directories and
     data_stream/<name>/lifecycle.yml files (lifecycle.yml takes precedence when both exist).

     If there are no policies defined, this renders as an empty string (no heading).
*/}}
{{ ilm }}

{{/* Export Transforms
     This will export the transforms used by this integration.
     If there are no transforms defined, this renders as an empty string (no heading).
*/}}
{{ transform }}
