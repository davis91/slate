---
title: OVI API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - java

includes:
  - errors

search: true
---

# Introduction

This document describes in detail the API between OVI and OVM. This document is intended for GTK developers only. In case this API document needs to be updated, please commit changes to the ovi-ovm-api repository in bitbucket.

# Authentication

In order to access the OVI API, the IP address of the calling system has to be cleared through OVI's firewall.

There are two places where the IP needs to be added in order to allow it through.

`External IP allowed to use PMSi service. Use comma ( , ) to seperate the IPs.`
And
`servletConfig.allowedIp`

<aside class="notice">
These configurations are under: <br>
- Synapse > PMSi > Global Configurations
<br>
- OVI Console > Applications > Servlets
</aside>

# Configuration set
Timeouts

# OVMRequest

Simple servlet in OVI which is called by OVM to perform various actions. <br>
`End point reachable at https://ovi_ip/api/pmsi/OVMRequest`
<aside class="notice">
Accepts XML. Responds in XML
</aside>

## post

```shell
curl -X POST -d \
"<?xml version='1.0' encoding='UTF-8'?> \
<post> \
<room>101</room> \
<amount>1295</amount> \
<description>MOVIE</description> \
<detailDescription>MOVIE-12911</detailDescription> \
</post>" \
http://darekovi.rndguest.tk/api/pmsi/OVMRequest
```

```java
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command should return XML:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<postReply>
    <description></description>
    <result>true</result>
    <status>no_error</status>
</postReply>
```

This call will post charges to the room.

### Expected XML Parameters

Parameter | Example |Description
--------- | ------- |-----------
room | 110 |room number where the charge should go to
amount| 1095 |Amount in cents how much the charge should be
description | MOVIE | PMS code to what the charge is for
detailDescription | MOVIE-1234 | additional information

### Return XML Paramters
parameter name | values | description
-------------- | ------ | -----------
description | String | hardcoded as ""
result | true/false | Did this request go through successfully?
status | String | short description of the result

### Possible STATUS Response

result | status |reason
------ | ------ |------
true | no_error | posting went through successfully
false| unknown_room | This room does not exist in OVI
false| vacant_room | Room vacant
false| express_checkout_not_permitted | Room is inhibited
false| unsupported_message | Incorrect internal request format
false| unsupported_feature | PMSi not configured
false| invalid_account | Invalid account number
false| PMS_error | No response from the PMS
false| unknown_error | PMS response was unknown to OVI


## roomStatusRequest

Method to request the current status of the room in OVI.

```shell
curl -X POST -d "<?xml version='1.0' encoding='UTF-8'?> \
<roomStatusRequest> \
<room>101</room> \
</roomStatusRequest>" http://darekovi.rndguest.tk/api/pmsi/OVMRequest
```

```java
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> Example of the XML response for the above command:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<roomStatus>
    <checkin>true</checkin>
    <guest>
        <accountNumber>101</accountNumber>
        <checkinDate>2017-10-16</checkinDate>
        <checkoutDate>2017-10-16</checkoutDate>
        <country></country>
        <firstName></firstName>
        <fullName>Plum</fullName>
        <groupNumber></groupNumber>
        <guestInhibit>false</guestInhibit>
        <lastName>Plum</lastName>
        <messageWaiting>none</messageWaiting>
        <oldRoom></oldRoom>
        <paymentMethod>unknown</paymentMethod>
        <primary>false</primary>
        <rateCodeMap/>
        <sharedRoom>false</sharedRoom>
        <title></title>
        <vip>false</vip>
    </guest>
    <inhibit>false</inhibit>
    <resync>false</resync>
    <room>101</room>
</roomStatus>

```

### Expected XML parameters

Parameter | Example |Description
--------- | ------- |-----------
room | 110 |Requested room

### Return Response
These responses correspond to the result XML message.
Possible responses from this endpoint:

parameter name | values | description
-------------- | ------ | -----------
checkin | true/false | Guest is checked in or not
guest | GuestInfo | GuestInfo object. See below for details
inhibit | true/false | Are charges allowed to this room
resync | true/false | Is OVI in resyc state?
room | String | Associated room name

### GuestInfo object parameters
parameter name | values | description
-------------- | ------ | -----------
accountNumber | String | Guest PMS account number
checkinDate| Date | Guest's checkin date (YYYY-MM-DD)
checkoutDate | Date | Guest's checkout date(YYYY-MM-DD)
country | String | Country of origin
firstName | String | Guest's first name
fullName | String | Guest's unparsed full name
groupNumber | String |?
guestInhibit | boolean | Are charges allowed for this guest
lastName | String | Guest's last name
messageWaiting | none/undelivered/alldelivered | Any messages for the guest
oldRoom | String | Guest's old room
paymentMethod | cash/creditCard/unkown | Guest's payement type
primary | boolean | Is this guest the primary account holder
rateCodeMap | Array<String> | ratecodes associated with this guest
sharedRoom | boolean | Is this room shared with another guest
title | String | Guest's title
vip | boolean | Is this guest considered a vip


## folioRequest

```shell
curl -X POST -d "<?xml version='1.0' encoding='UTF-8'?> \
<folio> \
<room>102</room> \
<account>102</account> \
</folio>" http://darekovi.rndguest.tk/api/pmsi/OVMRequest
```
> The example return XML:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<folioReply>
    <balance>0</balance>
    <item>
        <amount>1000</amount>
        <date>2017-10-16-06:00</date>
        <description>STANDARD INTERNET</description>
    </item>
    <item>
        <amount>995</amount>
        <date>2017-10-16-06:30</date>
        <description>MOVIE-1234</description>
    </item>
    <result>true</result>
    <status>no_error</status>
</folioReply>
```

### Expected XML Parameters

Parameter | Example |Description
--------- | ------- |-----------
room | 110 |room number for the folio request
account| 110 |account number for the folio request

### Return XML Paramters
parameter name | values | description
-------------- | ------ | -----------
balance | integer | total folio balance. Default -1 if no balance available
result | true/false | Did this request go through successfully?
item | Array<FolioItem> | List of FolioItem objects. See below for details
status | String | possible status responses. See below for details
description | String | short description for this response

### FolioItem object parameters
parameter name | values | description
-------------- | ------ | -----------
amount | integer | Charge amount
description | String | Folio description of the item
date | Date | Charge date (YYYY-MM-DD-HH:MM)
location | String | Optional.

### Possible STATUS Response

result | status |reason
------ | ------ |------
true | no_error | posting went through successfully
false| unknown_room | This room does not exist in OVI
false| vacant_room | Room vacant
false| express_checkout_not_permitted | Room is inhibited
false| unsupported_message | Incorrect internal request format
false| unsupported_feature | PMSi not configured
false| invalid_account | Invalid account number
false| PMS_error | No response from the PMS
false| unknown_error | PMS response was unknown to OVI

## expressCheckout

```shell
url -X POST -d "<?xml version='1.0' encoding='UTF-8'?> \
<expressCheckout> \
<room>100</room> \
<account>100</account> \
<balance>1000</balance> \
</expressCheckout>" http://darekovi.rndguest.tk/api/pmsi/OVMRequest
```
> The above command returns example XML:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<expressCheckoutReply>
    <description></description>
    <result>true</result>
    <status>no_error</status>
</expressCheckoutReply>
```

### Expected XML Parameters

Parameter | Example |Description
--------- | ------- |-----------
room | 110 |room number for the folio request
account| 110 |account number for the folio request
balance | 1295 |Total balance of the folio. Has to match the current folio balance for this room.

### Return XML Paramters
parameter name | values | description
-------------- | ------ | -----------
description | String | Description from PMS
result | boolean | Did this request go through successfully?
status | String | short description of the result

### Possible STATUS Response

result | status |reason
------ | ------ |------
true | no_error | posting went through successfully
false| unknown_room | This room does not exist in OVI
false| vacant_room | Room vacant
false| express_checkout_not_permitted | Room is inhibited
false| unsupported_message | Incorrect internal request format
false| unsupported_feature | PMSi not configured
false| invalid_account | Invalid account number
false| PMS_error | No response from the PMS or balance mismatch
false| unknown_error | PMS response was unknown to OVI

# OVMAPI

Endpoint in OVI which is called by OVM to get or delete messages which are stored in the PMS from the front desk.
These messages are displayed on TVs for hotel guests.

`End point reachable at https://ovi_ip/api/pmsi/OVMAPI`
<aside class="notice">
Accepts JSON. Responds in JSON
</aside>

<aside class="success">
Only supported by `FIAS` interface
</aside>

## retrieveMessagesRequest

A way of retrieving messages which are stored in the PMS for the guest. These can be entered by the front desk.

### Expected JSON Parameters

Parameter | Example |Description
--------- | ------- |-----------
room | 110 |room number for the folio request
accountId| 110 |account number for the folio request

### Return XML Paramters

TBD

```shell
curl -X POST -d "{ \
\"roomNumber\": \"101\",\
\"accountId\": \"101\"\
}" http://darekovi.rndguest.tk/api/pmsi/OVMAPI
```

> Should return this json

```json
{
    "messages" : [""]
}
```
## deleteMessageRequest

A way of deleting messages which are stored in the PMS

### Expected JSON Parameters

Parameter | Example |Description
--------- | ------- |-----------
roomNumber | 110 |room number for the folio request
accountId| 110 |account number for the folio request
messageId| 001 |id of the message that needs to be removed in the PMS

### Expected result

parameter name | values | description
-------------- | ------ | -----------
result | boolean | result of the removal


```shell
curl -X POST -d "{ \
\"roomNumber\": \"101\",\
\"accountId\": \"101\",\
\"messageId\": \"001\"\
}" http://darekovi.rndguest.tk/api/pmsi/OVMAPI
```

> Should return this json

```json
{
    "result":true
}
```

# OutGoingMessageController

This endpoint is used for sending messages to OVM.



