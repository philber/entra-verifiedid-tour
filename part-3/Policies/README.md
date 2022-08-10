# Microsoft Entra Verified ID integration with Azure AD B2C
This folder includes what you need to integrate Azure AD B2C with Microsoft Entra Verified ID. Use cases are:

## What you will be able to do with the solution in this repository is:

- Issue Verifiable Credentials, a.k.a. Custom credentials, for accounts in your Azure AD B2C tenant
- Issue Verifiable Credentials during signup of new users in your Azure AD B2C tenant
- Signin to B2C with your Verifiable Credentials by scanning a QR code

## Deploy your Azure AD B2C tenant

Follow this tutorial on how to create your Azure AD B2C tenant [https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant).

Then, you need to make the B2C tenant ready for B2C custom policies via following this tutorial [https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-get-started?tabs=applications#custom-policy-starter-pack](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-get-started?tabs=applications#custom-policy-starter-pack). Make sure you deploy the Custom Policy Starter Pack and signup a test user.

## Configure and upload the B2C custom policies

The policies for the B2C+VC integration are B2C custom policies with custom html, which is needed to generate the QR code. You therefore need to first upload the html files to your Azure Blob Storage account and then edit and upload the xml-formatted custom policy files.

| File   | Description |
| -------- | ----------- |
| TrustFrameworkExtensionsVC.xml | Extensions for VC's to the TrustFrameworkExtensions from the Starter Pack |
| SignUpVCOrSignin.xml | Standard Signup/Signin B2C policy but that issues a VC during user signup |
| SignUpOrSignInVC.xml | Standard Signup/Signin B2C policy but with Verifiable Credentials added as a claims provider option (button) |
| SignUpOrSignInVCQ.xml | Standard Signup/Signin B2C policy but with a QR code on signin page so you can scan it already there. Signup journey ends with issue the new user a VC via the id_token_hint flow. |
| SigninVC.xml | Policy lets you signin to your B2C account via scanning the QR code and present your VC |
| SigninVCMFA.xml | Policy that uses VCs as a MFA after you've signed in with userid/password |

### Deploy the custom html

-  You can use the same storage account that you use for your VC credentials, but create a new container because you need to CORS enable it as explained [here](https://docs.microsoft.com/en-us/azure/active-directory-b2c/customize-ui-with-html?pivots=b2c-user-flow#2-create-an-azure-blob-storage-account). If you create a new storage account, you should perform step 2 through 3.1. Note that you can select `LRS` for Replication as `RA-GRS` is a bit overkill.
- Upload the files `selfAsserted.html`, `unified.html` and `unifiedquick.html` to the container in the Azure Storage.
- Copy the full url to the files and test that you can access them in a browser. If it fails, the B2C UX will not work either. If it works, you need to update the [TrustFrameworkExtensionsVC.xml](.\policies\TrustFrameworkExtensionsVC.xml) files with the `LoadUri` references.

### Create the REST API key in the portal
The Technical Profile `REST-VC-PostIssuanceClaims`, that is used during VC innuance during user signup, is configured to use an api-key for security. Therefor, create a Policy Key in the B2C portal with the name `RestApiKey` and manually set the key value to something unique. You need to add this value to the sample app's appSettings file also.

### Edit and upload the B2C Custom Policies

Do search and replace in all the xml-formatted files in this folder for:

- `yourtenant.onmicrosoft.com` to the real name of your B2C tenant, like `litware369b2c.onmicrosoft.com`
 
Then do the following changes to the [TrustFrameworkExtensionsVC.xml](.\policies\TrustFrameworkExtensionsVC.xml) file:

- Find `LoadUri` and replace the value of the url for `selfAsserted.html` that you uploaded in the previous step. 
- all places where `VCServiceUrl` references your deployment of the samples, like `https://df4a-256-714-401-356.ngrok.io/api/`
- all place where `ServiceUrl` references your deployment of the samples, like `https://df4a-256-714-401-356.ngrok.io/api/`

Upload the xml policy files to your B2C tenant starting with TrustFrameworkExtensionsVC.xml first. Note that you already should have uploaded the TrustFrameworkBase.xml and trustFrameworkExtensions.xml in a previous step. 

**A tip here, if you use `ngrok`, is to start it and leave it running so that the hostname doesn't change after you have uploaded the B2C policy files.**

## Testing

Steps:

1. Run the `B2C_1A_VC_susiq` (SignupOrSigninVCQ.xml) policy and sign up a new user. During the signup flow you will issue this user a VC
1. Run the `B2C_1A_VC_susiq` (SignupOrSigninVCQ.xml) policy or the `B2C_1A_signin_VC` (SigninVC.xml) policy, scan the QR code and signin to your B2C account using your VC. 