---
title: Buy4Now Mobile API Reference

language_tabs:
  - Examples

toc_footers:
  - Copyright © 2012-2015 Buy4Now
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---
# Overview

This document provides details on the Buy4Now eGrocery Mobile API.

The API itself will be exposed by a HTTP-based interface which will implement elements of the Representation State Transfer (REST) operation methodology. The interface will use commands and data (parameters) embedded in query strings and / or in posted JSON data.

All responses from the API operations will be in the form JSON serialised objects. Exact specs on these JSON objects (and querystring information) can be viewed in more detail in the API Operations section of this document.

URL of the service listed in the Web Service Operation examples below is not the Live URL; it is in fact our public facing test environment. The Live URL will be made available when we’re much closer to the go live date.

* No anonymous calls can be performed without the application key.
* No customer access level calls can be performed without the session key.
* No credit card information will be processed by the mobile applications/API.

SSL will be required for both Login and Cancel Order operations

# Definitions

_Session Key_: Unique key used to identify a customer and his current session, the session key is returned to the application as the result of a successful Login call. This will be an AES (Rijndael) encoded GUID, probable length of 96 chars.

_Application Key_: Used to identify the application connection to the API service (unique application / OS). Applications will not be allowed to perform any actions without a valid application key.

_Basket_: Place where customer stores items when shopping in the APP, this basket will be shared with the web applications, anything added to the basket on the app will immediately appear in the customer’s basket on the website if refreshed and visa versa.

# API Operations

For the purpose of prioritising the development of the API in line with the parallel development of the app(s) themselves, the APIs have being organized in this section in the order they must be delivered and made available for use.

# Stage 1 - Home
## AppInit
```
http://shop.supervalu.ie/API/[VersionNumber]/AppInit.asmx/GetInfo
```
> JSON request data:

```json
{
  "appKey": "XYXYXYXY",
  "os": "5",
  "appVer": "1",
  "uuid": "23452345",
  "imagePackVersion": 1,
  "pickupLocationVer": 10
}
```
> JSON response data:

```json
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "Success",
    "appVer": "2.0.0",
    "CurrentImagePackVersion" : 22,
    "ProductPageSize": 40,
    "Images":
      [
        "http://supervalu.buy4nowbeta.com/shopping/images/banner/newimage.png",
        "http://supervalu.buy4nowbeta.com/shopping/images/banner/newimage2.png"
        ...
      ]
  }
}
```
### Purpose
This will be used to perform three tasks...

1. Verify the application is still valid and usable against the API.
2. Send a list of images needing to be updated on the app depending on the ImagePackVersion number sent.
3. Transmit the current ImagePackVersion number.

### API Keys

 Environment | Key
:------------|:----
 DEV Android AppKey  | c1d4688b-1585-4bf4-b2fa-a62735e29548
 LIVE Android AppKey | 2bd84f2f-fbfb-43c7-bf2a-91550c4d6b05
 DEV iPhone AppKey   | 02f514d8-36b3-48f6-8e27-b4155a99e2de
 LIVE iPhone AppKey  | 2182c0a9-acc0-466e-b6e8-fadc87b69762

## Login

```
https://shop.supervalu.ie/API/[VersionNumber]/Session.asmx/Login
```

> JSON request data:

```
{
  "appKey": "XXXXXXXXXXXXXXXXXXXX",
  "userEmail": "test@test.com",
  "pwd": "MyPasswordIsUsuallyMoreComplexThanThis"
}
```

> JSON response data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "Success",
    "SessionKey": "YYYYYYYYYYYYYYYYYYYY"
  }
}
```

### Purpose
Application will use this operation to log the user into the shopping system. A session key will be returned if the login is successful; this key will be used for the life of the users’ session and will identify the user for all commands were user information is transmitted.

The API will record the amounts of failed login attempts against each customer, if this count exceeds a set amount, the account will become locked out. Customers account becomes locked out for a set period of time (e.g. 30 secs).

Everytime the customer performs a Login action in the app, the sessionID will be refreshed, the login action in the mobile web section (aka checkout) will not refresh the sessionID

 QueryString Name | Value
:-----------------|:-----
 operation        | login
 appKey           | This value will define what application is attempting to connect and what release version.

 Parameters Name | Parameter Value
:----------------|:---------------
 userEmail       | The email address of the customer attempting to log in.
 pwd             | The password associated with the customer’s account.

 Return Parameter Name | Return Parameter Value
:----------------------|:----------------------
 responseCode          | Success/fail code
 responseInfo          | Text description of the responseCode
 sessionKey            | The sessionKey is a value used to indentify a correctly logged in customer. Without these key requests of customer information will not be successful.

## Logout

```
http://shop.supervalu.ie/API/[VersionNumber]/Session.asmx/logout
```

> JSON request data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```


> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "Success"
  }
}
```

### Purpose
Checks to see the provided session is still valid.


## GetHomePageInfo

```
http://shop.supervalu.ie/API/[VersionNumber]/Home.asmx/GetInfo
```

> JSON Post Data:

```
{    "sessionKey": "XXXXXXXXXXXXXXXXXXXX"}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "Success",
    "FirstName":  "Frank",
    "StoreID" : 322,
    "StoreName" : "Hurleys of Midleton",
    "SlotTime" : "20111020 14:00",
    "SlotEnd" : "20111020 15:00",
    "SlotType" : "Pickup",
    "SlotPrice" : 3.00,
    "LocationID": 17,
    "SlotMethod": 0,
    "AddressName" : "Home",
    "BasketTotalPrice": 42.60,
    "BasketItemQty": 25,
    "CurrentOrderRef" : 12312,
    "CurrentOrderTotalBasketPrice": 52.60,
    "CurrentOrderItemQTY": 27,
    "CurrentOrderSlotTime" : "20110020 14:00",
    "CurrentOrderEndTime" : "20110020 15:00",
    "CurrentOrderSlotType" : "Pickup",
    "PreviousOrder": "true"
  }
}
```

### Purpose
Returns information used to display the homepage.

If the customer has chosen a slot in the past, but the slot reservation has since expired, then no slot information will be returned and the customer should be forced to choose a slot before they can check out.

If a customer has no valid delivery addresses, then the list will be empty and the api will also return a response code of 12. A customer can register an address for which there is no store able to fulfil to that address as yet. Typically such customers can only then use the pickup service.

If storeID is NULL for the customer record, the system cannot create a basket for the customer and they need to select a valid shopping mode to have a store assigned. Getting a valid shopping mode can be done by selecting another valid address or selecting a pickup location. In this scenario the  getHomepageInfo api returns a 0 storeID and also returns responseCode 11, which tells the user to <code>"You must first book a slot to continue using the application"</code>

<aside class="notice">
In the JSON response data for the GetHomePageInfo call:<br>
`"StoreName" : "Hurleys of Midleton"` - Displayed at top of homepage in welcome<br>
`"AddressName" : "Home"` - contains Store Pickup Location Name, if slotType='Pickup'<br>
`"CurrentOrderTotalBasketPrice"` - includes service fee, same value in getMyOrders<br>
`"PreviousOrder": "true"` - false means customer has no previous orders
</aside>

## CheckSession

```
http://shop.supervalu.ie/API/[VersionNumber]/Session.asmx/CheckSession
```

> JSON Post Data:

```
{
    "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "Success"
  }
}
```
### Purpose
Checks to see the provided session is still valid.

If the session is no longer valid the customer should be directed to log in again to acquire a new session.

## CheckSlot

```
http://shop.supervalu.ie/API/[VersionNumber]/Session.asmx/CheckSlot
```

> JSON Post Data:

```
{
    "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

> JSON Response Data:

```
{
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
}
```

###Purpose
Checks to see if the current selected slot is still valid for placing an order.

A slot can only be reserved for a set period of time, also, each slot has a cut off time associated with it. If the cut off time is passed the reserved slot is released. If no slot is reserved or if the slot is no longer valid, a return code of 21 will be passed back in the response.
 

## ResetPassword

```
http://shop.supervalu.ie/API/[VersionNumber]/Session.asmx/ResetPassword
```

> JSON Post Data:

```
{
  "email": "forgetful@customer.com"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "Success"
  }
}
```
 
### Purpose
Gets the API to send an email to the customer with a new password (assuming the email address provided is already in the system).

The response code of success will be given regardless of the existence of this email in the backend systems; the reason for this is we don’t want to provide a potential brute force operation of extrapolating stored email addresses in the system.

# Stage 2 - Shop
## GetPickupLocations

```
http://shop.supervalu.ie/API/[VersionNumber]/logistics.asmx/GetPickupLocations
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "Version": 10,
    "Areas":
    [
      {
        "PickupAreaID": 1,
        "areaName": "Dublin South"
      },
      {
        "PickupAreaID": 2,
        "areaName": "Cork Somewhere"
      }
    ],
    "Stores":
    [
      {
        "PickupID": 1,
        "StoreID": 322,
        "ZoneID": 1,
        "StoreName": "Ballingcollig",
        "PickupAreaID": 2
      },
      {
        "PickupID": 2,
        "StoreID": 322,
        "ZoneID": 2,
        "StoreName": "Near Ballingcollig",
        "PickupAreaID": 2
      },
      {
        "PickupID": 3,
        "StoreID": 344,
        "ZoneID": 1,
        "StoreName": "Goatstown",
        "PickupAreaID": 1
      }
    ]
  }
}
```

### Purpose
Gets the list of active pickup areas and all of their associated stores, this to be used when selecting a pickup store.

## GetDeliveryAddresses

```
  http://shop.supervalu.ie/API/[VersionNumber]/logistics.asmx/SetDeliveryAddress
```

> JSON Post Data:

```
{
    "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "addresses":
    [
      {
        "ResponseCode": 0,
        "ResponseInfo": "SUCCESS",
        "AddressID": 345345,
        "AddressName": "Home",
        "HouseName": "25 Fake Street",
        "Addr1": "Faketown",
        "Addr2": "",
        "Addr3": "Cork City",
        "Addr4": "",
        "County": "Co. Cork",
        "Country": "Ireland",
        "PrimaryAddress": "true"
      }
    ]
  }
}
```

### Purpose
Get a list of delivery addresses associated with the customers session.

If a customer has no valid delivery addresses, then the list will be empty and the api will also return a response code of 12. A customer can register an address for which there is no store able to fulfil to that address as yet. Typically such customers can only then use the pickup service.


## GetSlots

```
http://shop.supervalu.ie/API/[VersionNumber]/slot.asmx/GetSlots
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "method":  1,
  "locationID": 456,
  "dayDelta": 0,
  "days":  7
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "SlotDays":
    [
      {
        "Date": "20111123",
        "Slots":
        [
          {
            "SlotID": 28649,
            "Status": 0,
            "Start": "18:00",
            "End": "19:00",
            "Price": 8.00
          },
          {
            "SlotID": 28650,
            "Status": 0,
            "Start": "19:00",
            "End": "20:00",
            "Price": 8.00
          }
        ]
      }
    ]
  }
}
```

### Purpose
Gets the slots for the customer in session (will filter slots by either delivery address or pickup store)

## SetSlot

```
http://shop.supervalu.ie/API/[VersionNumber]/slot.asmx/SetSlot
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "method":  1,
  "locationID": 456,
  "slotID": 23322
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "StoreID": 322
  }
}
```
### Purpose
Sets slot for the customer in session.

This clears existing chosen slots against the sessions customer.

## GetCategoryTree

```
http://shop.supervalu.ie/API/[VersionNumber]/Assortment.asmx/GetCategoryTree
```

> JSON Post Data:

```
{
    "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
    "type": "S",
    "cacheDate": "20111029 20:19",
    "cacheStoreID": 322
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "Date": "2011029 20:00",
    "Store": 327,
    "Categories":
    [
      {
        "CategoryID": 100,
        "CategoryName": "Fresh Foods",
        "ParentID": 0,
        "Prority": 3,
        "DisplayImage" : 1
      },
      {
        "CategoryID": 1001,
        "CategoryName": "Fruit",
        "ParentID": 100,
        "Prority": 27,
        "DisplayImage" : 0
      },
      ...
    ]
  }
}
```

### Purpose
Used to pull down the entire category tree for a store.

Needs to verify the cacheStoreID is the same as that stored in the customers sessions, if the storeIDs do not match, then the cache is no longer valid and the Category Tree will be transmitted in full (for that store).

The API will also use the supplied cacheDate to determine if the cache has expired, if it has, then the entire category tree for the customers selected store is transmitted.

Please Note: The cacheStoreID does not reflect the current storeID the customer is using, it is the storeID associated with the currently cached category tree.

The priority field in the category list is used to order the way in which the categories are listed on device with same parentid. The displayimage flag is an instruction to the device to look for a locally stored image on the device for the given category.

Response code 11 can be returned in this API if the customer record has not got a valid storeid assigned, in which case the app will trigger and error
 
## GetProductListing

```
http://shop.supervalu.ie/API/[VersionNumber]/Assortment.asmx/GetProductListing
```

> JSON Post Data:

```
{
    "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
    "type": 1,
    "filter": "2501",
    "listFrom": "",
    "listTo": "",
    "cachedDate": "20111120 20:00"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "ResultCount": 45,
    "Date": "20111119: 19:00",
    "ImagePath": "http://shop.supervalu.ie/shopping/images/products",
    "Products":
    [
      {
        "ProductID": 234545,
        "ProductName":  "Fernys Toasting Bread",
        "SmlImg": "234545_1.jpg",
        "MedImg": "234545_2.jpg",
        "LrgImg": "234545_3.jpg",
        "ProductType": 1,
        "UnitPrice": 1.20,
        "UnitOfMeasure": "kg",
        "PriceDesc": "(€1.20 each) ",
        "Qty" : 0,
        "Note": "Fresh Bread please",
        "Favorite": false,
        "PromotionBulletText": "Save €1.40",
        "PromoDesc": "Was €2.99 Now €1.20 Save €1.40",
        "PromotionID": 2342,
        "PromotionCountProducts": 2,
        "PromotionGroupID" : 1202,
        "PromotionGroupName" : "Main Course",
        "PromotionStartDate": "2011014 01:00",
        "PromotionEndDate": "2011114 01:00"
      },
      { . . . next product . . .},
      { . . . next product . . .}
    ]
  }
}
```

### Purpose
The operation will return a list of products, the source of this list can be from several locations (Category, Search, Promotions, My Usual’s).

#### Type
##### 0: Normal Category Browsing
The categoryID should be inserted into the filter field. e.g.

`"type": "0",`

`"filter": 2150`

Such listings are called from leaf nodes on the getCategoryTree where type = ‘N’

#### 1: Specials Category Browsing
The categoryID should be inserted into the filter field. e.g.

`"type": "1",`

`"filter": 120`

Such listings are called from leaf nodes on the getCategoryTree where type = ‘S’

#### 2: Search Results Listing
The search term should be inserted into the "filter" field e.g.

`"type": "2",`

`"filter": "apples"`

#### 3: Promotion Products Listing
The promotionID and the productID should be inserted into the  filter field ("[productID],[promotionID]")

`"type": "3",`

`"filter": "100345343,120324" /*100345343 being the productID and 120324 being the promotionID */`

#### 4: My Usuals Listing
The list needs to be cached, to check to see if the MyUsuals cache is still valid, a cached date is transmitted as part of the request. If the cache is deemed to have expired (or no date was supplied) then a response code of 0 is sent with a Date (to be stored as the cached date) and a list of products.
If the cached date is deemed to still be valid, then a response code of 1 is transmitted back and no product info is sent.

`"type": "4",`

`"filter":    /* ignored when type 4 submitted */`

#### Paging
Paging is access in all operation s that return products (excluding active/previous orders and viewing the basket). This is achieved by specifying the index range of the items you want in your page E.g. First Page <1..40> Second Page <41..80>

Currently, paging should only be used in the Search Results screen.

#### Image URL Construction
[ImagePath]/[ImageSize]/[ImageFileName]

Property                      | Value
:-----------------------------|:-----
ImagePath (API header)        | http://shop.supervalu.ie/shopping/images/products
ImageSize (determined by app) | small
ImageName (API detail)        | 234545_1.jpg

Should build the below URL
<code>http://shop.supervalu.ie/shopping/images/products/small/234545_1.jpg</code>

#### Product Types
Extract from BRS for product type supplied in the API for each item returned

Product Type | On The Website | On The App
:------------|:---------------|:----------
1            | Each           | Appears as normal each item
2            | Weighted       | Each Appears as normal each item (unlike the website where options are available to buy the item in either way). The API will simply hand such items back as each items and the app really doesn’t know the difference between a type 1 and 2 item
3            | Weighted       | Appears as weighted item with modal popup for the quantity selector

QueryString Name | Value
:----------------|:-----
Operation        | ProductListing
sessionKey       | The session key associated with the customers current session on this app.

Parameter Name | Parameter Value
:--------------|:---------------
Type           | The type of the product Listing we want to return <br>* Category (0): Returns a normal category Listing<br>* Specials (1): Returns a normal category Listing<br>* Search (2): Returns the results of a product keyword search.<br>* Promotion (3): Returns the list of products associated with one promotion (should not be used for group promotions).<br>* MyUsuals (4): Returns the list of products in the customers MyUsuals list
Filter         | The filter has different meanings based on the Type specified.<br>Category (0)  CategoryID<br>Specials (1)  CategoryID<br>Search (2)  Search Term<br>Promotion (3) PromotionID<br>myUsuals (4)  [NOT USED]

 
# Stage 3 – Checkout
This stage also covers the development of the product detail screen, search and main trolley. All checkout APIs have now being removed from scope and these screens are being addressed through an embedded set of mobile web screens within the application.

## GetProductInfo

```
http://shop.supervalu.ie/API/[VersionNumber]/Assortment.asmx/GetProductInfo
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "productID": 1023437,
  "barcode": ""
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ProductID": 1023437,
    "ProductName":  "Fernys Toasting Bread",
    "ProductType": 1,
    "UnitPrice": 1.20,
    "UnitOfMeasure": "kg",
    "PriceDesc": "(€1.20 each) ",
    "Qty" : 0,
    "Note": "Fresh Bread please",
    "PromotionBulletText": "Save €1.40",
    "PromoDesc": "Was €2.99 Now €1.20 Save €1.40",
    "PromotionID": 2342,
    "PromotionCountProducts": 2,
    "PromotionQualifierID" : 1202,
    "PromotionQualifierName" : "Main Course",
    "PromotionStartDate": "2011014 01:00",
    "PromotionEndDate": "2011114 01:00"
  }
}
```

###Purpose
Returns a single product from either its productID or its barcode.
Barcodes should be EAN13, EAN8, UPC-A, UPC-E or ITF-14

NOTE: image information not part of this API response as it will be pulled as part of a

## SetQuantity

```
http://shop.supervalu.ie/API/[VersionNumber]/Basket.asmx/SetQuantity
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "productID": 2342332,
  "qty": 5
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "BasketTotalPrice": 42.60,
    "BasketQty": 25
  }
}
```

### Purpose
Adds or Removes a products quantity. Setting the qty to 0 will remove the product from the basket.

## SetNote

```
http://shop.supervalu.ie/API/[VersionNumber]/Basket.asmx/SetNote
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "productID": 2342332,
  "note" : "fresh apples please"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Adds or Removes a products quantity. Setting the qty to 0 will remove the product from the basket.


## AddFavourite

```
http://shop.supervalu.ie/API/[VersionNumber]/Assortment.asmx/AddFavourite
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "productID" : "productID"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Adds a favourite from the customers list


## RemoveFavorite

```
http://shop.supervalu.ie/API/[VersionNumber]/Assortment.asmx/RemoveFavourite
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "productID" : "productID"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Removes a favourite from the customers list

## RemoveUsual

```
http://shop.supervalu.ie/API/[VersionNumber]/Assortment.asmx/RemoveUsual
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "productID" : "productID"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Removes a Usual from the customers list

## GetBasketOrderListing

```
http://shop.supervalu.ie/API/[VersionNumber]/Basket.asmx/GetBasketOrderListing
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

>  JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "BasketTotalPrice": 42.60,
    "BasketItemQty": 25,
    "Products":
    [
      {
        "ProductID": 234545,
        "ProductName":  "Fernys Toasting Bread",
        "ProductType": 1,
        "UnitPrice": "1.20",
        "PriceDesc": "(€1.20 each) ",
        "UnitOfMeasure": "kg",
        "Qty": 25
      }
    ]
  }
}
```

### Purpose
Lists the Product contents of the customer’s basket. Response code 11 can be returned in this API if the customer record has not got a valid storeid assigned, in which case the app will trigger and error

## ClearBasket

```
http://shop.supervalu.ie/API/[VersionNumber]/Basket.asmx/ClearBasket
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Clears all items from the customers shopping basket.

## SetAllowSubPolicy

```
http://shop.supervalu.ie/API/[VersionNumber]/Basket.asmx/SetAllowSubPolicy
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "productID": 2342332,
  "allow" : "true"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Sets the Substitution policy of items in the customer’s basket. This can only be performed on items in a basket.

# Stage 4 – Active Order
## GetMyOrders

```
http://shop.supervalu.ie/API/[VersionNumber]/CustomerOrder.asmx/GetMyOrders
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "Orders":
    [
      {
        "OrderRef": 232323,
        "ItemQty": 45,
        "OrderValue": 42.60,
        "SlotStart": "20111020 09:00",
        "SlotFinish": "20111020 11:00",
        "DeliveryType": "pickup",
        "LiveOrder": true,
        "AllowAmendPayment": true
      },
      {
        "OrderRef": 232322,
        "ItemQty": 50,
        "OrderValue": 58.65,
        "SlotStart": "20109020 09:00",
        "SlotFinish": "20111020 11:00",
        "DeliveryType": "delivery",
        "LiveOrder": false,
        "AllowAmendPayment": false
      }
    ]
  }
}
```

### Purpose
Gets the list of customer’s previous orders. A previous order which is still active is flagged by the "liveOrder" attribute being set to a value of "true". An active order, is another which has not yet reached the time before a slot starting for which changes are still allowed to that order.

<aside class="notice">
In the JSON response data for the GetMyOrders call:<br>
`"LiveOrder": true` - indicates that this is an active order.<br>
`"LiveOrder": false` - indicates that this is a normal previous order and changes are no longer allowed.
</aside>
 
## GetMyOrderInfo

```
http://shop.supervalu.ie/API/[VersionNumber]/CustomerOrder.asmx/GetMyOrderInfo
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "orderRef": 232323
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "OrderRef": 232323,
    "OrderValue": 42.60,
    "ItemQty": 45,
    "StoreID": 831,
    "SlotStart": "20111020 09:00",
    "SlotFinish": "20111020 11:00",
    "LocationID": 17,
    "SlotMethod": 0,
    "DeliveryType": "pickup",
    "DeliveryLocation": "Pickup Store Name or Address Name",
    "ChargeType": "Visa",
    "ChargeIdentifier": "********* 4567",
    "LiveOrder": true,
    "AllowAmendPayment": true,
    "CurrentBasketPrice": 42.60,
    "CurrentBasketQty": 45,
    "Items":
    [
      {
        "ProductID": 234545,
        "ProductName":  "Fernys Toasting Bread",
        "ProductType": 1,
        "UnitPrice": "1.20",
        "PriceDesc": "(€1.20 each) ",
        "UnitOfMeasure": "kg",
        "Qty": 25
      }
    ]
  }
}
```

### Purpose
Lists the items and details of a customer’s previous order

<aside class="notice">
In the JSON response data for the GetMyOrderInfo call:<br>
`"OrderValue": 42.60` - Estimated value of this order<br>
`"ItemQty": 45` - Quantity of items in this order<br>
`"ChargeIdentifier": "********* 4567"` - shown on website for card #?<br>
`"CurrentBasketPrice": 42.60` - value of current basket – for use in add items popup<br>
`"CurrentBasketQty": 45` - qty of items in current basket – for use in add items popup
</aside>

## AddMyOrderToBasket

```
http://shop.supervalu.ie/API/[VersionNumber]/CustomerOrder.asmx/AddMyOrderToBasket
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "orderRef": 1231231
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "BasketTotalPrice": 50.12,
    "BasketQty": 23
  }
}
```

### Purpose
Adds my previous (non-active) order to the current basket of items.

<aside class="notice">
In the JSON response data for the AddMyOrderToBasket call:<br>
`"orderRef": 1231231` - Previous order reference, take these items and add to basket
</aside>

## CancelOrder

```
https://shop.supervalu.ie/API/[VersionNumber]/CustomerOrder.asmx/CancelOrder
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "orderRef": 1231231,
  "password": "IAmAPassword"
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Cancels an active order.

## AddBasketToActiveOrder

```
http://shop.supervalu.ie/API/[VersionNumber]/CustomerOrder.asmx/AddBasketToActiveOrder
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "orderRef": 1231231
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Adds the current basket to an existing order.

## ActiveOrderUpdateItems

```
http://shop.supervalu.ie/API/[VersionNumber]/CustomerOrder.asmx/ActiveOrderUpdateItems
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "orderRef": 1231231,
  "items":
  [
    {
      "productID" : 5343,
      "itemQty": 10
    },
    {
      "productID": 5643,
      "itemQty": 0
    }
  ]
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```
 
### Purpose
Updates the quantity of all items currently in the active order

<aside class="notice">
In the JSON post data for the ActiveOrderUpdateItems call:<br>
`"itemQty": 0` - Setting the itemQty to 0 will remove the item from the order
</aside>

## ActiveOrderChangeLogistics

```
http://shop.supervalu.ie/API/[VersionNumber]/CustomerOrder.asmx/ActiveOrderChangeLogistics
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "orderRef": 1231231,
  "deliveryMethod":  1,
  "locationID": 456,
  "slotID": 1231
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

### Purpose
Allows customer to select a different slot to the one originally selected when the order was placed. And allows the customer to switch mode from the original order, as part of updating to the new slot. This information is all passed into Shop4Now via the post data.

deliveryMethod | LocationID Meaning
:--------------|:------------------
0: Pickup      | PickupAreaID
1: Delivery    | AddressID

<aside class="notice">
In the JSON post data for the ActiveOrderChangeLogistics call:<br>
`"locationID": 456` - refers to either PickupAreaID or AddressID depending on deliveryMethod
</aside>

# API – Response Codes
Each response JSON object will start with a field called ResponseCode, these response codes will be accompanied by a field called response Info, this field will explain the code from the previous field, below are some examples of Response Codes and their meanings…

Code | Response Info
:----|:-----------------------
0    | Request Successful.
1    | Successful, cache still valid.
11   | Success, no store is session.
12   | Success, no valid delivery addresses returned.
20   | Slot Full.
21   | Invalid ID, no data
22   | Invalid Logistics selection.
30   | Order not found
31   | Unable to cancel order
32   | Unable to amend order.
40   | Invalid promotions, vouchers, alcohol or "other" on the order.
50   | Invalid qty for this product.
51   | Product not found.
100  | Login Error.
101  | Login locked out
102  | Account configuration error
103  | Invalid Email Address Format.
110  | Session Expired.
111  | Session Key Invalid.
200  | AppKey is not valid.
201  | AppKey and OS version do not match. Force user to upgrade to use app
300  | Not Implemented
900  | Unknown internal error
901  | Request Timeout
902  | Request Unknown

 
# Native App & Mobile Web Screen Interaction
## Embedded Full Screen Mobile Web Screen and Header Bar
The following set of javascript calls are made by the app when interacting with the embedded mobile web screens it uses

Embedded Mobile Web Screen | UAT URL | Parameters passed in by app | Display on Left (Back) | Display on Left (Logout) | Display on Right (Continue)
:--------------------------|:--------|:----------------------------|:-----------------------|:-------------------------|:---------------------------
Checkout Login | /shopping/Checkout/CheckoutLogin.aspx?mode=m | App Session Key | N | N | N
Checkout 1 | /shopping/Checkout/CheckoutLogistics.aspx | | N | N | N
Checkout 2 | /shopping/Checkout/CheckoutFinal.aspx | | Y | N | N
Checkout Confirmation | /shopping/Checkout/CheckoutConfirmation.aspx | | N | Y | Y (Home)
Active Order Change Payment Login | /shopping/Checkout/CheckoutLogin.aspx?mode=m <br>(same as normal checkout login but orderref javascript injected for this function instructing the website to know this is a change payment request and not a standard login) | App Session Key <br>Order Ref | | N | N | N
Active Order Change Payment | /shopping/MyAccount/PreviousOrderChangePayment.aspx | | N | N | N
Active Order Change Payment Confirmation | /shopping/MyAccount/PreviousOrderChangePaymentConfirmation.aspx | | N | Y | Y (Active Order Home)
Registration 1 (Loyalty or Not) | /shopping/StartShopping/Register.aspx?mode=m | | N | N | N
Registration 2a (Loyalty Entry) | /shopping/StartShopping/RegistrationWithRewardCard.aspx | | Y | N | N
Registration 2b (Full Entry) | /shopping/StartShopping/RegistrationWithoutRewardCard.aspx | | Y | N | N
Registration 3 (Confirmation) | /shopping/StartShopping/RegisterConfirmation.aspx?qs=1 | | N | N | Y (Login)
General Content | /shopping/Content.aspx | | Y | N | N
Mobile Web ERROR PAGE | /Shopping/Error.aspx | | Y | N | N


NOTE:
* Ignore any parameters when app is trying to recognise the URLs above
* For Live URL links: Replace  **supervalu.buy4nowbeta.com** with **shop.supervalu.ie**

## Javascript Calls by Mobile Web Screen
Javascript calls made by app on each screen with a mobile embedded page, where necessary depending on the URL being called:

Embedded Mobile Web Screen               | UAT URL
:----------------------------------------|:-------
Checkout Login                           | setSessionKey (app will set session key) <br><br> displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘false’)
Checkout 1                               | displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘false’)
Checkout 2                               | displayBackButton (will return ‘true’) <br> completedCheckout (will return ‘false’)
Checkout Confirmation                    | displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘true’)
Active Order Change Payment Login        | setSessionKey (app will set session key) <br> setOrderRef (app will pass order reference number) <br> displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘false’)
Active Order Change Payment              | displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘false’)
Active Order Change Payment Confirmation | displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘true’)
Registration 1 (Loyalty or Not)          | displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘false’)
Registration 2a (Loyalty Entry)          | displayBackButton (will return ‘true’) <br> completedCheckout (will return ‘false’)
Registration 2b (Full Entry)             | displayBackButton (will return ‘true’) <br> completedCheckout (will return ‘false’)
Registration 3 (Confirmation)            | displayBackButton (will return ‘false’) <br> completedCheckout (will return ‘true’)
General Content                          | displayBackButton (will return ‘true’) <br> completedCheckout (will return ‘false’)
Mobile Web ERROR PAGE                    | displayBackButton (will return ‘true’) <br> completedCheckout (will return ‘false’)

On the confirmation pages (checkout and change payment and registration), the buttons displayed on the app behave as follows (where completedCheckout javascript returns true):

* Clicking the Continue button (right of the header) brings customer
    * Back to the homepage on the app for "checkout"
    * Back to the active order homepage on the app for "change payment"
    * Back to the login on the app for "registration"
    * Back button javascript is ignored but should always return false anyway
    * Clicking the Logout button (left of the header) brings customer
        * Calls the logout api for current app session ID
        * Back to the login on the app  

# API – Version Control
Different URL for each new release of the API, the reason for this is to allow old versions of the Mobile Application to keep making their calls on old versions of the API.

New Versions of the App, if they require changes to the API, will call URLs like…

`http://shop.supervalu.ie/shopping/API/[Version]/[EndPoint].asmx/[Method]`

e.g.

`http://shop.supervalu.ie/shopping/API/2/Home.asmx/GetInfo`

# API – UAT to Production
Embedded Mobile Webpage URL Updates

* Replace supervalu.buy4nowbeta.com/ with shop.supervalu.ie/
* All existing prefixes (http/https) and post fixes for the rest of the URLs are the same in production

# API URL Updates

* Replace supervalu.buy4nowbeta.com/API/ with shop.supervalu.ie/API/[VersionNumber]/
    * where version number for this first production build = 1
* All existing prefixes (http/https) and post fixes for the rest of the URLs are the same in production

# API – Auditing
This is an internal requirement of the API deliverables for Buy4Now to ensure all calls to the API are recorded in the database for support purposes.

Implemented, calls are logged; severity levels dictate if a call will be logged. Each call is given a level and the application has a setting in the config file which levels to log, if the API call sev’ level is less than that in the config file, it is not logged.

 
# API – Out of Scope / Retired API Operations
## GetServerDateTime

```
http://supervalu.buy4nowbeta.com/shopping/APIServices/rest.aspx?command=GetServerDateTime
```

> JSON Post Data:

```
{
  "appKey": "XXXXXXXXXXXXXXXXXXXX"
}
```

> JSON Response Data:

```
{
  "responseCode": 0,
  "responseInfo": "SUCCESS",
  "date": "20111129 19:21"
}
```
<aside class="warning">
**NO LONGER IN USE**
</aside>
### Purpose
Gets the time currently on the API server.


## GetMessage

```
http://supervalu.buy4nowbeta.com/shopping/APIServices/rest.aspx?command=GetMessage
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "locationID": 20,
  "sectionID": 3
}
```

> JSON Response Data:

```
{
  "responseCode": 0,
  "responseInfo": "SUCCESS"
  "message": "No alcohol can be sold on good Friday"
}
```

<aside class="warning">
**NOT CURRENTLY IN SCOPE**
</aside>
### Purpose
Get a section of content text.

## GetCategoryChildren

```
http://supervalu.buy4nowbeta.com/shopping/APIServices/rest.aspx?command=GetCategoryTree
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "storeID": 322,
  "categoryID": 1001
}
```

> JSON Response Data:

```
{
  "categories":
  [
    {
      "categoryID": 10011,
      "categoryName": "Green",
      "parentID": 1001
    },
    {
      "categoryID": 10012,
      "categoryName": "Red",
      "parentID": 1001
    },
    {… another category ETC…}
  ]
}
```

<aside class="warning">
**NOT CURRENTLY IN SCOPE**
</aside>
### Purpose
Returns all the child categories from the specified category, only goes down one level (direct descendants of the provided categoryID).

## GetStores

```
http://supervalu.buy4nowbeta.com/shopping/APIServices/rest.aspx?command=GetStores
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "ver": 9,
  "areaID" : 2
}
```

> JSON Response Data:

```
{
  "responseCode": 0,
  "responseInfo": "SUCCESS",
  "ver": 10 ,
  "stores":
  [
    {
      "storeID": 322,
      "storeName": "Ballingcollig",
      "type": "pickup",
      "pickupArea": 1
    },
    {
      "storeID": 344,
      "storeName": "Goatstown",
      "type": "pickup" ,
      "pickupArea": 1
    },
    {
      "storeID": 322,
      "storeName": "Ballingcollig",
      "type": "delivery" ,
      "pickupArea": 0
    },
    {
      "storeID": 344,
      "storeName": "Goatstown",
      "type": "delivery" ,
      "pickupArea": 0
    }
  ]
}
```
<aside class="warning">
**NO LONGER IN USE**
</aside>
### Purpose

<aside class="notice">
In the JSON post data for the GetStores call:<br>
`"areaID" : 2 ` - This is optional, leave out to get entire list
</aside>

## SetPickupLocation

```
http://shop.supervalu.ie/API/[VersionNumber]/logistics.asmx/SetPickupLocation
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "storeID": 322,
  "zoneID": 2
}
```

> JSON Response Data:

```
{
  "d":
  {

    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS"
  }
}
```

<aside class="warning">
**NO LONGER IN USE**
</aside>
### Purpose
Sets the Pickup Store of the customer, also sets the slot mode to Pickup.

Clear down customers slot if storeID/zoneID is not the same. Basket will be re-assigned to new store and items within the basket that don’t exist in the new store will be removed.

This API is no longer required, as its function has being merged into the setSlot api call

## SetDeliveryAddress

```
 http://shop.supervalu.ie/API/[VersionNumber]/logistics.asmx/SetDeliveryAddress
```

> JSON Post Data:

```
{
  "sessionKey": "XXXXXXXXXXXXXXXXXXXX",
  "addressID": 324523
}
```

> JSON Response Data:

```
{
  "d":
  {
    "ResponseCode": 0,
    "ResponseInfo": "SUCCESS",
    "StoreID": 322
  }
}
```

<aside class="warning">
**NO LONGER IN USE**
</aside>
### Purpose

Sets the Delivery address of the customer, also sets the slot mode to delivery.

Clear down customers slot if storeID/zoneID is not the same. Basket will be re-assigned to new store and items within the basket that don’t exist in the new store will be removed.



This API is no longer required, as its function has being merged into the setSlot api call
