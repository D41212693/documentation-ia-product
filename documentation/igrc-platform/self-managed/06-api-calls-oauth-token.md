# How to configure API calls

This guide provides information on the Access Portal API, including instructions for setting up a Client ID and Secret within your Keycloak realm for your self-managed Identity Analytics application.

## Set up API access

To authenticate against Keycloak and access the Portal API, you'll need to update the values used in your helm chart, update your deployment and get the newly generated client secret.

### Helm configuration

For authentication and authorization, you will impersonate an account when calling API's. In the example below we use the account named "setup" that is present in all timeslots.

As a reminder, it is required to use an account with access to the portal, _i.e._ an active account present in the current timeslots and reconciled with an active identity.

To configure this, add the following values to your helm values for IDA file:

```yaml
portal:
  internalWebServices:
    enabled: true
    impersonate: setup
    assumeRoles: technicaladmin
    authorizedEndpoints: /ws,/get_logs
    authorizedPort: 8081
  api:
    enabled: true # Required -- enables all 3 routes
    limits:
      concurrency:
        enabled: false # optional
        max: 5
        burst: 0
        delay: 1
      count:
        enabled: false # optional
        max: 20
        timeWindow: 60
      request:
        enabled: false # optional
        rate: 5
        burst: 0
```

> `portal.api.enabled` defaults to `false`. It must be set to true for the routes to be created in **APISIX**.

By default, the assumed role is "technicaladmin". You can modify the roles if needed by updating the value of the `assumeRoles` field.

Once the values of your helm updated update the deployment of the application using `helm upgrade --install ...`. Refer to the [updating a deployment section](./index.md/#updating-a-deployment) to learn more.

### Keycloak client secret

Log in to the Keycloak Admin Console, navigate to the Clients section and locate the client names `ida-portal-api`.

Retrieve the Client ID and Client Secret within the credentials tab.

## Executing an API call

### Getting the access token

To authenticate and get an access token, you need to make a POST request to the OpenID Token endpoint of your application realm passing different parameters.

The token endpoint is:

```sh
AUTH_URL="https://<EXTERNAL_DNS>/auth/realms/<CONTEXT>/protocol/openid-connect/token"
```

Where:

- `<EXTERNAL_DNS>` is the `externalDns` parameter configured in your helm values
- `<CONTEXT>` is the `pathPrefix` configured in your helm values

The parameters to pass are:

- `client_id`: The client ID of your application `ida-portal-api`
- `client_secret`: The client secret retrieved from Keycloak.
- `grant_type`: The type of grant being used, which is client_credentials in this case.

Please find below an example of cURL command to use:

```sh
token=$(curl \
  -d "client_id=$CLIENT_ID" \
  -d "client_secret=$CLIENT_SECRET" \
  -d "grant_type=client_credentials" \
  $AUTH_URL | jq -r '.access_token')
```

Where:

- `$AUTH_URL`: URL of the OpenID Token endpoint
- `$CLIENT_ID`: Client ID of your application
- `$CLIENT_SECRET`: Client secret retrieved from Keycloak
- `$API_URL`: URL of the API endpoint

> Note that in this example, we are utilizing jq, a command-line tool for processing JSON data.

### Making an API Request

The following endpoints are available:

- `/api/views/<view-name>`, to get view results
- `/api/compliancereport/<campaignInstanceId>`, to retrieve a compliance report
- `/api/workflow/<workflowIdentifier>`, to launch a new workflow instance
- `/api/workflow/terminate`, to kill a wokflow instance

#### Breaking change for version 3.5.1 & newer

> [!warning] The previous catch-all `/api/` route (version <= **3.5.0** ) has been renamed to `/api/views/` (version >= **3.5.1**).  
> Existing calls must be updated accordingly.
>
> For example:
> 
> `GET /api/my-view-name`
>
> Must **be changed** to:
>
> `GET /api/views/my-view-name`
>
> Only the `views` endpoint was available for version `3.5.0` & older.

#### View results (/api/views/)

You can call a view to get data from the database.

Sample curl call:

```bash
curl -H "Authorization: Bearer $TOKEN" "https://<EXTERNAL_DNS>/<CONTEXT>/api/views/<view-name>"
```

Where `view-name` is the view identifier in the project.
If you want to know more about calling views, see [Views and WebServices](../views/08-webservices.md)

#### Compliance report (/api/compliancereport/)

You can download the compliance report of a campaign.

Sample curl call:

```bash
curl -H "Authorization: Bearer $TOKEN" "https://<EXTERNAL_DNS>/<CONTEXT>/api/compliancereport/<campaignInstanceId>"
```

Where `campaignInstanceId` is the campaign's unique identifier (its ticketlog recorduid).

The response should look like this:

```json
{
  "success": [Boolean],
  "title": [Report name + extension],
  "content": [Report as a Base64-String]
}
```

#### Launching a workflow (/api/workflow/)

You can launch workflows via the API.

Sample curl call:

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  "https://<EXTERNAL_DNS>/<CONTEXT>/api/workflow/<workflowIdentifier>" -d '{...}'
```

Where `<workflowIdentifier>` is the workflow identifier in the project.

The body of the call must contain the following information:

```json
{
  "vars": {
    [List of variables to set in the dataset of the workflow]
  },
  "timeout": [Integer],
  "waitend": [Boolean]
}
```

> The timeout is in seconds

Here is a sample body to pass when calling the endpoint (in this cas for the `bwf_updateapplicationmetadata` workflow):

```json
{
  "vars": {
    "application": "SAGE_1726079098010_4785",
    "description": "Description updated via webservice"
  },
  "timeout": 10,
  "waitend": false
}
```

#### Terminating a workflow (/api/workflow/terminate )

You can launch workflows via the API.

Sample curl call:

```bash
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  "https://<EXTERNAL_DNS>/<CONTEXT>/api/workflow/terminate" -d '{...}'
```

The body of the call must contain the following information:

```json
{
  "processInstanceId": [String],
  "reason": [String],
  "failIfNotFound": [Boolean]
}
```

Here is an example:

```json
{
  "processInstanceId": "33564_1753868279311_7813",
  "reason": "Kill workflow from webservice",
  "failIfNotFound": true
}
```
