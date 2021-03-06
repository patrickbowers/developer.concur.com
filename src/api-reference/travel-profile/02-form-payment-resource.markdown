---
title: Form of Payment
layout: reference
---


## Description
The Form of Payment Web service consists of a set of resources that provide form of payment details customized in specific ways for developers, travel suppliers, and travel management companies (TMCs).

Developers, travel suppliers, and travel management companies (TMCs):

* View user’s personal methods of payment
* View user’s corporate methods of payment
* Will only be included if Corporate Ghost Card scope has been enabled
* Update credit card data


## Version
2.0
[Version 1.0](/api-reference-deprecated/version-one/Travel/form-payment-resource.html) has been **deprecated**

## URI
    https://{InstanceURL}/api/travelprofile/v2.0/fop

## Who can use this resource?
This endpoint can be used by travel suppliers or travel management companies (TMC). The scope of information returned varies depending on who makes the request.

## Operations
* [Get preferred method of payment details](#a1)
* [Create/Update method of payment details](#a2)
* [Data Model](#a3)
* [Possible Warning and Error Messages](#a4)
* [Account Number Validation](#a5)

## <a name="a1">Get preferred method of payment details</a>
This endpoint can be used by travel suppliers or travel management companies (TMC) to get the preferred method of payment details for the specified user. The scope of information returned varies depending on the entity making the request.

### Headers

#### Content-Type header
application/xml

#### Authorization header
Authorization: OAuth {access_token}

Where access_token is the OAuth 2.0 access token of the user whose travel credit card information you want to retrieve or update.

## <a name="a2">Create/Update method of payment details</a>
This endpoint can be used by travel suppliers or travel management companies (TMC) to update the method of payment details for the specified user.

### Helpful Tips
* Corporate (Ghost) Cards cannot be updated using this service
* Existing Cards will be updated (based on Vendor and Account Number), others will be created
	* **Cards not passed in, that did exist, will be DELETED**
	* **Perform a Get before Post in order to verify existing cards status to prevent unintentional deletions**
* Marking a card as default for a specific Segment will make other cards no longer default for the segment


### Headers

#### Content-Type header
application/xml

#### Authorization header
Authorization: OAuth {access_token}

Where access_token is the OAuth 2.0 access token of the user whose travel credit card information you want to retrieve or update.


## <a name="a3">Data Model</a>

The schema for v2.0 is available [here.](https://www.concursolutions.com/ns/FormOfPayment.xsd)

The root element contains the following attribute:

Name | Type | Format | Description
-----|------|--------|------------
`unique`    |   `string`  |   `-` |   The user’s unique identifier, associated with loyalty information if accessed by a vendor who has provided that information.


## CreditCard Elements

Name | Type | Format | Description |
------------|-----------------|---------|-------------|
`DisplayName` | `string` |`-` |Display name associated with the card.|


The CreditCard element contains the following child elements:

Name | Type | Required | Description |
------------|-----------------|---------|-------------|
`Vendor` |`string` |`Creation` |The card vendor. [See Reference][1] for list of vendors |
`AccountNo` | `string`|`Always` |The credit card account number. |
`ExpDate` |`date/time`|`-` |The expiration date of the credit card. Format: YYYY-MM |
`NameOnCard` | `string` |`Creation` |The name on the credit card. **Business Cards only.** |
`UsageType`|`string` |`-` |For what purpose the card is to be used, which will be one of the following values: **Corporate**, **Business** |
`BillingAddress` |`string`|`Required` |This parent element contains information about the billing address. For information about the child elements of this parent element, see the **BillingAddress element** table below. |
`Segments`|`string`|`Required` |A list of segments with which the card may be used. For information about the child elements of this parent element, see the **Segment element** table below

#### BillingAddress element

Element Name|Type|Required|Description|
------------|-----------------|---------|-----------|
`StreetAddress` | `string`|`Required`  |The street and unit information for the billing address.|
`City` | `string`| `Required` |The city information for the billing address.|
`StateProvince` | `string`| `Required` |The state or province information for the billing address.|
`Country`| `string`|`Required`  |The country information for the billing address.|
`ZipCode`| `string`| `Required` | The zip code information for the billing address.|

#### Segments element

Element Name|Type|Format|Description|
------------|-----------------|---------|-----------|
`Type` | `string`|`Required`  |Type of Segment, which will be one of the following values: Air, Rail, Hotel, Car, Ground|
`Mandatory` | `boolean`| `-` |A Boolean that notes if this card must be used for payment for this segment type. **Corporate Ghost Cards only.** |
`Default` | `boolean`| `Required` |A Boolean that notes if this card has a default use for payment for this segment type. |


### Examples for Travel Suppliers

#### Example 1: Get forms of payment for the user associated with the specified OAuth 2.0 access token
**Request**

```http
GET {InstanceURI}/api/travelprofile/v2.0/fop HTTP/1.1
Authorization: OAuth {access token}
...
```

#### Example 2: Set forms of payment for the user associated with the specified OAuth 2.0 access token
**Request**

```http
POST {InstanceURI}/api/travelprofile/v2.0/fop HTTP/1.1
Authorization: OAuth {access token}
...
```

#### Sample XML Input

```xml
<CorporateFOPResponse xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" LoginId="developer@concur.com">
    <CreditCards>
        <CreditCard DisplayName="My Visa Card">
            <Vendor>Visa</Vendor>
            <AccountNo>4111111111111111</AccountNo>
            <ExpDate>2020-09</ExpDate>
            <NameOnCard>Concur Developer</NameOnCard>
            <UsageType>Business</UsageType>
            <BillingAddress>
                <StreetAddress>123 Home St</StreetAddress>
                <City>Bellevue</City>
                <StateProvince>WA</StateProvince>
                <Country>US</Country>
                <ZipCode>98006</ZipCode>
            </BillingAddress>
            <Segments>
                <Segment Type="Air" Default="false" />
                <Segment Type="Hotel" Default="false" />
                <Segment Type="Car" Default="false" />
                <Segment Type="Rail" Default="false" />
            </Segments>
        </CreditCard>
    </CreditCards>
</CorporateFOPResponse>
```

## <a name="a4">Possible Warning and Error Messages</a>

Error Messages|Possible Issues|
------------|-----------------|
`There is an error in XML document` | Invalid Vendor, UsageType, or Segment Type attempted to pass in, Invalid extra elements in the XML, or Other XML formatting issues|
`You must specify at least one credit card to add or update` | An empty list of CreditCards is being supplied |
`Cannot update Corporate (Ghost) cards using this service` | Attempting to update a card with `UsageType = Corporate` |
`Cannot update Mandatory field for Corporate (Ghost) cards using this service.` | This field is only used by Ghost cards, which cannot be updated using this service.|
`Only one segment of a particular type can be provided for each Credit Card.` | Duplicate segments are being supplied to an individual credit card (ie multiple car segments) |
`You do not have permissions for element: {type}` | An attempt is being made to update a card of a conflicting vendor type |
`Forbidden Request` | The entity trying access the Form of Payment endpoint does not have the proper permissions. |
`Invalid Account Number` | Account Number check failed due to prefix, length, luhn, or other required format [See Reference][1] |

[1]: /api-reference/travel-profile/99-reference-resource.html#account-number-validation