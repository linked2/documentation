# Generalised API for configuration & setup of products.
Considering the issue of how to configure, too avoid having to develop specific UI for each product we create, and to provide a white label API for setup of a functioning integration pipeline.

## Language
### Product
A product is a definition of software integration product that can be installed on the Linked2 platform. Think of a product as any uninstalled software package

### Pipeline
A pipeline is a descriptive term for a configured live product.

### Stage
A stage is an installed and configured product.

### Filter
The part of stage functionality that is responsible for the validation of received data. A filter can check if an incoming document is as expected by checking it conforms to a formal schema and it can check that the document content is suitable for transformation (i.e. checking for status fields and similar).
A filter also acts as a router and routes the incoming document to one or more transformers depending on the content of the document.

### Transformer
The part of stage transforms the incoming document into the target format.

### User Defined Constant
A User Defined Constant may appear in the conditions of a filter or the mapping specification of a transformer. It is simply a constant value, but it can be set by the user during configuration setup or update

## Product Configuration and Setup 
A product will have at least one stage, and a stage will have at least one filter and at least one transformer.
A stage template is created which will be used to create configured stages as clients purchase new products and bring them on line.
A stage template has a template filter and a template transformer, fully configured & with mapping specifications fully defined and with example values
populated.

### User Defined Constant in Use
A user defined constant is just a function. It is a function that can be applied in filters and transformers.
It looks like this: `!udfConst("constantName", <constantValue> )` where the constant value can be a string, integer, decimal or boolean.
In fact a user defined constant is just the same as a normal constant using the function `!const( <constantValue>)` there is no difference.
The function name `!udfConst` is purely a mechanism to identify this constant as a user definable constant.
 
### Template Filter

A user defined constant in a filter will look like this, the `!udfConst` appears in the last filter condition:

Also notice that the filter has reference to the client, stage and product. We could say it is owned by a client, is part of a stage and
is operating as an instance of a product.

```
{
    "_id" : "749d0000-0194-1005-f0e2-08d5eec02e9d",
    "productId" : "749d0000-3375-7005-ea23-08d5eec07c10",
    "stageId" : "749d0000-0194-1005-f7a8-08d5eec02e9d",
    "clientId" : "749d0000-0194-1005-de3e-08d5eec02e9d",
    "filterVariables" : [],
    "filterConditions" : [ 
        {
            "filterPath" : "$.meta.presented",
            "match" : "\"HandlerForSampleWebhook\""
        }, 
        {
            "filterPath" : "$.meta.clientId",
            "match" : "\"749d0000-0194-1005-de3e-08d5eec02e9d\""
        }, 
        {
            "filterPath" : "$.meta.topic",
            "match" : "\"orders/create\""
        }, 
        {
            "filterPath" : "$.meta.host",
            "match" : "\"Space Walk Footwear\""
        }, 
        {
            "filterPath" : "$.included[?( @.type == 'order-status')].attributes.name.no",
            "match" : "!udfConstant(\"orderStatus\", \"Ikke behandlet\")"
        }
    ],
    "transformers" : [ 
        "749d0000-0194-1005-0be8-08d5eec3b6d9"
    ]
}
```

### Configuration
Now it is possible to identify user defined constants in the filter and transformer templates.
The jsonPath expression `$.filterConditions[?(@.match ==  "/\b!udfConst\b/")]` (or something similar, yet to be tested!)
will return an array of all filters that are using a `!udfConst` in the the match property.

Then doing a little string manipulation on the value of the match property we can extract the name of the constant (in the
example above that would be `orderStatus`). User defined constant names must be unique within a product.

I propose to use that name to look up the configuration of this constant.

Note, that there may be more than one user defined constant in match, for example in an `!or()` expression
```
!or( !udfConstant(\"orderStatus1\", \"Ikke behandlet\") ,!udfConstant(\"orderStatus2\", \"Behandlet\") )
```
### User Defined Constant Specification

Based on json schema notation. 

```
"orderStatus1" : {
    "type" : "string",
    "description" : "The status of the MyStore order, used to filter out updated order events.",
    "enum" : ["Pending", "Awaiting Fulfillment", "Awaiting Shipping", "Shipped", "Completed"]
},
"orderStatus2" : {
    "type" : "string",
    "description" : "The status of the MyStore order, used to filter out updated order events.",
    "enum" : ["Pending", "Awaiting Fulfillment", "Awaiting Shipping", "Shipped", "Completed"]
}
```

Using the example where we have the two `!udfConst` expressions in the `!or()` then our user defined constant specification would
need to have an specification for each of them.


### User Defined Constant and Product Definition

I suggesting that we store the user defined constant specifications with the other product data. In other words it seems that
user defined constants are part of the product definition.

## Product Setup & Configure API

The api needs to be able to deliver all the information needed by the calling app to configure the product into a live pipeline.

That will be any publication configuration and the user defined constants. So we pass a json object as in the example below and we 
expect to receive the same document back but fully populated.

We then take this to configure a products filters and transformers.

```
{
    "productId : "749d0000-0194-1005-f7a8-08d5eec02e9d",
    "stageId" : "749d0000-0194-1005-f7a8-08d5eec02e9d",
    "name" : "MyStore2PO",
    "description" : "MyStore orders to PowerOffice for WebhookClient",
    "clientId" : "749d0000-0194-1005-de3e-08d5eec02e9d",
    "client" : "WebhookClient",
    "culture" : "nn-NO",
    "group" : "MyStore2PO",
    "timeZoneId" : "Europe/Oslo",
    "publishContext" : {
        "clientId" : "749d0000-0194-1005-de3e-08d5eec02e9d",
        "publishContextId" : "749d0000-0194-1005-1065-08d5eec30971",
        "stageId" : "749d0000-0194-1005-f7a8-08d5eec02e9d",
        "publishTarget" : "PowerOffice",
        "testRun" : true,
        "context" : {
            "powerOfficeConfig" : {
                "apiKey" : "749d0000-0194-1005-df3f-08d5eeef198e",
                "clientKey" : "749d0000-0194-1005-df3f-08d5eeef198e"
            }
        }
    },
    "userDefinedConstants" : [
        {
            "name" : "orderStatus1"
            "schema" : {
                "type" : "string",
                "description" : "The status of the MyStore order, used to filter out updated order events.",
                "enum" : ["Pending", "Awaiting Fulfillment", "Awaiting Shipping", "Shipped", "Completed"]
            },
            "value" : "Shipped"
        },
        {
            "name": "orderStatus2"
            "schema" : {
                "type" : "string",
                "description" : "The status of the MyStore order, used to filter out updated order events.",
                "enum" : ["Pending", "Awaiting Fulfillment", "Awaiting Shipping", "Shipped", "Completed"]
            },
            "value" : ""
        },
        {
            "name": "AltAccountCode"
            "schema" : {
                "type" : "string",
                "description" : "The alternative account code",
                "pattern" : "(^[0-9]+$|^$|NULL)"
            },
            "value" : ""
        }
    ]
}
```

From top to bottom:

product & stage IDs are obvious.

name:  a name for the stage. We can let the client set this or we can leave it as a default.

description: the clients description, but again we can leave as default.

culture:  should be set by the client, but we can default to 'nn-NO' for Norway for now.

group:  is the clients setting. Again we can default. It is just a tag to help seperate, label and search for stages when there may be many.

timezone:  is IANA timezone. The user should choose but we can default to 'Europe/Oslo' for now.

publish:  context object is fairly clear, some repeated information here (clientId, etc) because this object can be detached and used separately.

testrun:  in the publish context will stop the publication from actually happening, although the results will be notified to the client.

userDefinedConstants array: contains the name, json schema and a value for the constant. The value is what the client selected to set this constant to.
Of course, initially values will be empty because they haven't been set.

schema:  By passing the json schema for the user defined constant to the calling app the calling app is able to make decisions about how to
ask for that information.

For the order status constants in the example the calling app can see it is collecting a string and the `enum` shows all allowable values.
The obvious way to implement this bit of UI would be with a drop down list.

a json schema can also include regular expression patterns so in the last example the pattern defines what is acceptable input. In this case
any number with at least one digit (no other characters except 0-9) or the word 'NULL' is allowed. The calling app can use this to validate the
input.

value : this will be the value that the client actually selects/enters.

When the call app has fully populated this configuration object it should be passed back to our api where it can be processed to create a functional
pipeline.

## JSON Schema

Using json schema for defining constants also create the possibility of generating forms on the fly from the schemas. There are a few 
open source json schema based form generator libraries available.

It would also be possible to include json schema for the rest of the configuration data. It is mostly standard.

For example
```
    "schema" : {
        "type" : "string",
        "description" : "Time zone",
        "enum" : ["Europe/Oslo", "Europe/Stockholm", "Europe/Amsterdam", "Europe/Athens", "Europe/Berlin", "Europe/Brussels"]
    }
```

## Using The API

The interaction, once authorised, would follow something like:

1) Send new client command, receive back client Id
2) Request product list, receive back list of products
3) Request product set up, receive back product configuration object
4) populate object
5) send new pipeline with configuration object.

In this way we also really provide a great white label api service.

Normally an api tells you nothing much about what how to use it. But with this proposal we send all the data we need to be returned,
we also tell the calling app what kind of data it is expecting to collect and even what that data looks like.

In other words because configurations can vary a lot in the data the they need to collect this presents a way of sending an object
that specifies all the data that is needed and defines how that data should look.