# DRAFT DOCUMENTATION
Status: Unpublished

Author: Rob Smith

Version: 1.0

Date: 27th Dec 2018


# Configuration API

The configuration API is a RESTful api, all endpoints are idempotent, even when creating a stage.
If you are not familiar with Linked2 terminology then start here.

As a minimum you should understand what we mean by the following terms:

* product
* stage
* filter
* transformer
* publisher & publish context
* support

## Quick Start

1. Authenticate with your clientId and secret

```bash
curl --request POST \
  --url https://linked2.eu.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"UVFBjN8UL4oyhdNRtGwPA0IDBoQNd6Om","client_secret":"L-jPDlNS6jghCQs4_NWTuRVJslXWcLWeDhJS_M8fHRDqJHtHdTdph1SjAu5s3Qsfg","audience":"https://platform.linked2.io/api","grant_type":"client_credentials"}'
```

2. Request list of your products

```bash
curl -X GET \
  https://platform.linked2.io/api/config/products/ \
  -H 'Authorization: Bearer {your_bearer_token}' \
  -H 'cache-control: no-cache'
  ```

3. Request a new stage object using the correct productId and configure it

```bash
curl -X POST \
  https://platform.linked2.io/api/config/stage/{productId} \
  -H 'Authorization: Bearer {your_bearer_token}' \
  -H 'cache-control: no-cache'
```

4. Validate the stage object with the schema

5. Save the stage object

```bash
curl -X PUT \
  https://platform.linked2.io/api/config/stage \
  -H 'Authorization: Bearer {your_bearer_token}' \
  -H 'cache-control: no-cache'
  --data '{
    "stageConfiguration": {
        ...attributes for global stage config...
    },
    "filterConfiguration": {
        ...attributes for filter config...
    },
    "transformerConfiguration": {
        ...attributes for transformer config...
    },
    "publishContextConfiguration": {
        ...attributes for publish context config...
    },
    "supportConfiguration": {
        ...attributes for support config...
    }
}'
```

6. Test for readiness

```bash
curl -X POST \
  https://platform.linked2.io/api/notifications/{stageId} \
  -H 'Authorization: Bearer {your_bearer_token}' \
  -H 'cache-control: no-cache'
```

## Authentication and Authorization

When you take out a Linked2 subscription we provide you with an Application Client Id (an Id that is unique to your application) and Secret which is used to sign and validate Id tokens for the authentication flow. You need to safely store these for authentication. If at any time you suspect your secret may be compromised immediately inform us and we will rotate your secret and reissue.

Linked2 use Auth0 token based authentication services, which is itself an OAuth 2.0 standard ([RFC 6749, section 4.4](https://tools.ietf.org/html/rfc6749#section-4.4)) . Linked2 tokens have a 12 hour expiry after which you will need to refresh or obtain a new token.

It is a breach of your license to pass your application client Id and secret to a 3rd party. If it is necessary for a 3rd party to access the Linked2 the 3rd party must contact us for their own authentication.

If you have more than one application that you would like to access the Linked2 platform please contact us directly. You are allowed more than one application client Id per subscription and you will have advantages in separating different application and resources on separate logins, including better security, less Api throttling restriction, better billing reports and better support options.

The following example requests an `access_token` using curl. The given `client_id` and `client_secret` here are examples and should be replaced with your own.


```bash
curl --request POST \
  --url https://linked2.eu.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"UVFBjM9UL4oyhdNRtGwPA0IDBoQNd6Om","client_secret":"L-jPDlNS6jghCQs4_NWTuRVJslXWcLWeDhJS_M8fHRDqJHtHdTdph1SjAu5s3Qsfg","audience":"https://platform.linked2.io/api","grant_type":"client_credentials"}'
```

The response from this request will be

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ik5FUkNRekF5UXpaQ1FrRkVPVFF3TlRBeU9FUTFPRGRHUmpFME5qUkJRalkxTURrNE1ERTBOZyJ9.eyJodHRwczovL3BsYXRmb3JtLmxpbmtlZDIuaW8vYXBpL2xpbmtlZDJDbGllbnRJZCI6Ijc0OWQwMDAwLTAxOTQtMTAwNS01ZjY4LTA4ZDUxNGFjYTcwMSIsImlzcyI6Imh0dHBzOi8vbGlua2VkMi5ldS5hdXRoMC5jb20vIiwic3ViIjoiUzNrWE5qdG5qRFY4STBQQllFMkliMWlTdHVtNDdjTEtAY2xpZW50cyIsImF1ZCI6Imh0dHBzOi8vcGxhdGZvcm0ubGlua2VkMi5pby9hcGkiLCJpYXQiOjE1NDU4OTU2NTksImV4cCI6MTU0NTk4MjA1OSwiYXpwIjoiUzNrWE5qdG5qRFY4STBQQllFMkliMWlTdHVtNDdjTEsiLCJzY29wZSI6InByb2R1Y3Q6cmVhZCBzdGFnZTpyZW1vdmUgc3RhZ2U6ZW5hYmxlIHN0YWdlOmRpc2FibGUgc3RhZ2U6Y29uZmlndXJlIHN0YWdlOmNyZWF0ZSBiYXRjaDp1cGxvYWQgbGlua2VkMkNsaWVudElkOjc0OWQwMDAwLTAxOTQtMTAwNS01ZjY4LTA4ZDUxNGFjYTcwMSIsImd0eSI6ImNsaWVudC1jcmVkZW50aWFscyJ9.Wu2GdMSl5fyUDzpd4KajxQGHcxK6xj5qPPQ3887bmT7Tix0uGCmiE9yb4PySdFEKIRvU7Wfa7xilAQpLAZN_92zq9urQfeOwcyFLgwW3XsLLvPaeIMm9n2hvhAGHjT5xUDAbW0PEZJP-gLv2le6Om8FPO9Jz9XTCn_SfRR0t3HDVRBLRHioIluYJDObwiVNVS17g_loG_pu14pYO2sYhHQYk70X1K3QhTYH23QLLUTGdx1rYup5B96lk6XNmmYMCUpaWJ_KKLU_YoFZUTJhKj75uNUMX2SUKk7jd-6LnKOnDBAcHqE15Xm0X4CJ8j4aitjAPsuKC0yKIpXKjXYtAtA",
    "scope": "product:read stage:remove stage:enable stage:disable stage:configure stage:create batch:upload linked2ClientId:749d0000-0194-1005-5f68-08d514aca701",
    "expires_in": 86400,
    "token_type": "Bearer"
}
```

The `access_token` is a Json web token [RFC7519](https://tools.ietf.org/html/rfc7519) and you can decode them at JWT.io if you would like to examine it.

The `scope` property provides a list of grants your application is entitled to. This may vary according to subscription level. Each Linked2 endpoint will require one or more scopes for authorization which you must be granted in order to use the endpoint. Notice that the very last scope provides you with a human readable linked2 platform ClientId, this is always present. You need to provide this Id when accessing some Linked2 API endpoints.


## Onboarding a customer (installing a stage)

To install one of your integration products we first need to know the id for the product you want to install.

To get a list of products that you own use the `https://platform.linked2.io/api/config/products/{linked2_clientId}` endpoint, replacing the {linked2_clientId} with yours returned from the authentication.

```bash
curl -X GET \
  https://platform.linked2.io/api/config/products/749d0000-0194-1005-5f68-08d514aca701 \
  -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ik5FUkNRekF5UXpaQ1FrRkVPVFF3TlRBeU9FUTFPRGRHUmpFME5qUkJRalkxTURrNE1ERTBOZyJ9.eyJpc3MiOiJodHRwczovL2xpbmtlZDIuZXUuYXV0aDAuY29tLyIsInN1YiI6IlRzMmZDZTNnQjVkUG9ZU0xZTVRLNTEwRGtscjRIYmNWQGNsaWVudHMiLCJhdWQiOiJodHRwczovL3BsYXRmb3JtLmxpbmtlZDIuaW8vYXBpIiwiaWF0IjoxNTQ1NzM0Njk2LCJleHAiOjE1NDU4MjEwOTYsImF6cCI6IlRzMmZDZTNnQjVkUG9ZU0xZTVRLNTEwRGtscjRIYmNWIiwic2NvcGUiOiJwcm9kdWN0OnJlYWQgc3RhZ2U6cmVtb3ZlIHN0YWdlOmVuYWJsZSBzdGFnZTpkaXNhYmxlIHN0YWdlOmNvbmZpZ3VyZSBzdGFnZTpjcmVhdGUgYmF0Y2g6dXBsb2FkIiwiZ3R5IjoiY2xpZW50LWNyZWRlbnRpYWxzIn0.WHgzGhziFPBs-ax8Sd8ymDVmNoJROyAXgZ1YpbUMmtLtOpKuO5ZRivYyv7Q73LF7-IhGU9MBZbhHeXxEpkhL1tRNCMmiYVYNL-Q0muqyCQGquZO4n5LLEoVjmAkZbyl9BeOudQEbcb0Iz-whTHf2xTVxqnY_EyH4Rx0HH6cKHLl76rFTXRVfq5MSwSkAn1nUfPNvr0kjm3PAGg5ABSzsA7f-4p4DKcFe34yoL_b4LK4yhkiKVarrHkrVLa3pADFEVdArtlJJwLjGPqlQ2HKbmiBu1LQL6CjUR4H1RI-F-Xzu3bDJ5hwH2b2LhUwtkYCdhfD50hNxtlIjhVbDZeB38A' \
  -H 'cache-control: no-cache'
  ```

The response will be:

```json
[
    {
        "productId":"749d0000-0194-1005-1d6c-08d625e7104f",
        "productDefinitionVersion":"1.0",
        "productName":"MyCustomerData2Accounts",
        "productDescription":"Batch integration, Your Company CRM Data to Accounting System.","clientId":"749d0000-0194-1005-5f68-08d514aca701",
        "clientName":"Your Company Name AS"
    },
    {
        "productId":"749d0000-0194-1005-048d-08d625ee9167",
        "productDefinitionVersion":"1.0",
        "productName":"MyOrderData2Accounts",
        "productDescription":"Live integration, Your Orders to Accounting outgoing invoices.","clientId":"749d0000-0194-1005-5f68-08d514aca701",
        "clientName":"Your Company Name AS"
    }
]
```

a list of products that we have defined with you. The clientId here is the Linked2 ClientId that was returned as part of your authorization. This is not the same as the application client Id you use to obtain an access_token.

If you already know the product Id for the product you wish to install this call can be skipped.

## Requesting a new stage installation

This creates a temporary stage object which can be used to install and configure a new stage. You can create and dispose of many stages as you like. If you don't make use of the stage (i.e. save it) then there will be no impact on your account.

To start the stage installtion off use the newStage endpoint with the one of the productIds returned in your product list. `https://platform.linked2.io/api/config/newStage/{productId}`

```bash
curl -X POST \
  https://platform.linked2.io/api/config/stage/749d0000-0194-1005-141c-08d648591961 \
  -H 'Authorization: Bearer {your_bearer_token}'\
  -H 'cache-control: no-cache'
```

this results in a rather large object, but don't be dismayed there are basically two parts; the first is the stage configuration objects and the second part is a json schema intended for validating and understanding what is being requested by for configuration. In total there are 6 top level objects, 5 parts to the configuration and the schema which contains all the definition for the parameters found in configuration objects

```json
{
    "stageConfiguration" : { },
    "filterConfiguration" :  { },
    "transformerConfiguration" : { },
    "publishContextConfiguration" : { },
    "supportConfiguration" : { },
    "schema" : { }
}
```

This is the same for all stages. Each part is a configuration of one part of the stage, i.e. each part has a number of parameters that must be set in order for the stage to do its job. It is these parameters that allow each of your end-users to have a tailored stage. The numbers of parameters will vary depending on your integration product.

Some properties in these objects will already have data, some will be empty placeholders. Your are free to change data already present *except for any property ending in `id`*. A property ending in `id` such as `productId` or `stageId` is referenced by the platform and changing these will cause the stage to fail or simply not be valid when you try to save it.

The nature of different integration products means that we cannot know in advance of building a product what configuration parameters and objects will be required. However each products configuration will have a schema that describes what the parameters are and what acceptable data for each parameter are.

There are some consistent parameters in every configuration.

### stageConfiguration`

This is concerned with information about the stage at a global level. Examples of properties that might be found here are productId, stageId, timezone, and customer (your customer) identifier.

There will be some suggested data in the `stageName` and `stageDescription` properties and we suggest you change these to something more meaningful to your customer (for example, use their name rather than yours).

The properties `stageId` and `secret` are important to you. You must to store these two values in order to send integration requests for this stage (i.e. for this customer).

Our webhooks and Batch Api both check to see if the requests they receive are from who they say they are from and this is done using the secret. You can change the `secret` if you wish, assuming you update the stage configuration with you new secret, but you cannot change the `stageId`. We generate secrets according to the  [NIST RIJNDAEL Advanced Encryption Standard block cipher algorithm](https://www.cs.mcgill.ca/~kaleigh/computers/crypto_rijndael.html).

#### clientId

The linked2 clientId returned after authorization.

#### clientName

Your business name that you registered with us.

#### productId

The productId you specified in the call.

#### stageId

A newly created id for this stage. You must save the stageId so you can use it later. You should not change this id.

#### customerName

 Your end-customer name. We will use this to help you identify your customers usage and billing.

#### customerIdentifier

A unique identifier for your customer. We can use this as a validation to ensure stage and customer are matching, we also use it in reporting your customer usage back to you.

#### stageName

A sample name is provided, we suggest updating it to something more relevant that includes your customer name.

#### stageDescription

Descriptive text for this integration.

#### stageGroup

A tag, text based, that is used just for helping to manage groups of stages together. For example an end-customer may have several stages that are all grouped under their name.

#### noteDetail

The amount of data that will be included in a notification.

* Low -  will include only informational and error messages in notifications

* Medium -  will include the data being sent for integration.

* High.  - will add in the stage definition as well.

#### minimumNoteLevel

Debug, Information, Success, Warning, Error. Sets the level when a notification will be fired. Setting notification level to Warning will ignore lower level events. The levels are:

* Debug 
* Success
* Information
* Warning
* Error
* Critical
* Platform_Error
* Platform_Critical

#### secret

The secret key used for signing webhooks and batch Api requests. You must save this key. We recommend saving this key with the stageId together they are essential for using this stage.

### `filterConfiguration`

The first section of any integration stage is the filter. The job of the filter is to validate the data arriving for integration. Validation checks for conformance of data types, data ranges, data patterns, business rules and more. Any document failing validation will be reported on through your customers notification stream and does not progress further.

The filter configuration enables you to set validation parameters that may differ between customers. There is less in common between filters of different stages but typically we expect to see at least a your clientId and/or a customerIdentifier. These two allow some fundamental validation to happen around the data in the request actually being given to the correct stage.

#### clientId

see above.

#### customerIdentifier

see above.

### `transformerConfiguration`

The transformer does the work of transforming one set of business objects into another or creating new business objects in response to existing ones. This is where the real integration work is done, on a level that is much deeper than a simple string transformation the Linked2 platform operates on relationships within the context of the data it is given.

The transformer configuration allows the specifics of lookups, references, codes and even rules and relationships to be manipulated and varied from one installation to the next.

There is rarely, if ever, any commonality between the transformer configurations of different products, but typically we expect to see items such as tax codes, payment codes, etc that vary from customer to customer.

### `publishContextConfiguration`

Describes how the resulting transformation will be published. A publisher is able to connect to the target system with the appropriate account/user and send the transformed data in the target system format and/or protocols.

Publication might be via an API, a file transfer, EDIFACT, or more. Typically the publish context will need specific account information for your customer (username, password, client keys, etc), email addresses, and similar target system identifiers. Also expect to provide filenames, descriptions, etc if batch files are being published.
    
### `supportConfiguration`

Linked2 iPaaS can proactively issue support tickets to a ticketing system of your choosing. Tickets can be configured for warnings, errors, failures levels. The tickets that get reported here are typically about data errors and product errors (transformation and filter errors). This is typically the same system or team for each of your customers but does not have to be so. A large customer may have their own support and tickets can be raised in their system if that is a requirement.

Support tickets are integrated to the target support system using the Linked2 platform so you will find a defaulted support stageId provided but if you have a custom support channel you will need to talk to us about creating a the required integration.

#### supportProviderStageId

The stageId to be used when a support issue is raised.

#### minimumNotificationLevel

Tickets are raised on notification level. You can set the level here: 

* None
* Warning
* Error
* Critical

#### supportContactEmail

The email address of the primary contact for the support ticket system. 

### `Schema`

The schema uses [json-schema draft 07](https://json-schema.org/). You can use the schema in a variety of ways, perhaps the most obvious is simply to study the schema so that you understand the configuration parameters being requested and can ensure they properly completed. Every parameter is described both technically and with a written description. It will be clear that some parameters you will need to ask your customers for and others you can supply directly. 

You can use the schema to validate input in at least two different ways. First you can complete the all the stage configuration, wrap all the configuration objects in a surrounding object and attempt to use the schema to validate.

So a typical set of configuration objects might look like this:

```json
{
    "stageConfiguration": {
        "productId": "749d0000-0194-1005-141c-08d648591961",
        "stageId": "749d0000-0194-1005-0124-08d66be09896",
        "customerName": "A Customer AS",
        "customerIdentifier": "123876987",
        "stageName": "CustomerOrders2Go",
        "stageDescription": "Batch integration, Customer Orders to PowerOffice Go Vouchers.",
        "stageGroup": "A Customer",
        "secret": "MSKT65wVwZnM9jEk6AouanSsjp3ESQApfigJUYRg21A=",
        "noteDetail": "Low",
        "minimumNoteLevel": "Warning"
    },
    "filterConfiguration": {
        "customerIdentifier": "123876987"
    },
    "transformerConfiguration": {
        "Vat25Code": "3100",
        "Vat15Code": "3110",
        "Vat0Code": "3120",
        "completedPaymentType": "payment-completed"
    },
    "publishContextConfiguration": {
        "clientKey": "089106e1-21b5-47c4-9991-88889a57ac70",
        "importFileDescription": "A Customer Daily Import",
        "importFileName": "ACustomerDaily"
    },
    "supportConfiguration": {
        "supportContactEmail": "support@customer.no",
        "phone": "+2103939393",
        "primaryContact": "Support Person",
        "minimumNotificationLevel": "Warning"
    }
}
```

There a variety of tools to help you with the validation. One list can be found here https://json-schema.org/implementations.html#validators

We run this exact validation when you send us the stage to save. But by running locally as inputs are completed by the user you can deliver a better more responsive validation experience.

It is also possible to auto-generate a form from the schema. There are a number of tools available for this but perhaps the most mature and the one we have had most success with is the [React Json-Schema Form](https://github.com/mozilla-services/react-jsonschema-form)

How you use the schema is up to you, we simply recommend using it. Many of the schema standards present in it will have been developed with you to reflect your software.

## Saving or configuring a stage

A stage installation is completed when by successfully saving the set of configuration objects. You can save the completed set of objects together with the schema or you can discard the schema. Typically when we do this we like to discard just to avoid sending unnecessary data.



## Checking stage readiness

