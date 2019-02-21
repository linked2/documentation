# Onboarding Shopify2Go with the configuration API

Shopify2Go is an integration product for moving Shopify orders into PowerOffice Outgoing Invoices.

In this two part blog we will look at how to configure and install a Shopify2Go integration using the Linked2 config API. We will build a small JavaScript console app for Node.js. In Part 2) we will use this small console app to actually on-board a Shopify customer to the Shopify2Go order integration.

First lets get clear about what we mean by "onboarding"? When we talk about onboarding we mean the whole process required to get a new end-user running data through their own integration pipeline. We want this to be as easy and smooth as possible.

We are going to use a demonstration integration product for this walk through and that product is Shopify orders to PowerOffice Go outgoing invoices. We call this product Shopify2Go.

Our goal is to end up with an integration pipeline configuired and live for a Shopify customer moving orders as they are taken in Shopify directly into PowerOffice.

In the blog here I have just include code snippets for the functions we will use in the app. The complete code is available on github, comes in at under 200 lines including a whole bunch of comments and console.logs!

## Authentication

The first step; you will need your Linked2 white lable "machine to machine" credentials. Linked2 will issue these to you when you open your account. They enable your application to authenticate with the Linked2 platform. You can communicate on behalf of your customers at the application level without requirement for your customers to give 3rd party application permissions.

The credentials consist of a client id and a client secret.

```JavaScript
async function authenticate() {
    const credentials = {
        client_id : "IfI1Y3lpFmTmCWXD7W5Ah4jTAuKWc2Gn",
        client_secret : "bfYvKC2EVKgaY4OT2lKRW7PW7OLjr-AU8_IiHQ9eImhQGekosmUWKscbkCF8kuiu",
        audience : "https://platform.linked2.io/api",
        grant_type : "client_credentials"
    }

    const authHeaders = {
        "Content-Type" : "application/json"
    }

    let rawResponse = await fetch('https://linked2.eu.auth0.com/oauth/token', {
        method : "post",
        body : JSON.stringify(credentials),
        headers : authHeaders
    });
    let json = await rawResponse.json();

    return json.access_token;
}
```

ok, pretty straight forwards so far. Lets try and keep it like this.

## Integration Product List

Once we have successfully authenticated we can retreive a list of integration products that we own hosted on the Linked2 platform.

We do that with a simple http get.

```JavaScript

async function getProductList() {
    let rawResponse = await fetch(baseUrl + "products", {
        method : "get",
        headers : headers
    });
    let json = await rawResponse.json();
    return json;
}

```

So far so good, almost too easy.

Once we have a list of our products we can retreive the productId for the Shopify2Go product from the list.

## Installing a product

A little terminology; we call an installed product an integration "stage". A product is a template, the uninstalled software. We must configure and install it before we have a useable integration. The great thing here is that you can install your product as many times as you like at no cost (it is yours after all).

To install your product for a customer of yours you will need to give some configuraton information and to do that you must obtain a *stage configurator* object.

```JavaScript
async function getStageConfigurator(productId) {
    let rawResponse = await fetch(baseUrl + "new/stage/" + productId, {
        method : "get",
        headers : headers
    });
    let json = "";
    json = await rawResponse.json();
    return json;
}

```

Provide the product Id for the Shopify2Go product and we get given a stage configurator that looks like this:

```JSON
{ "stageConfiguration":
   { "productId": "749d0000-0194-1005-611c-08d694a6c44f",
     "stageId": "a28e0000-3a2a-000d-c1a9-08d6973d8e3c",
     "customerName": "",
     "customerIdentifier": "",
     "secret": "nwojMpuGtfxxzCnt9eCyU994wKlx8N7PFTxliXFO/uw=",
     "stageName": "Shopify2Go",
     "stageDescription": "This will be customers stage, configure with the customer name, etc.",
     "stageGroup": "shopify2Go" },
  "filterConfiguration": {},
  "transformerConfiguration": {},
  "publishContextConfiguration": {
      "powerOfficeClientKey" : ""
      },
  "supportConfiguration": {
      "supportContactEmail": "",
      "phone": "",
      "primaryContact": ""
      },
  "schema" :  { }
  }
```

Except in a real one the schema property will be populated with a JSON schema for the other properties of the object.
You can see most of the stage configurator is defaulted and that some of the properties are empty objects. The empty objects simply mean that there is no configuration options required for that part.

There is really only one rule you must follow and that is do not change any of the 'Id' property values.

We simply want to fill in the empty property values, we don't worry about the empty objects {}. For Shopify2Go that is just the following properties:

* customerName
* customerIdentifier
* powerOfficeClientKey
* supportContactEmail
* phone
* primaryContact

`customerName` and `customerIdentifier` will appear in every stage configurator object for every integration product you have. In general we would aslo expect to see support information but it is not required in the same way. Everything else is specific to the product.

`customerIdentifier` should be a unique identifier for your end-customer. We will track your customers usage with this identifier. It can be anything so long as it is unique for you. The shop domain makes a good human-readable identifier for Shopify, so we will use that.

The `powerOfficeClientKey` is a GUID that you must obtain from PowerOffice. We will use it to publish outgoing invoices into your account.

Then we just have some support contact details `supportContactEmail`, `phone` and `primaryContact`. These are contact details for you to use when Linked2 proactively raises a support ticket. You will see this information in that ticket so for example, if an order failed to publish into PowerOffice and the Linked2 platform subsquently raised a support ticket detailing the failure you can contact the person you named in this support information.

Notice in the `stageConfiguration` object there is the `secret` property. This is a cryptographically generated shared secret that you can use to "sign" the webhooks you send us with an HMAC-SHA256 digest. We will use this to verify that your webhook really did come from you. Only you and Linked2 have this secret and we will issue one every time you request a new stage configurator. It is up to you how you use this.

All secrets are keys for an HMAC-SHA256 digest and we will also calculate the same digest using the same secret and compare the results.

If you wish to replace the secret with a differnt one you have generated then go ahead, we will use yours. You can even use the same secret everytime you install, but we would not recommend this for obvious security reasons.

```JavaScript
function configureStage( stageConfigurator)
{

    stageConfigurator["stageConfiguration"]["customerName"] = "My Test Customer AS";
    stageConfigurator["stageConfiguration"]["customerIdentifier"] = "piet-emonkey-no.myshopify.com";
    stageConfigurator["filterConfiguration"] = {};
    stageConfigurator["transformerConfiguration"] = {};
    stageConfigurator["publishContextConfiguration"]["powerOfficeClientKey"] = "04ffda34-bb33-4c6b-9205-6b3c8442c239";
    stageConfigurator["supportConfiguration"]["supportContactEmail"] = "at@email.com";
    stageConfigurator["supportConfiguration"]["phone"] = "1234 552 526";
    stageConfigurator["supportConfiguration"]["primaryContact"] = "Support Person";


    return {
        stageConfiguration : stageConfigurator["stageConfiguration"],
        filterConfiguration : stageConfigurator["filterConfiguration"],
        transformerConfiguration : stageConfigurator["transformerConfiguration"],
        publishContextConfiguration : stageConfigurator["publishContextConfiguration"],
        supportConfiguration : stageConfigurator["supportConfiguration"],
    }
}
```

The `configureStage` function accepts the configurator returned by the `getStageConfigurator` function, sets the values as required and then returns a new configurator object of just the configuration without the schema.

## Data you must store

Clearly you will need to store the `secret`. The other peice of information you must store is the `stageId`. The `stageId` uniquely identfies the installed stage for your end-customer. Every webhook or batch you send us must be identified with the `stageId`. It is unique to this installation. You may simply choose to store it as the webhook URL. More on this below.

Thats it, just the shared secret and the stageId.

## Schema

The schema property holds a JSON Schema (Draft 7) object. I'm not going to say much about it here, there will be a follow up blog on getting the most from this schema. For now it is useful because you can use it to see what data is valid for the configuration. For instance the `PowerOfficeClientKey` is defined as

```JSON
"powerOfficeClientKey" : {
    "title" : "Your Power Office Client Key",
    "description" : "The client key (a GUID) you are given by Power Office for Api access",
    "type" : "string",
    "pattern" : "^([0-9A-Fa-f]{8}[-][0-9A-Fa-f]{4}[-][0-9A-Fa-f]{4}[-][0-9A-Fa-f]{4}[-][0-9A-Fa-f]{12})"
}
```

where we can see some basic information about what the value should be, including description and data type. More than that we can see it is a GUID and it must match the regular expression in the pattern.

We use the exact same schema to validate the stage configurator object when you submit the completed one back to Linked2 for installation.

You can copy the configurator object and schema object into separate objects if you wish to. Linked2 doesn't care if you return the configurator object with or without its schema. In the example code I have separated the schema from the rest of the configurator.

You can optionally choose to validate the configurator object with the schema before PUTing it back.

We recommend AJV for this, it is fast, well used and runs both for Node.js and browser. Plus it is really easy to use.

```JavaScript
var configuratorOnly = configureStage( stageConfigurator );

var ajv = new Ajv();
var valid = ajv.validate(stageConfigurator["schema"], configuratorOnly);
```

## Completing the Installation

The almost final step is just to save the configurator, this is effectively the install step.

```JavaScript
async function installStage( configurator ) {
    let rawResponse = await fetch(baseUrl + "stage", {
        method : "put",
        body : JSON.stringify( configurator ),
        headers : headers
    });
    let json = "";
    json = await rawResponse.json();
    return json;
}
```

The response content will be a JSON array. The response code will tell you if the configurator object was accepted or not.

## Failed installation
If the configurator failed the response content will be an JSON array of the reasons why. The response code will be a 400 Bad Request.

```JSON
{
    "keyword": "pattern",
    "dataPath" : ".stageConfiguration.customerIdentifier",
    "schemaPath" :"/definitions/stageConfig/properties/customerIdentifier/pattern",
    "params" : {
         "pattern": "^[a-zA-Z0-9_\\-.~]+$"
          },
    "message" : "should match pattern '^[a-zA-Z0-9_\\-.~]+$'"
}
```

This example error shows a failure to validate `customerIdentifier`. The `dataPath` shows the JSON path of `customerIdentifier` in the stage configurator object. The `schemaPath` shows you exactly which schema definition picked up the validation, and of course the message is showing us that the `customerIdentifier` did not match the required pattern.

Validation messages are a litle "raw" still. We hope in Beta 3.0 to be releasing custom, internationalised and localised messages. This will allow us to deliver messages that are meaningful to your end users in their language along side technical error messages that we have now.

## Successful installation

With a successful installtion the response code will be a 200 Success and you will receive a JSON object containing all the endpoints for the newly installed stage.

```JSON
{
  "stageId" : "a28e0000-3a2a-000d-9d65-08d697d3597d",
  "name": "Shopify2Go",
  "client": "emonkey Solutions AS",
  "customer": "My Test Customer AS",
  "webhookUrl":   "https://platform.linked2.io/api/webhooks/incoming/mystore/a28e0000-3a2a-000d-9d65-08d697d3597d",
  "batchJsonUrl":   "https://platform.linked2.io/api/batch/json/a28e0000-3a2a-000d-9d65-08d697d3597d",
  "batchCsvUrl": "coming soon!",
  "stageUsage":  "https://platform.linked2.io/api/billing/a28e0000-3a2a-000d-9d65-08d697d3597d/{year}/{month}",
  "allNotificationsResource": "https://platform.linked2.io/api/notification/backwards/a28e0000-3a2a-000d-9d65-08d697d3597d/{max}",
  "errorNotificationResource": "https://platform.linked2.io/api/notification/error/a28e0000-3a2a-000d-9d65-08d697d3597d/{max}",
  "warningNotificationResource": "https://platform.linked2.io/api/notification/warning/a28e0000-3a2a-000d-9d65-08d697d3597d/{max}",
  "successNotificationResource": "https://platform.linked2.io/api/notification/success/a28e0000-3a2a-000d-9d65-08d697d3597d/{max}",
  "license": "A valid license is required to access these API resources"
  }
  ```

  The webhook URL we want to copy that straight into the Shopify Webhooks creation.

  We can use the notification resource to check we have an installed and functioning stage.

## First notification

Initially you will only have a single notification. But after operating for a while you may have hundereds if not more. So a `{max}` number is required on the resource url to limit the number of notifications you receive back.

```JavaScript
async function readNotification( resource ) {
    let rawResponse = await fetch(resource, {
        method : "get",
        headers : headers
    });
    let json = "";
    json = await rawResponse.json();
    return json;
}
```

Here we have a simple function that is just GETing from any URL, so if we use it like the example below we will get our notification of success, where `installResults` is the object returned from a successful install:

```JavaScript
var notification = await readNotification( installResults.successNotificationResource.replace("{max}", "10") );
```

Although stage installation is pretty quick there are a few things happening behind the scenes and on a busy day it may be that it takes a moment longer to complete the installation. In other words if you immdeiately check for a successful installation it is likely that the network travel time between requests will be slower than the installation but it might not be. So the smart thing to do is wait a short time before checking.

The notification object that is returned has several items within it, including the complete definition for the installed stage. If you check the `noteType` and `message` properties you will see the important parts.

```JavaScript
    console.log( notification[0].noteType );
    console.log( notification[0].message  );
```

```Text
Success
Denne scenen ble initialisert vellykket, og den er klar til å motta data.
```

## 