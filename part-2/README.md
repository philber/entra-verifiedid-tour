---
page_type: sample
languages:
- node.js
- powershell
products:
- Microsoft Entra Verified ID
description: "A code sample application demonstrating custom issuance and verification of managed Custom Credentials."
urlFragment: "Part-2"
---

# A guided tour of Microsoft Entra Verified ID - Part 2 Sample

Welcome to the second part of the guided tour of Microsoft Entra Verified ID. 

In this part, we'll teach you to issue your first Custom Credential to your employees: a managed Verified Credential Card. You'll then use this card to prove to a verifier that you are a "verified employee" of your (fictitious) organization. 

## About this code sample application

This code sample application demonstrates how to use Microsoft Entra Verified ID to issue and consume managed Custom Credentials. 

This application in NodeJS is forked from the repo available at [https://github.com/Azure-Samples/active-directory-verifiable-credentials-node](https://github.com/Azure-Samples/active-directory-verifiable-credentials-node). The related code is located in the 1-node-api-idtokenhint.

**PLEASE REFER TO THESE REPOS FOR THE LATEST AVAILABLE BITS.**

The code sample application uses the Request Service REST API which supports ID Token to pass a payload for the verifiable credential. It has been adapted accordingly.

## Contents

As before, the NodeJS project is divided in 2 parts, one for issuance and one for verifying a Custom Credential. Depending on the scenario you need you can remove 1 part. To verify if your environment is completely working you can use both parts to issue a CustomCredentialTest VC and verify that as well.

| Issuance | |
|------|--------|
| public\issuer.html|The basic webpage containing the javascript to call the APIs for issuance. |
| issuer.js | This is the file which implements express routes which contains the API called from the webpage. It calls the Request Service REST API after getting an access token through MSAL. |
| issuance_request_config.json | The sample payload send to the server to start issuing a VC. |

| Verification | |
|------|--------|
| public\verifier.html | The website acting as the verifier of the verifiable credential.
| verifier.js | This is the file which implements express routes which contains the API called from the webpage. It calls the Request Service REST API after getting an access token through MSAL and helps verifying the presented verifiable credential.
| presentation_request_config.json | The sample payload send to the server to start issuing a VC.

## Setup

Before you can run this code sample application make sure your environment is setup correctly, follow the instructions provided in the first and second parts of this guide and/or in the documentation [here](https://aka.ms/vcsample).

## Setting up and running the code sample application
To run the sample, clone the repository, compile & run it. It's callback endpoint must be publically reachable, and for that reason, use a tool like `ngrok` as a reverse proxy to reach your app.

```Powershell
git clone https://github.com/philber/entra-verifiedid-tour.git
cd entra-verifiedid-tour\part-2
```

### Create your Custom Credential
To use the code sample application we need a configured managed Custom Credential in the azure portal.
In the project directory CredentialFiles you will find the `CustomCredentialTestDisplay.json` file and the `CustomCredentialTestRules.json` file. Use these 2 files to set the releated Display abd Rules definitions for your own CustomCredentialTest VC. 

If you navigate to your [Verified ID|Credentials](https://portal.azure.com/#view/Microsoft_AAD_DecentralizedIdentity/InitialMenuBlade/~/cardsListBlade) blade in azure portal, follow the instructions how to create your first verifiable credential.

You can find the instructions on how to create a managed Custom Credential in the azure portal [here](https://aka.ms/didfordev)

Make sure you copy the value of the credential URL after you created the credential in the portal. 
Copy the URL in the `CredentialManifest` part of the `config.json`. 
You need to manually copy your Microsoft Entra Verified ID service created Decentralized Identifier (did:ion..) value from this page as well and paste that in the config.json file for `IssuerAuthority` and `VerifierAuthority`.

### API Payloads
The API is called with special payloads for issuing and verifying Custom Credentials. The sample payload files are modified by the sample code by copying the correct values defined in the `config.json` file.
If you want to modify the payloads `issuance_request_config.json` and `presentation_request_config.json` files yourself, make sure you comment out the code overwriting the values in the issuer.js and verifier.js files. The code overwrites the Authority, Manifest and trustedIssuers values. The callback URI is modified in code to match your hostname.

The file [run.cmd](run.cmd) is a template for passing the correct variables and run your node.js code sample application.

## Running the code sample application

In order to build & run the code sample application, you need to have the [node](https://nodejs.org/en/download/) installed locally. 


1. After you have edited the file [config.json](config.json), start the node app by running this in the command prompt
```Powershell
npm install
.\run.cmd
```

1. Using a different command prompt, run ngrok to set up a URL on 8080. You can install ngrok globally from this [link](https://ngrok.com/download).
```Powershell
ngrok http 8080
```
1. Open the HTTPS URL generated by ngrok.
![API Overview](ReadmeFiles/ngrok-url-screen.png)
The sample dynamically copies the hostname to be part of the callback URL, this way the VC Request service can reach your sample web application to execute the callback method.

1. Select GET CREDENTIAL

1. In Authenticator, scan the QR code. 
> If this is the first time you are using Verifiable Credentials the Credentials page with the Scan QR button is hidden. You can use the `add account` button. Select `other` and scan the QR code, this will enable the Verifiable Credentials in Authenticator.
6. If you see the 'This app or website may be risky screen', select **Advanced**.
1. On the next **This app or website may be risky** screen, select **Proceed anyways (unsafe)**.
1. On the Add a credential screen, notice that:

  - At the top of the screen, you can see a red **Not verified** message.
  - The credential is based on the information you uploaded as the display file.

9. Select **Add**.

## Verify the verifiable credential by using the code sample application
1. Navigate back and click on the Verify Credential link
2. Click Verify Credential button
3. Scan the QR code
4. select the VerifiedCredentialExpert credential and click allow
5. You should see the result presented on the screen.

## About the code
Since the Request Service API is a multi-tenant API it needs to receive an access token when it's called. 
The endpoint of the API is https://verifiedid.did.msidentity.com/v1.0/verifiableCredentials/createIssuanceRequest and https://verifiedid.did.msidentity.com/v1.0/verifiableCredentials/createPresentationRequest.

Please note that the prior beta.did.msidentity and the beta.eu.did.msidentity hostnames for this API continue to work. For example for the former:
  - https://beta.eu.did.msidentity.com/v1.0/{yourTenantID}/verifiablecredentials/request if your tenant is located in Europe.
  - https://beta.did.msidentity.com/v1.0/{yourTenantID}/verifiablecredentials/request otherwise.

To get an access token we are using MSAL as library. MSAL supports the creation and caching of access token which are used when calling Azure Active Directory protected resources like the verifiable credential request API.

Typical calling the library looks something like this:
```JavaScript
var accessToken = "";
try {
  const result = await mainApp.msalCca.acquireTokenByClientCredential(mainApp.msalClientCredentialRequest);
  if ( result ) {
    accessToken = result.accessToken;
  }
} catch {
    console.log( "failed to get access token" );
    res.status(401).json({
      'error': 'Could not acquire credentials to access your Azure Key Vault'
      });  
    return; 
}
```
And creating an access token:
```JavaScript
const msalConfig = {
  auth: {
      clientId: config.azClientId,
      authority: `https://login.microsoftonline.com/${config.azTenantId}`,
      clientSecret: config.azClientSecret,
  },
  system: {
      loggerOptions: {
          loggerCallback(loglevel, message, containsPii) {
              console.log(message);
          },
          piiLoggingEnabled: false,
          logLevel: msal.LogLevel.Verbose,
      }
  }
};
const cca = new msal.ConfidentialClientApplication(msalConfig);
const msalClientCredentialRequest = {
  scopes: ["3db474b9-6a0c-4840-96ac-1fceb342124f/.default"],
  skipCache: false, 
};
```
> **Important**: At this moment the scope needs to be: **3db474b9-6a0c-4840-96ac-1fceb342124f/.default** This might change in the future

Calling the API looks like this:
```JavaScript
var payload = JSON.stringify(issuanceConfig);
console.log( payload );
const fetchOptions = {
  method: 'POST',
  body: payload,
  headers: {
    'Content-Type': 'application/json',
    'Content-Length': payload.length.toString(),
    'Authorization': `Bearer ${accessToken}`
  }
};
var client_api_request_endpoint = `https://verifiedid.did.msidentity.com/v1.0/verifiableCredentials/createIssuanceRequest`;
const response = await fetch(client_api_request_endpoint, fetchOptions);
var resp = await response.json()
```

## Deploying the code sample application to Azure AppServices
If you deploy the sample to **Azure AppServices**, as an alternative to using [ngrok](https://docs.microsoft.com/en-us/azure/active-directory/verifiable-credentials/verifiable-credentials-faq#i-can-not-use-ngrok-what-do-i-do), you need to update the files `config.json`, `issuance_request_config.json` and `presentation_request_config.json` with your changes before you do **Deploy To Web App** in VSCode. After you have deployed your code sample application, AppServices starts it via invoking the `npm start` command, which means if those files contain the right settings, your app will start successfully. When you create an Azure AppService instance, you should select **Runtime stack** = `Node` and **OS** = `Linux` and select the same region that you have your Azure KeyVault deployed to.

**package.json**
```json
  ...
  "scripts": {
    "start": "node app.js ./config.json ./issuance_request_config.json ./presentation_request_config.json",
  ...
```

If the app doesn't start, if you view the logs in AppServices LogStream, you may see the problem. See [docs](https://docs.microsoft.com/en-us/azure/app-service/configure-language-python#customize-build-automation).

## Troubleshooting

### Did you forget to provide admin consent? This is needed for confidential apps
If you get an error when calling the API `Insufficient privileges to complete the operation.`, this is because the tenant administrator has not granted permissions
to the application. See step 6 of 'Register the client app' above.

You will typically see, on the output window, something like the following:

```Json
Failed to call the Web Api: Forbidden
Content: {
  "error": {
    "code": "Authorization_RequestDenied",
    "message": "Insufficient privileges to complete the operation.",
    "innerError": {
      "request-id": "<a guid>",
      "date": "<date>"
    }
  }
}
```

### Understanding what's going on
As a first source of information, the Node sample will trace output into the console window of all HTTP calls it receives. Then a good tip is to use Edge/Chrome/Firefox dev tools functionality found under F12 and watch the Network tab for traffic going from the browser to the Node app.

## Best practices
When deploying applications which need client credentials and use secrets or certificates the more secure practice is to use certificates. If you are hosting your application on azure make sure you check how to deploy managed identities. This takes away the management and risks of secrets in your application.
You can find more information here:
- [Integrate a daemon app with Key Vault and MSI](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/tree/master/3-Using-KeyVault)

## More information

For more information, see MSAL.NET's conceptual documentation:

- [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure a client application to access web APIs](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis)
- [Acquiring a token for an application with client credential flows](https://aka.ms/msal-net-client-credentials)
