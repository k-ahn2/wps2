## Table of Contents
1. [Type cp - Channel Post](#type-cp---channel-post)
2. [Type cpE - Channel Post Edit](#type-cpE---channel-post-edit)
3. [Type cpR - Channel Post Response](#type-cpR---channel-post-response)

## Type cp - Channel Post
### Client to Server

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cp`|String|Always type `cp` for Channel Post|
|Type|`cid`|`1`|Number|id of the channel|
|From Call|`fc`|`M0AHN`|String|Callsign of the sender|
|Timestamp|`ts`|`1750804825979`|Number|Milliseconds since epoch|
|Post|`p`|`Testing 123`|String|The posted message|
|**Optional Fields**|
|Reply Timestamp|`rta`|`1750804825979`|Number|The timestamp of the post being replied to
|Reply From Call|`rfc`|`M0AHN`|String|The send of the post being replied to
|**Server Only Fields**|
|Delivery Timestamp|`dts`|`1750804826875`|Number|The timestamp the server received and processed the message. This is returned to the client in the `cpR` response for the client to calculate the delivery time to server

### JSON Example

```json
{
   "t": "cp",
   "cid": 6,
   "fc": "M0AHN",
   "ts": 1750804825979,
   "p": "Testing 123"
}
```

```json
{
   "t": "cp",
   "cid": 6,
   "fc": "M0AHN",
   "ts": 1750805783394,
   "p": "Blah",
   "rts": 1750804825979,
   "rfc": "M0AHN"
}
```

## Type cpE - Channel Post Edit
## Client to Server

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cpE`|String|Always type ‘cpE’ for Channel Post Ecit
|Type|`cid`|`6`|Number|id of the channel|
|Timestamp|`ts`|`1750804825979`|Number|Timestamp of original post|
|Post|`p`|`Testing 123`|String|The updated post|
|Edited Timestamp|`ts`|`1750804825979`|Number|Timestamp of original post|

### JSON Example

```json
{
   "t": "cpE",
   "cid": 6,
   "ts": 1750804825979,
   "p": "Test1",
   "edts": 1750805550246
}
```

## Server to Client

## Type cpR - Channel Post Response

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cpR`|String|Always type ‘cpR’ for Channel Post Response
|Timestamp|`ts`|`1750804825979`|Number|Timestamp of the post
|Delivery Timestamp|`dts`|`1750804827975`|Number|The timestamp the server received and processed the message. Used by the client to calculate the delivery time to server

### JSON Example

```json
{
   "t": "cpR",
   "ts": 1750804825979,
   "dts": 1750804827975
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
