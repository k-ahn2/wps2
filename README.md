# WPS - A Messaging Service and Protocol for Packet Radio

WPS is a backend service and protocol that provides messaging services over Packet Radio. Currently built to interface with a BPQ or Xrouter node, WPS is directly exposed to the AX:25 packet network and can be systematically accessed by end user applications that implement its protocol. 

WPS was built specifically to enable the functionality in the WhatsPac front end, but implements a protocol that could be used by any Packet Radio messaging application.

WPS is capable of operating effectively over 1200 baud links or greater and without any internet dependency.

WPS runs entirely in Python, starts with just three files, has minimal dependencies, minimal setup and runs with single command. It can be run manually or as a service

> [!IMPORTANT]
> WPS is in active development and is changing on a regular basis - please remember to watch the repo to be alerted when there are new versions

## Overview and Setup
1. [WPS Schematic](#wps-schematic)
2. [Key functions](#key-functions)
3. [Server Capabilities](#server-capabilities)
4. [How WPS Works - An Overview](#how-wps-works---an-overview)
5. [WPS Installation and Prereqs](#wps-installation-and-prereqs)
6. [Sending a JSON object to WPS (Javascript Example)](#sending-a-json-object-to-wps---a-javascript-example)
7. Node Integration - Interfacing with BPQ or Xrouter
8. Common Processing Considerations
9. Configuring `env.json`
10. Connect Sequence
11. Files

## Protocol Definition
1. [Type c - Connect](#type-c---connect)
2. [Type m and sR - Message Send & Message Send Response](#type-m-and-sr---message-send-&-message-send-response)

## WPS Schematic
<img src="wps.jpg" alt="blah" width="500px"/>

\* Supported via the Packet Alerts app, integrated with WhatsPac

## Key Functions
- **Direct Messaging:** Message send and receive (similar to SMS, WhatsApp, Signal or iMessage)
- **Channels:** Post to themed channels (similar to a Rocket Chat, Slack or Discord)
- **Who is Online:** WPS updates clients when a user connects or disconnects
- **Reply:** Users can sent new messages and posts, or reply to existing
- **Emojis:** Include and react to messages and posts with Emojis
- **Edits:** Edit messages and posts after sending
- **User Registration:** New users are automatically registered upone connecting
- **Push Notifications:** Send push notfications when there is new activity (if paired with the Packet Alerts app, currently via WhatsPac only)
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
- **Replication:** Supporting the ability to replicate to other WPS instances hosted on the Packet Network
- **Websocket / REST APIs:** For connecting directly over TCP, it's the intent that WPS will offer Websocket and REST APIs for access. Possible use cases are via Hamnet or local sysop access

## How WPS Works - An Overview

WPS is designed for system access only - it does not provide a human interface for direct user access. To connect to WPS:

1. An application opens an AX:25 connection to the node hosting WPS
2. The application sends `WPS` (or whichever name configured) and the node opens the TCP connection to WPS. WPS expects the first string to be the connecting users callsign 
3. The application then sends JSON compatible with the WPS Protocol and returns the corresponding JSON in response

> [!TIP]
> BPQ and Xrouter support publising an application directly onto the AX:25 network with a callsign and alias. If configured, steps 1 and 2 can be merged. A connecting application can invoke WPS directly upon connecting

WPS is a reactive service - activity is only triggered upon receipt of an instruction from a connect application. It is also connection aware - if it recieves a message from M0AHN to G5ALF and G5ALF is connected, WPS will send the message to G5ALF in real-time

The sequence for a new message is:
1. WPS receives a type `m` JSON object from a connected application, meaning a new message
2. WPS writes the message to the database
3. WPS returns a delivery receipt to the sender. 
4. WPS then decides:
   - if the recipient is connected, send in real-time
   - if the recipient is not connected, check whether registered for push notifications, send if yes
   - if the recipient is not registered, end processing
5. If not sent in real-time, when the recipient connects and sends a type `c` JSON object, WPS will then return the new message(s)

## WPS Installation and Prereqs

> [!NOTE]
> The author has only tested WPS on a Raspberry Pi running Raspbian. There is no known reason it shouldn't run in any Python environment. Please share your feedback so we can update the docs for others

1. Clone the repository using git clone `https://github.com/k-ahn2/wps`
2. Go to the `wps` directory
3. Run `python3 wps.py`

This should start WPS with a default configuration. When running for the first time, WPS will create and initialise the database `wps.db`, plus `wps.log` and `db.log`

Check for errors in the console. Confirmation of the TCP Port is shown - check this matches the port in BPQ or Xrouter.

## How WPS handles JSON

WPS receives everything from the packet network and node as a string, with discreet packets delimited by `0x0D` (13 decimal), Additional delimiters are used for compressed packets. 

WPS preprocesses received strings by:
- adding the string to an RX buffer
- splits the buffer on `0x0D` into an array
- for each string in the array:
   - if enclosed in compression delimiters, attempt decompression
   - if enclosed in `{}`, attempt conversion to a JSON object
   - if last in the array and is neither, return the string to the buffer
- If either of the decompression or JSON conversion fails, this is considered a FATAL error and WPS disconnects the user

In the even of conversion failure, WPS will log and `ERROR` in `wps.log`

The only exceptions to the above are the first and second strings recieved:
- The first string recieved is always the callsign - e.g. `M0AHN\r`. This is sent by the node and happens before any subsequent processing
- If the second string fails conversion, this is likely a manual connect by a human. WPS returns a friendly message (configurable in `wps.py`) and then disconnects

Must add a /r
WhatsPac strips whitespace
Strips SSID
Integrity Checks

## Sending a JSON object to WPS - A Javascript Example

With an open channel to WPS, connected applications should:
1. Convert the JSON object to a string via `JSON.stringify` (Javascript), `json.dumps` (Python) or equivalent
2. Add a `chr(13)` or `\r` or `0x0D` or equivalent, then send.

Javascript Example:

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
send(`${JSON.stringify(sendConnectString)}\r`)
```

## Node Integration - Interfacing with BPQ or Xrouter

> [!NOTE]
> Xrouter node setup to be added

> [!WARNING]
> This section requires basic familiarity with BPQ configuration files and ideally custom application setup. Examples shown but please consult the BPQ documentation for more information

### Simple Application Config
`APPLICATION 1,WPS,C 8 HOST 0 TRANS`

### Config with NETROM Alias
`APPLICATION 1,WPS,C 8 HOST 0 TRANS,MB7NPW-9,WTSPAC,200,WTSPAC`

### Config Entries (abridged)
```
PORT
   PORTNUM=8
   DRIVER=TELNET
   CONFIG
   DisconnectOnClose=1       ; Ensures the client is fully disconnected if the TCP Port disconnects
   CMDPORT 63001 63002       ; Port and position must match. HOST 0 is 63001, HOST 1 is 63002
   MAXSESSIONS=25            ; Maxmimum simultaneous connections
   ....
END PORT
```

## Type c - Connect
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

### Object Fields

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
### Overview
## Client to Server
### Object Fields

| Friendly Name | Key | Sample Values | Data Type | Notes |
| - | :-: | :-: | :-: | - |
|Type|`t`|`m`|String|Always type ‘m’
|Id|`_id`|`9cb62327-a67d-4a5c-abbe-eb2fd8471fb3`|String|Guid - common between Client and Server
|From Call|`fc`|`G5ALF`|String|Sending Callsign
|To Call|`tc`|`M0AHN`|String|Receiving Callsign
|Message|`m`|`This is a test`|String|The actual message
|Message Status|`ms`|`1`|Number|0 = Client Sent, 1 = Server Received
|Timestamp|`ts`|`1740312733`|Number|Timestamp of message - seconds since epoch

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
