
# Description

Set of Policies help you to integrate/connect the Axway API-Management solution with the external Identity-Provider: https://auth0.com.
The purpose is to provide Application-Developers Self-Service capabilities they want using an API-Developer-Portal including API-Subscrioptions. On the other hand use a dedicated solution for Identity- & Token-Management.

![Self-Service-Flow](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/External_Token-Provider-Self-Service.png)

With this project the following flow is supported:
1. An Application Developer is using the API-Developer-Portal to create consuming applications & generate Application-Credentials (Client-ID & Secret)
2. Application credentials generated in API-Portal will be provided/generated by Auth0
3. Now an Application Developer can obtain an Access Token from Auth0
4. And call a protected API at the API-Management solution with that access token
5. The API-Management solution validates the token, identify the consuming application, approves API Access and apply quotas
6. An API-Administrator can manage API-Access (Disable the Application, remove the API-Subscription, Monitor consumption, etc.)

## API Management Version Compatibilty
This artefact can be used with Axway API Management version 7.6.2 and higher

## Prerequisites
An Auth0 Account - https://manage.auth0.com

## Configure Auth0
The API-Management solution will use the Auth0 Management REST-API to integrate. This API is secured by OAuth. 

Login to your Auth0 Management console and make sure you have selected the correct tenant in the upper right corner. 
Now an application must be created, that corresponds to your API-Management solution and is used to access the Auth0 Management REST-API.
Steps needed:
 1. Applications sections
 2. Create application
 3. Name it e.g. Axway API-Management
 4. Type: Machine to Machine app.
 5. Create
 6. Select Auth0 Management API
 7. Authorize (here you may restrict permissions of this application)
 8. In the settings tab please note the Client ID & Secret for later

Later, when issuing access tokens, they will be issued only for a certain usage (audience) and this is named in Auth0 an API. Before creating an API, please note your Auth0-Tenant-ID shown as audience for the Auth0 Management API:

![Auth0 Tenant ID](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/auth0_tenant_id.png)

Create an API in Auth0:
 1. APIs section
 2. Create API
 3. Give it a friendly name: e.g. "My APIs", meaning, that these tokens can only be used to access APIs on your API-Management platform
 4. Provide the Identifier: e.g. https://api.customer-name.com - Will be used to validate the token at API-Management runtime.
 5. Leave the default Signing Algorithm

With that, we are done with the basic Auth0 setup. More advanced configuration is not in scope of this document. 

## Configure your API-Management solution
### Policy import
The first step is to import the pre-configured Policy-Set and KPS-Collection. 
After import they will appear in the following container:
![Imported Policy set](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/imported_auth0_policy.png)

In addtion you need to the import the KPS-Collection, which will create the required Cassandra-DB table, during deployment:
![Auth0 KPS-Collection](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/imported_auth0_kps_collection.png)

### OAuth Client setup
To communicate with the Auth0 Management API an Access-Token is needed, which will be generated by the API-Gateway automatically, when needed. However, you need to setup the required Auth0 Token-Endpoints and Client-Credentials created before.

 1. In Enviroment configuration -> External Connections -> Client Credentials -> OAuth2
 2. Select Auth0 - Which has been imported with the Policy XML-Fragment
 3. In tab: "OAuth2 Credentials" edit the existing entry
 4. Add the Client-Id & Secret you have noted from the steps at Auth0
 5. In the tab: "OAuth2 Provider Settings"
 6. Fill in your tenant information, also noted from Auth0 above

![OAuth Client Credentials](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/oauth2_client_credential_settings.png)

![OAuth Provider settings](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/oauth2_client_provider_settings.png)

With that, the API-Gateway is able to automatically get an access token, when communicating with the Auth0 Management API.

### Auth0 Tenant setup
As the API-Management platform must communicate with your Auth0 tentant, the right tenant must be configured in some places.

Create a new EnvSettings property
Open <apigw-install>/groups/group-<n>/instance-<n>/conf/**envSettings.props**
Add a new propery: 
env.auth0_domain=<Your_Auth0_Tenant_Id> (e.g. test-axway.eu.auth0.com)

Open the Policy: Identity Provider/Auth0/**Auth0 - Validate Token**
Edit the filter: Check token details that it corresponds to you Audience & Tentant

Open the Policy: Identity Provider/Auth0/**Auth0 - AppCredentials Created**
Edit the filter: Create Client-Grant message and adjust the identifier for your API (Audience) to what you have configured.

### API-Manager configuration
Please make sure, in the API-Manager the following alerts are enabled. They are used to communicate with Auth0 for the required actions:
![Application alerts](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/application_alerts.png)
![Application Credential alerts](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/application_credential_alerts.png)

At runtime, tokens issued by Auth0 must be validated by the API-Manager. For this APIs will be protected with the security device "OAuth (External)" and a custom policy is doing the validation.

Steps to set this up:
1. In PS -> Server settings -> API Manager -> OAuth Token Information Policies
2. The Policy: "Auth0 - Token validation" is configured
3. In API-Manager Web-UI open a Front-End API (state: unpublished)
4. Configure Inbound security to: OAuth (External)
5. Select the Policy: "Auth0 - Token validation"

![API with OAuth External](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/apim-frontend_api_oauth_external_security_device.png)

## API-Portal
To test the integration just create an application in API-Portal and in the Authentication tab, generate an "External OAuth Credential". The Credentials you provide in this dialog doesn't matter, as it will be overwritten by the Client-ID generated by Auth0.

![External Client-Id](https://github.com/Axway-API-Management-Plus/auth0-apim-integration/blob/master/images/client_id_from_auth0.png)

So, for each Client-Id in the Authentification tab, one Application in Auth0 is generated. Deleting the Client-ID or the application in API-Portal, will delete all belonging applications in Auth0.

It's recommended to customize the API-Portal to have a dedicated section for Client-ID handling with Auth0 instead of using the standard External credentials section, but this is out of scope for now.


## Changelog
- 0.0.1 - 06.09.2018
  - Initial version


## Limitations/Caveats
- Enabling / Disabling of Client-IDs isn't supported, as Auth0 doesn't support it
- API-Portal customizing is not described / included
- displaying the generated secret would be part of API-Portal customizing as well

## Contributing

Please read [Contributing.md](https://github.com/Axway-API-Management-Plus/Common/blob/master/Contributing.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Team

![alt text][Axwaylogo] Axway Team

[Axwaylogo]: https://github.com/Axway-API-Management/Common/blob/master/img/AxwayLogoSmall.png  "Axway logo"


## License
[Apache License 2.0](/LICENSE)
