## Table of Contents
1. [Type c - Connect](#type-c---connect)
2. [Type m and sR - Message Send & Message Send Response](#type-m-and-sr---message-send-&-message-send-response)

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

## Type m and sR - Message Send & Message Send Response

> [!TODO]
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
|Reply Id|`r`|`4ef02c95-c448-4dd8-b35f-e7d70f9ea7a0`|Number|0 = Client Sent, 1 = Server Received
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
   "_id": "b9de5114-d6b7-477b-8d22-042844ff2db9"
}
```

TODO
Remove ms key
Does not need to be sent over RF. Client sets as 0 until it gets the Send Reply. Server should set to 1 upon receipt
Remove _id key
Does not need to be sent over RF. Client and Server can use different database ids (same as built for Posts). Currently the delivery receipt is applied using the _id - can use the timestamp instead (again same as a Post)
Remove fc
Does not need to be sent over RF. The server knows who sent the message from the session - the server can add the from call upon receipt

## Type M - Message
### Overview
## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |

### JSON Example

```json
```

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |

### JSON Example

```json
```

## Type M - Message
### Overview
## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |

### JSON Example

```json
```

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |

### JSON Example

```json
```
