## Table of Contents
1. [Type c - Connect](#type-c---connect)
2. [Type m and sR - Message Send & Message Send Response](#type-m-and-sr---message-send-&-message-send-response)
2. [Type mE - Message Edit](#type-me-message-edit)
2. [Type m and sR - Message Send & Message Send Response](#type-m-and-sr---message-send-\&-message-send-response)
2. [Type m - Message Send](#type-m---message-send)
3. [Type mE - Message Edit](#type-me-message-edit)
4. [Type sR - Message Delivery Response](#type-sR---message-delivery-response)
5. [Type eA and eR - Emoji Add and Emoji Remove](#type-ea-and-er---emoji-add-and-emoji-remove)
6. [Type p and pR - Enable Pairing and Pairing Response](#type-p-and-pr---enable-pairing-and-pairing-response)
7. [Type uE - User Enquiry](#type-ue---user-enquiry)

## Type c - Connect
This is the first data exchange after connect - the client sends a type ‘c’ object to the server, which uses this data to determine the client state and which messages and/or posts to return.

Upon receipt, the server:
1. Uses the timestamps to establish the new messages and/or posts to be sent to the client
2. Returns a type `c` object to the client, containing the new message and post counts
3. Then uses the timestamps to return:
   - new messages
   - new message emojis
   - new message edits
   - new posts
   - new post emojis
   - new post edits

See **Connect Sequence** for a detailed explanation of the subsequent connect process

## Client to Server

### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`c`|String|Always type c|
|Name|`n`|`Kevin`|String|Name as entered in WhatsPac Client|
|Callsign|`c`|`M0AHN`|String|Callsign as entered in WhatsPac Client, minus the SSID if added|
|Last Message|`lm`|`1740299150`|Number|Timestamp of last message - seconds since epoch|
|Last Emoji|`le`|`1740266497`|Number|Timestamp of last message emoji - seconds since epoch|
|Last Edit|`led`|`1739318078`|Number|Timestamp of last message edit - seconds since epoch|
|Version|`v`|`0.44`|Number|Version of client, used for logging only|
|Channel Connect|`cc`|`[]`|Array of Objects|Contains one JSON object per channel subscribed|
|**Channel Connect Objects**|
|Channel Id|`cid`|`2`|Number|The Channel Id|
|Last Post|`lp`|`1740251305826`|Number|Timestamp of last post - milliseconds since epoch|
|Last Emoji|`le`|`1740252223588`|Number|Timestamp of last post emoji - milliseconds since epoch|
|Last Edit|`led`|`1738869165936`|Number|Timestamp of last post edit - milliseconds since epoch|

### JSON Example

```json
{
   "t": "c",
   "n": "Kevin",
   "c": "M0AHN",
   "lm": 1740299150,
   "le": 1740266497,
   "led": 1739318078,
   "v": 0.44,
   "cc":[
      {
         "cid": 3,
         "lp": 1740242101095,
         "le": 1740247089774,
         "led": 1740250727684
      },
      {
         "cid": 4,
         "lp": 1740251305826,
         "le": 1740252223588,
         "led": 1738869165936
      }
   ]
}
```

## Server to Client

### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`c`|String|Always type ‘c’
|New Message Count|`mc`|`5`|Number|Number of new messages pending download
|Welcome|`w`|`1`|Boolean|Set to 1 if the connect if from a new user, used to display welcome message. Only sent if True
|Version|`v`|`0.44`|Number|Set to the latest available client version. The client will prompt to upgrade if this value is higher than the currently used client version
|New Post Count|`pc`|`[]`|Array of Objects|Number of new posts pending download
|**New Post Count Objects**|
|Channel Id|`cid`|`2`|Number|The Channel Id
|Unread Count|`uc`|`4`|Number|The number of unread posts in the channel

### JSON Example

```json
{
   "t": "c",
   "w": 1,
   "mc": 25,
   "v": 0.44,
   "pc": [
      {
         "cid": 3,
         "uc": 1059
      },
      {
         "cid": 4,
         "uc": 167
      }
   ]
}
```

## Type m - Message Send

> [!WARNING]
> Type `m` is being updated to align with the improved object used by type `p`.
> The `_id` field is being depracated, the `ts` will be used instead.
> Type `sR` is being renamed to type `mR` for consistency with the post format

## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`m`|String|Always type ‘m’
|Id|`_id`|`9cb62327-a67d-4a5c-abbe-eb2fd8471fb3`|String|Guid - common between Client and Server
|From Call|`fc`|`G5ALF`|String|Sending Callsign
|To Call|`tc`|`M0AHN`|String|Receiving Callsign
|Message|`m`|`This is a test`|String|The actual message
|Timestamp|`ts`|`1740312733`|Number|Timestamp of message - seconds since epoch
|**Optional Fields**|
|Reply Id|`r`|`4ef02c95-c448-4dd8-b35f-e7d70f9ea7a0`|String|The id of the message being replied to
|**Server Only Fields**|
|Message Status|`ms`|`1`|Number|0 = Client Sent<BR>1 = Server Received<BR>2 = Recipient Delivered<BR>3 = Recipient Read.<BR>Currently unused - future use case. server value always currently 1|
|Logged Timestamp|`lts`|`1740312745`|Number|The timestamp the server received and processed the message

> [!NOTE]
> WPS will store and forward any optional fields sent by the client. 

### JSON Example

```json
{
   "t":"m",
   "_id":"9cb62327-a67d-4a5c-abbe-eb2fd8471fb3",
   "fc":"G5ALF",
   "tc":"M0AHN",
   "m":"This is a test",
   "ms":1,
   "ts":1740312733
}
```

## Type mE - Message Edit
## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`mE`|String|Always type ‘mE’
|Id|`_id`|`d25e2702-2023-4906-93f0-5c60a4c18b4d`|String|Guid of the edited message - common between Client and Server
|Edited Message|`m`|`This is a test`|String|The edited message in full
|Edited Flag|`ed`|`1`|Number|Currently used to determine if a message has been edited
|Edited Timestamp|`edts`|`1740312733`|Number|Edited timestamp of message - seconds since epoch

### JSON Example

```json
{
    "t": "mE",
    "_id": "d25e2702-2023-4906-93f0-5c60a4c18b4d", 
    "m": "Blah 2", 
    "edts": 1750713928
}
```

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`sR`|String|Always type ‘sR’ for Send Response
|Id|`_id`|`d25e2702-2023-4906-93f0-5c60a4c18b4d`|String|id of the message to apply the emoji

### JSON Example

```json
{
   "t": "sR", 
   "_id": "d25e2702-2023-4906-93f0-5c60a4c18b4d"
}
```
## Type mR - Message Delivery Response

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`sR`|String|Always type ‘sR’ for Send Response
|Id|`_id`|`9cb62327-a67d-4a5c-abbe-eb2fd8471fb3`|String|Guid - common between Client and Server

### JSON Example

```json
{
   "t": "sR", 
   "_id": "9cb62327-a67d-4a5c-abbe-eb2fd8471fb3"
}
```

**TODO**
- Remove _id key - Does not need to be sent over RF. Client and Server can use different database ids (same as built for Posts). Currently the delivery receipt is applied using the _id - can use the timestamp instead (again same as a Post)
- Remove fc - Does not need to be sent over RF. The server knows who sent the message  from the session - the server can add the from call upon receipt

## Type eA and eR - Emoji Add and Emoji Remove

## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`eA` or `eR`|String|`eA` for Emoji Add or `eR` for Emoji Remove
|Id|`_id`|`d25e2702-2023-4906-93f0-5c60a4c18b4d`|String|id of the message to apply the emoji
|Emoji|`e`|`1f44d`|The unicode value of the emoji to add or remove

### JSON Example

Emoji Add
```json
{
   "t": "eA",
   "_id": "241a9c25-662b-4de4-a2b6-0a5482b65241",
   "e": "1f44d"
}
```

Emoji Remove
```json
{
   "t": "eR",
   "_id": "241a9c25-662b-4de4-a2b6-0a5482b65241",
   "e": "1f44d"
}
```

## Server to Client

If the recipient of the Emoji is connected in real-time, WPS relays the same `eA` or `eR` object

## Type p and pR - Enable Pairing and Pairing Response

## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`p`|String|Always type `p` for Pairing
|Callsign|`fc`|`M0AHN`|String|The callsign of the user entering pairing mode

### JSON Example

```json
{
   "t": "p",
   "fc": "M0AHN"
}
```

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`pR`|String|Always type `pR` for Pairing Response
|Enabled|`e`|`true`|Boolean|Tells the client pairing is enabled
|Start Time|`st`|`1750799200`|Number|The server start time for pairing, seconds since epoch

### JSON Example

```json
{
   "t": "pR",
    "e": true,
    "st": 1750799200
}
```

## Type uE - User Enquiry
Used to determine if a user is registered with WPS
## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`uE`|String|Always type ‘uE’ for User Enquiry
|Callsign|`fc`|`M0AHN`|String|The callsign of the user you're enquiring about

### JSON Example

```json
{
   "t" : "uE",
   "c": "G5ALF"
}
```

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`ueR`|String|Always type `ueR` for User Enquiry Response
|Registered|`r`|`true` or `false`|Boolean|True if registered
|Start Time|`tc`|`G5ALF`|String|Callsign of the user enquired about


### JSON Example

```json
{
   "t": "ueR",
   "r": false,
   "tc": "G5ALF"}
```
