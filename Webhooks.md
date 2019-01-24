# DRAFT DOCUMENTATION
Status: Unpublished

Author: Rob Smith

Version: 1.1

Date: 1st Dec 2018

# Linked2 Webhook Receivers

Webhook Receivers are used for receiving single documents for transformation, batches should use the **Batch API**. There is a size limitation for the amount of data received by webhook.

The Linked2 platform receives webhook integration requests. Webhooks originating from different SaaS platforms will differ for reasons of choice, design and requirements of the SaaS business. As such we will install a webhook receiver for your particular webhook headers and body.

However, if you are considering developing a webhook and trigger on your SaaS application then we recommend the following method which is so common these days it can be considered a standard approach.

## Quick Start
We will assign you a webhook url with your subscription. It will be of the form `https://platform.linked2.io/api/webhooks/incoming/{clientName}/{id}`.

`{clientName}` will be your regsistered client name or an abbreviation of your client name, without spaces or hyphentations. `{id}` is the id that you will be given for each of your end-customers during the on-boarding process. See "Using the Linked2 Configuration API" for details on this. You must store this id from the on-boarding process.

Make a POST request to that resource in which you should include at least two headers; the topic and the base64 encoded HMAC-SHA256 digest of the body of the request.

The topic is the event that caused the webhook to fire.  Topic header data is typically structured as strings like this (but does not have to be):

```
customer/created
customer/updated
customer/deleted

```

The HMAC-SHA256 digest is calculated from the body of the request with a shared secret. 

You can calculate the HMAC like this:

c#
```c#
var key = Convert.FromBase64String(stage.Secret);
string base64Hash = null;
using (var shaAlgorithm = new System.Security.Cryptography.HMACSHA256(key))
{
    byte[] bodyBytes = System.Text.Encoding.UTF8.GetBytes(body);
    byte[] signatureHashBytes = shaAlgorithm.ComputeHash(bodyBytes);
    base64Hash = Convert.ToBase64String(signatureHashBytes);
}
```

Notice in .Net we use a disposable HMACSHA256 object, which is constructed with the shared secret key. If the key is base64 encoded remember to decode it first. Also that the body needs to be in the form of a byte array to compute the hash digest.

php
```php

$hmac_digest = base64_encode(hash_hmac('sha256', $body, $secret, $raw_output = true));
```

Suggested header names:
```
X-{clientName}-Hmac-Sha256
X-{clientName}-Topic
```

Optionally include a header with the customer identifier that you used during this customers on-boarding process. See "Using the Linked2 Configuration API" for more informtion on this.

`X-{clientName}-customerIdentfier`

If you include this header we will verify the header against information we hold as an extra validation step.

On receiving the webhook we will validate against the {id} and the HMAC digest by calculating the HMAC digest from the body of the request, base64 encoding the result and comparing it to the one contained in the header.

If all expected headers are present with expected contents the request will be accepted and you will receive an OK (200) response. If there is anything unexpected you will receive a Bad Request (404) response.

## Resource URLs
When you on-board a customer, create for them an integration pipeline, we will create a webhook receiver REST resource for their integration identified by the id we give you during this process. The id is a guid. You will store this guid and create your POST request resource URL using it.

`https://platform.linked2.io/api/webhooks/incoming/{clientName}/{id}`

This is important because your customers may have more than one integration and this id will tell us which integration pipeline the webhook body is intended for. Pipelines are specific to a single integration and a single customer. You cannot change this id. See "Using the Linked2 Configuration API" for details on this

## Webhook Topics and Events
Events could be customer created, customer updated, customer deleted, etc. Think about events as something that has happened, as opposed to commands which are something that is about to happen. It is important that a webhook only fires for successfully completed events (we do not wish to update another system on something that might happen).

## HMAC-SHA256 Digest
The HMAC-SHA256 digest allows us to verify with some confidence that the webhook has come from you - that it actually originated from who it claims to be originated from.

When you on-board a new customer we will generate a shared secret which you can store (along with the id we give you). Alternatively you can overwrite the shared secret we give you with your own and we will use the one you give us.

The secrets we generate are cryptographically secure, generated using the [NIST block cipher algorithm](https://www.cs.mcgill.ca/~kaleigh/computers/crypto_rijndael.html), which makes the secrets unpredicatble. The secret is base64 encoded. See "Using the Linked2 Configuration API" for more information on this.

Without access to the shared secret, if an attacker should intercept the webhook request all the attacker can do is replay the same request.

## Request Body
This can be any valid JSON. This is straight forward for simple business entities. Perhaps a simple product, or a simple contact with a single address. A contact record could look like this:
```json
{
    "customer-number" : "01234567",
    "first-name" : "Simon",
    "last-name" : "Templar",
    "street-address" : "",
    "city" : "",
    "postcode" : "",
    "country" : "",
    "email" : "",
    "phone" : ""
}
```

It is easy to imagine this being sent as the body of the webhook on an update or create event for this contact.

The biggest issues to consider for the body json arise when there may be mulitple objects that need to be sent for integration.

This can be quite common with orders, but also with customer entities where they may be related to mulitple addresses, phones, emails and information about tax, etc.

Using an order as an example. An order is typically made up from an order header, which references a customer and contains other global data for the order, plus order lines each of which references a product.

A JSON order may look something like this:

```json
{
    "order-number" : "000123456",
    "customer-id" : "000987234",
    "invoice-address" : "000129992",
    "currency" : "NOK",
    "total-price" : 532.45,
    "order-lines" :  [
        {
            "product-id" : "0006437823",
            "sku" : "qw-098763",
            "quantity" : 1,
            "vat" : 25.0
        },
        {
            "product-id" : "0006433321",
            "sku" : "as-0876222",
            "quantity" : 2,
            "vat" : 25.0
        },
    ]
}
```

a JSON order object with an array of order line objects. Lets say we wish to send this to an accounting system. The issue here becomes the references for customer, invoice address and product, plus the relationships in the data. The order -> order lines relationship is presented but the relationship of order to customer, invoice address and order lines to products are not. We have only internal identifiers plus the sku in the case of the product and we assume that the customer entity referenced in the order has a relationship to an invoice address.

The issue here, of course, is if any of the related entities do not exist in the accounting system this order cannot be processed into the accounting system. If the customer or invoice address or any of the products do not exist then the order integration will fail.

There are two strategies we can adopt to get around this.
1) We can present all the customer, address and product objects in the JSON along with the relationship. In other words we can completely describe the business entity with all the data required.

2) We can treat customer and product as seperate integrations and ensure those integrations are run first, before the order is run. Presumably we will have to cascade this down for any other relationships such as customer->address to get the invoice address.

Clearly the 2nd option has issues with timing, workflow, failures, etc so the first option is preferable.

Rather than re-invent the wheel for this we recommend you draw on the work done by [JSON API here](https://jsonapi.org/).

We are familiar with using JSON API which means we can develop your transformations drawing on experience of using this format and while it can look daunting there are [many JSON API tools available](https://jsonapi.org/implementations/) to help you convert your objects into JSON API format for sending. Using these tools will greatly simplify your coding and reduce your learning curve dramatically.

JSON API lets us send an aggregate of related entities in the request. That is a full description of all the entities involved and the data contained therein for a particular instance along with the relationships within the aggregate. We can transform that into the required target. In this scenario we are carrying out a real business level transformation involving several entities that relate to each other. A much more powerful technique than simply converting records separately where the relationship information is lost.

Transformation of these entities will likely mean a transformation into a completely different type of entity. In the case of an accounting system an order may be transformed into an outgoing invoice, or an accounting system may not have a concept of delivery address as seperate from billing address. But given a rich data object we can correctly transform one into the other.

## Webhook Testing
Webhook testing can be done in several ways, bear in mind that you don't need to test the resource only to test your webhook. This is especially true with Linked2 platform because we let you define your webhook (headers and body) as you need to. The guidance above is just that, *guidance*. So with this in mind you can test your webhook and trigger with any URL.

Check out [ngrok for webhook testing](https://ngrok.com/); this tool will let you redirect your webhook to a localhost address which is especially useful during development. ngrok is our preferred way of testing webhooks.

You can also find other tools available that are useful searching for webhook testing that will let you set up a temporary URL you can use and then via the website examine the body and headers of your webhook.

## Testing the Integration
Consider setting up a test customer that you can use for this purpose. When you have a product completed and installed it for the test customer you will be in a position to test your webhook plus integration pipeline.

You can set the customer pipeline into test mode, see "Using the Linked2 Configuration API" for more information on this. Then fire the webhook for the test customer and check the results vis the Notifications API. You will be looking for a publisher success notification that will state it is from the test run and if successful you will have a copy of the transformed result as it would be published to the target system.

Also check the notifications for any warnings or errors.
