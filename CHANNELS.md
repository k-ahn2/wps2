## Table of Contents
1. [Type cp - Channel Post](#type-cp---channel-post)
2. [Type cpE - Channel Post Edit](#type-cpE---channel-post-edit)
3. [Type cpR - Channel Post Response](#type-cpR---channel-post-response)
4. [Type cpeA and cpeR - Channel Post Emoji Add and Emoji Remove](#type-cpea-and-cper---channel-post-emoji-add-and-emoji-remove)
5. [Type cpB - Channel Post Batch]((#type-cpb----channel-post-batch))

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

A simple post
```json
{
   "t": "cp",
   "cid": 6,
   "fc": "M0AHN",
   "ts": 1750804825979,
   "p": "Testing 123"
}
```

A reply to a post
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

## Type cpeA and cpeR - Channel Post Emoji Add and Emoji Remove

## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cpeA` or `cpeR`|String|`cpeA` for Channel Post Emoji Add or `cpeR` for Channel Post Emoji Remove
|Id|`ts`|`1750361450494`|Number|The ts of post to add or remove the emoji
|Type|`cid`|`6`|Number|id of the channel|
|Emoji|`e`|`1f44d`|String|The unicode value of the emoji to add or remove

### JSON Example

Emoji Add
```json
{
   "t": "cpeA",
   "ts": 1750361450494,
   "cid": 6,
   "e": "1f44d"
}
```

Emoji Remove
```json
{
   "t": "cpeR",
   "ts": 1750361450494,
   "cid": 6,
   "e": "1f44d"
}
```

## Server to Client

If the recipient of the Emoji is connected in real-time, WPS relays the same `cpeA` or `cpeR` object

## Type cS - Channel Subscribe

## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cS` or `cS`|String|`cS` for Channel Subscribe
|Subscribe|`s`|`1`|Number|`1` to subscribe, `0` to unsubscribe
|Type|`cid`|`6`|Number|id of the channel|
|Last Channel Post|`lcp`|`1750361450494`|Number|Usually 0 because the user hasn't previously subscribed, but will send the `ts` of the last post for this channel if one exists on the client 

### JSON Example

Channel Subscribe
```json
{
   "t": "cS",
   "s": 1,
   "cid": 1,
   "lcp": 0
}
```

Channel Unsubscribe
```json
{
   "t": "cS",
   "s": 0,
   "cid": 1,
   "lcp": 0
}
```

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cS` or `cS`|String|`cS` for Channel Subscribe
|Type|`cid`|`6`|Number|id of the channel|
|Subscribe|`s`|`1`|Number|`1` to confirm subscribed, `0` to confirm unsubscribed
|Post Count|`pC`|`25`|Number|Only applicable for Subscribe, this is the number of new posts in the channel. Used by the client to prompt the user how many to download

### JSON Example

``` json
{
   "t": "cSR", 
   "cid": 1, 
   "s": 1, 
   "pC": 0
}
```

``` json
{
   "t": "cSR", 
   "cid": 1, 
   "s": 0, 
}
```

## Type cpB - Channel Post Batch

## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cpB`|String|`cpB` for Channel Post Batch
|Type|`cid`|`6`|Number|id of the channel|
|Post Count|`pc`|`17`|Number|The number of posts to return. Would return the last 17 posts in the channel, sent to the client in ascending (oldest first) order 

### JSON Example

Channel Subscribe
```json
{
   "t": "cPB",
   "cid": 6,
   "pc":17
}
```

## Server to Client
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`cPB`|String|`cPB` for Channel Post Batch
|Type|`cid`|`6`|Number|id of the channel|
|Meta|`m`|`1`|Object| pT = Post Total, in the overall batch <BR>pC = Post Count, the cumulative total after this batch is processed<br>```{ "pT": 17, "pC":4 }```
|Posts|`p`|`[]`|Array|Array of `cp` objects to return to the client. Would include any applicable post fields if added - e.g. emojis, edit and reply

### JSON Example

``` json
{
   "t": "cPB",
   "cid": 6,
   "m": {
      "pT": 17,
      "pC":4
   },
   "p": [
      {
         "t": "cp",
         "cid": 6,
         "fc": "G5ALF",
         "ts": 1750359728258, 
         "p": "TesT"
      },
      {
         "t": "cp",
         "cid": 6,
         "fc": "M0AHN",
         "ts": 1750359773884,
         "p": "Test"
      },
      {
         "t": "cp",
         "cid": 6,
         "fc": "M0AHN",
         "ts": 1750359775310,
         "p": "Test"
      },
      {
         "t": "cp",
         "cid": 6,
         "fc": "M0AHN",
         "ts": 1750359846362, "p": "Test"
      }
   ]
}
```
