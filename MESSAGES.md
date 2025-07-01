## Table of Contents

Singular Types

1. [Type m - Message Send](#type-m---message-send)
2. [Type med - Message Edit](#type-med---message-edit)
3. [Type mr - Message Delivery Response](#type-mr---message-delivery-response)
4. [Type mem - Message Emoji Add or Remove](#type-mem---message-emoji-add-or-remove)

Batch Variants

1. [Type mb - Message Send](#type-mb---message-send-batch)
2. [Type medb - Message Edit](#type-medb---message-edit-batch)
3. [Type memb - Message Emoji Batch](#type-memb---message-emoji-batch)

## Type m - Message Send

> [!WARNING]
> Type `m` is being updated to align with the improved object used by type `p`.
> The `_id` field is being depracated, the existing `ts` field will be used instead for message identification.

### Client to Server

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`m`|String|Always type ‘m’
|Message Id|`_id`|`9cb62327-a67d-4a5c-abbe-eb2fd8471fb3`|String|Guid specfic to the message - common between Client and Server
|From Call|`fc`|`G5ALF`|String|Sending Callsign
|To Call|`tc`|`M0AHN`|String|Receiving Callsign
|Message|`m`|`This is a test`|String|The actual message
|Timestamp|`ts`|`1740312733`|Number|Timestamp of message - seconds since epoch
|**Optional Fields**|
|Reply Id|`r`|`4ef02c95-c448-4dd8-b35f-e7d70f9ea7a0`|String|The id of the message being replied to
|**Server Only Fields**|
|Message Status|`ms`|`1`|Number|0 = Client Sent<BR>1 = Server Received<BR>2 = Recipient Delivered<BR>3 = Recipient Read.<BR>Currently unused - future use case. Server value always currently 1|
|Logged Timestamp|`lts`|`1740312745`|Number|The timestamp the server received and processed the message

> [!NOTE]
> WPS will store and forward any optional fields sent by the client. 

### JSON Example

```json
{
   "t": "m",
   "_id": "9cb62327-a67d-4a5c-abbe-eb2fd8471fb3",
   "fc": "G5ALF",
   "tc": "M0AHN",
   "m": "This is a test",
   "ms": 1,
   "ts": 1740312733
}
```

## Type med - Message Edit

### Client to Server

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`med`|String|Always type ‘mE’
|Id|`_id`|`d25e2702-2023-4906-93f0-5c60a4c18b4d`|String|Guid of the edited message - common between Client and Server
|Edited Message|`m`|`This is a test`|String|The edited message in full
|Edited Flag|`ed`|`1`|Number|Currently used to determine if a message has been edited
|Edited Timestamp|`edts`|`1740312733`|Number|Edited timestamp of message - seconds since epoch

### JSON Example

```json
{
    "t": "med",
    "_id": "d25e2702-2023-4906-93f0-5c60a4c18b4d", 
    "m": "Blah 2", 
    "edts": 1750713928
}
```

### Server to Client

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

## Type mr - Message Delivery Response

### Server to Client

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`mr`|String|Always type ‘sR’ for Send Response
|Id|`_id`|`9cb62327-a67d-4a5c-abbe-eb2fd8471fb3`|String|Guid - common between Client and Server

### JSON Example

```json
{
   "t": "mr", 
   "_id": "9cb62327-a67d-4a5c-abbe-eb2fd8471fb3"
}
```

## Type mem - Message Emoji Add or Remove

WPS doesn't send delivery confirmation responses for emoji additions or removals - they are not deemed essential to the integrity of WPS. 

If the emoji reaches the server, it should always be delivered to the connected client in real-time, or, get picked up at next connect.

There are some edge cases where a client could send a message and the packet network fails before delivery to the server. In this edge case, the sender may see the emoji on their client, but it hasn't been delivered.

Ater every emoji add or remove, both for real-time connections and during the connect sequence, WPS will always return the latest full emoji state for a message. For example, if a message has 1 emoji and 2nd is added, WPS will return both 1st and 2nd in the update.

### Client to Server

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`mem`|String|Message Emoji Add or Remove
|Action|`a`|`1`|Number|`1` for Add, `0` for Remove
|Id|`_id`|`d25e2702-2023-4906-93f0-5c60a4c18b4d`|String|id of the message to apply the emoji
|Timestamp|`ets`|`1750713928`|Number|Timestamp the emoji is created on the client
|Emoji|`e`|`1f44d`|String|The unicode value of the emoji to add or remove

### JSON Example

Emoji Add
```json
{
   "t": "mem",
   "a": 1,
   "_id": "241a9c25-662b-4de4-a2b6-0a5482b65241",
   "e": "1f44d",
   "ets": 1750713928
}
```

Emoji Remove
```json
{
   "t": "mem",
   "a": 0,
   "_id": "241a9c25-662b-4de4-a2b6-0a5482b65241",
   "e": "1f44d",
   "ets": 1750713928
}
```

### Server to Client (Connected Clents Only)

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`mem`|String|Type Emoji
|Id|`_id`|`d25e2702-2023-4906-93f0-5c60a4c18b4d`|String|id of the message to apply the emoji
|Timestamp|`ets`|`1750713928`|Number|Timestamp the emoji is applied at the server, replicated to the client
|Array of Emojis|`e`|`[ "1f44d", "1f603" ]`|

### JSON Example

```json
{ 
   "t": "mem", 
   "_id": "241a9c25-662b-4de4-a2b6-0a5482b65241", 
   "e": [
      "1f44d",
      "1f603"
   ],
   "ets": 1750713928
}
```
