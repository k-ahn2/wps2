# WPS - A Messaging Service and Protocol for Packet Radio

WPS is a backend service and Layer 7 application protocol that provides messaging services over Packet Radio. Currently built to be published via a BPQ or Xrouter node, WPS is directly exposed to the AX:25 packet network and can be systematically accessed by end user applications. 

WPS was built specifically to enable the functionality in the WhatsPac front end, but could be used by any Packet Radio messaging application that implements its protocol.

WPS is capable of operating effectively without any internet dependency over link speeds of 1200 baud, albeit the latest 2400 and 3600 baud speeds offered by the NinoTNC are typically used and are the recommended minimum.

WPS runs entirely in Python, starts with just three files, has minimal dependencies, minimal setup and runs with single command. It can be run manually or as a service

> [!IMPORTANT]
> WPS is in active development and is changing on a regular basis - please remember to watch the repo to be alerted when there are new versions

## Table of Contents
1. [WPS Schematic](#wps-schematic)
2. [Key functions](#key-functions)
3. [Server Capabilities](#server-capabilities)
4. [How WPS Works - An Overview](#how-wps-works---an-overview)
5. [Timestamps and Delivery Sequence](#timestamps-and-delivery-sequence)
6. [How WPS handles JSON](#how-wps-handles-json)
7. [WPS Installation and Protocol Documentation](#wps-installation-and-protocol-documentation)

## WPS Schematic
<img src="wps.jpg" alt="blah" width="500px"/>

## Key Functions
- **Direct Messaging:** Message send and receive (similar to SMS, WhatsApp, Signal or iMessage)
- **Channels:** Post to themed channels (similar to a Rocket Chat, Slack or Discord)
- **Who is Online:** WPS updates clients when a user connects or disconnects
- **Reply:** Users can send new messages and posts, or reply to existing
- **Emojis:** Include and react to messages and posts with Emojis
- **Edits:** Edit messages and posts after sending
- **Auto Registration:** New users are automatically registered upone connecting
- **Push Notifications:** Send push notfications when there is new activity (if paired with the Packet Alerts app, currently via WhatsPac only)
- **Callsign Lookup:** Enquire a callsign is registered
- **Name Change:** WPS distrbutes display name updates, if changed
- **Last Seen Times:** See when users you have messaged were last connected
- **Delivery Receipts:** WPS responds to new and edited messages and posts with a delivery receipt, guaranteeing server delivery
- **Version Control:** Advise the client a new software version is available, configurable within WPS in real-time

## Other Capabilities
- **Compression:** WPS compresses every packet before sending, then sends whichever of the compressed or uncompressed version is shorter
- **Data Batching:** WPS batches bulk post downloads, optimising compression and delivery
- **Logging:** WPS includes error logging by default, with extensive info logging configurable if required
- **Run as Service:** WPS runs as a standard linux service (and assume could on Windows too)

## Future
- **Replication:** Supporting the ability to replicate to other WPS instances hosted on the Packet Network
- **Websocket / REST APIs:** For connecting directly over TCP/IP, it's the intent that WPS will offer Websocket and REST APIs for access. Possible use cases are via Hamnet or local sysop access

## How WPS Works - An Overview

WPS is designed for system access only - it does not provide a human interface for direct user access. To connect to WPS:

1. An application opens an AX:25 connection to the node hosting WPS
2. The application sends `WPS` (or whichever name configured) and the node opens the TCP connection to WPS. WPS expects the first string to be the connecting users callsign 
3. The application then sends JSON compatible with the WPS Protocol and returns the corresponding JSON in response

> [!TIP]
> BPQ and Xrouter support publising an application directly onto the AX:25 network with a callsign and alias. If configured, steps 1 and 2 can be merged. A connecting application can invoke WPS directly upon connecting

WPS is a reactive service - activity is only triggered upon receipt of an instruction from a connected user. It is also connection aware - if it recieves a message from M0AHN addressed to G5ALF and G5ALF is connected, WPS will send the message to G5ALF in real-time

As an example, the sequence for a new message is:
1. WPS receives a type `m` JSON object from a connected application, meaning a new message
2. WPS writes the message to the database
3. WPS returns a delivery receipt to the sender. 
4. WPS then decides:
   - if the recipient is connected, send in real-time
   - if the recipient is not connected, check whether registered for push notifications, send if yes
   - if the recipient is not registered, end processing
5. If not sent in real-time, when the recipient connects and sends a type `c` JSON object, WPS will then return the new message(s)

> [!IMPORTANT]
> WPS uses timestamps extensively. A post sent by a user will use the client timestamp on both client, server and destination user's database. If the sending user's clock is materially incorrect, WPS may incorrectly sort messages and you may encounter issues with certain functions. 

## Timestamps and Delivery Sequence

Timestamps are used extensively in the design of WPS:
- Due to the potential variability in the RF packet network - where delivery times to the server can very depending on the number of hops, traffic and/or network drop outs - the timestamp assigned to a message or post when sent by the client is the authority used on both the server and all recipients. This ensures the sequencing of posts and messages remains as intended by the sender
- For Posts, the timestamp is also used to:
  - tell the sender how long a message took to reach the WPS by returning the server delivery timestamp `dts` to the client
  - tell connected recipients of a post the end-to-end delivery time, calculated by comparing the `ts` of the post to the timestamp it was received
- When downloading new posts, either on connect, subscribe or in real time, WPS always sends Posts and Messages in timestamp ASCENDING order - i.e. oldest first. This ensures if a user gets disconnected, the client can resume from the last message

> [!WARNING]
> WPS works on the assumption that modern OSs have time syncronisation and therefore accurate clocks. Beware if offline or time syncronisation is not live that sending a posts with an incorrect system clock could cause strange behaviours

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

In the event of conversion failure, WPS will log an `ERROR` in `wps.log`

The only exceptions to the above are the first and second strings received:
- The first string recieved is always the callsign - e.g. `M0AHN\r`. This is sent by the node and happens before any subsequent processing
- If the second string fails conversion, this is likely a manual connect by a human. WPS returns a friendly message (configurable in `wps.py`) and then disconnects

> [!IMPORTANT]
> WPS strips the SSID, if one is received from the node. The WPS user is always the callsign minus any SSID

> [!NOTE]
> The use of JSON offered many advantages when developing WPS:
> <br> 1. Conversion to JSON offers a very simple way of assuring data integrity across a number of fragmented packets, a feature of the varying PACLENs used across the network
> <br> 2. JSON is simple to use by both WPS and connected applications
> <br> 3. JSON offers complete flexibility to add / amend data when required
> <br> 4. JSON compresses well due to its repetitive use of certain characters. Overall ompression typically achieves up to a 40% reduction in packet length
> <br><br>WPS could easily add support for a different payload construct without material effort. It already recognises and supports two payload types - compressed and native JSON

## WPS Installation and Protocol Documentation

Links to the documentation in the `/docs` directory

1. [Installation](docs/installation/INSTALLATION.md)
2. [Protocol - General](docs/protocol/GENERAL.md)
3. [Protocol - Channels](docs/protocol/CHANNELS.md)
4. [Protocol - Messages](docs/protocol/MESSAGES.md)
