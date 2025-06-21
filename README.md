# WPS - A Messaging Service and Protocol for Packet Radio

WPS is a backend service and protocol that provides messaging services over Packet Radio. Currently built to interface with a BPQ or Xrouter node, WPS is directly exposed to the AX:25 packet network and can be systematically accessed by end user applications that implement its protocol. 

WPS was built to enable the WhatsPac front end, but implements a protocol that could be used by any Packet Radio messaging application.

WPS is capable of operating effectively over 1200 baud links or greater and without any internet dependency.

> [!IMPORTANT]
> WPS is in active development and is changing on a regular basis - please remember to watch the repo to be alerted when there are new versions

## Overview and Setup
1. [WPS Schematic](#wps-schematic)
2. Key functions
3. Server Capabilities
4. [How WPS Works - An Overview](#how-wps-works---an-overview)
5. Installation
6. [Sending a JSON object to WPS (Javascript Example)](#sending-a-json-object-to-wps-(javascript-example))
7. Node Integration - Interfacing with BPQ Xrouter
8. Common Processing Considerations
9. Configuring `env.json`
10. Connect Sequence

## Protocol Definition
1. [Type C - Connect](#type-c---connect)

## WPS Schematic
<img src="wps.jpg" alt="blah" width="500px"/>

\* Supported via the Packet Alerts app, integrated with WhatsPac

## Functionality
- **Direct Messaging:** Message send and receive (similar to SMS, WhatsApp, Signal or iMessage)
- **Channels:** Post to themed channels (similar to a Rocket Chat, Slack or Discord)
- **Who is Online:** WPS updates clients when a user connects or disconnects
- **Reply:** Users can sent new messages and posts, or reply to existing
- **Emojis:** Include and react to messages and posts with Emojis
- **Edits:** Edit messages and posts after sending
- **User Registration:** New users are automatically registered upone connecting
- **Push Notifications:** Send push notfications when there is new activity (if paired with the Packet Alerts app, currently via WhatsPac only)
- **Packet Terminal:** Includes a traditional Packet Terminal for browsing the network
- **Callsign Lookup:** Determine if a callsign is registered
- **Name Change:** WPS distrbutes name updates when they change
- **Last Seen Times:** See when users you have messaged were last connected
- **Delivery Receipts:** WPS responds to new and edited posts with a delivery receipt, guaranteeing server delivery
- **Version Control:** Advise the client a new software version is available, configurable within WPS in real-time

## Other Capabilities
- **Compression:** WPS compresses every packet before sending, then sends whichever of the compressed or uncompressed version is shorter
- **Data Batching:** WPS batches bulk post downloads, optimising compression and delivery
- **Logging:** WPS includes error logging by defauly, with extensive info logging configurable if required
- **Run as Service:** WPS runs as a standard linux service (and assume can on Windows too)

## Future
- REST APIs
- Replication

## How WPS Works - An Overview
Must add a /r
WhatsPac strips whitespace
Strips SSID
WPS is a reactive service - activity is only triggered upon receipt of an instruction from a connect client. For example:
1. WPS receives a new message from a connected user (the sender)
2. It writes the message to the database and returns a delivery receipt to the sender. It then decides:
   - if the recipient is connected, send in real-time
   - if the recipient is not connected, check whether registered for push notifications and end processing
   - if the recipient is not registered, end processing
3. The recipient connects and sends its last message timestamps, then the server will return the new message

## Sending a JSON object to WPS (Javascript Example)

```javascript
const sendConnectString = {
   t: "c",
   n: "Kevin",
   c: "M0AHN",
   lm: 123,
   le: 456,
   led: 789,
   lhts: 123,
   v: 0.44
}
ws.send(`${JSON.stringify(sendConnectString)}\r`)
```

## Type C - Connect
### Overview
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
This is the first data exchange after connect - the client sends a Type ‘c’ object to the server, which uses this data to determine the client state and what to return.

### Schematic

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`c`|String|Always type c|
|Name|`n`|`Kevin`|String|Name as entered in WhatsPac Client|
|Callsign|`c`|`M0AHN`|String|Callsign as entered in WhatsPac Client, minus the SSID if added|
|Last Message|`lm`|`1740299150`|Number|Timestamp of last message - seconds since epoch|
|Last Emoji|`le`|`1740266497`|Number|Timestamp of last emoji - seconds since epoch|
|Last Edit|`led`|`1739318078`|Number|Timestamp of last edit - seconds since epoch|
|Version|`v`|`0.44`|Number|Version of client, used for logging only|
|Channel Connect|`cc`|`[]`|Array of Objects|Contains one JSON object per channel subscribed|
|**Channel Connect Objects**|
|Channel Id|`cid`|`2`|Number|The Channel Id|
|Last Post|`lp`|`1740251305826`|Number|Timestamp of last post - milliseconds since epoch|
|Last Emoji|`le`|`1740252223588`|Number|Timestamp of last emoji - milliseconds since epoch|
|Last Edit|`led`|`1738869165936`|Number|Timestamp of last edit - milliseconds since epoch|

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

### Schematic

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
