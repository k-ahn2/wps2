# Installation

## Table of Contents 

1. [WPS Installation and Prereqs](#wps-installation-and-prereqs)
2. [Sending a JSON object to WPS (Javascript Example)](#sending-a-json-object-to-wps---a-javascript-example)
3. [Node Integration - Interfacing with BPQ or Xrouter](#node-integration---interfacing-with-bpq-or-xrouter)
4. [Configuring `env.json`](#configuring-envjson)
5. [WPS System and Log Files](#wps-system-and-log-files)

[Return to README](/README.md)

## WPS Installation and Prereqs

> [!NOTE]
> WPS has only been tested on a Raspberry Pi running Raspbian. There is no known reason it shouldn't run in any Python environment. Please share your feedback so we can update the docs for others!

1. Clone the repository using `git clone https://github.com/k-ahn2/wps`
2. Go to the `wps` directory
3. Run `python3 wps.py`

This should start WPS with a default configuration. When running for the first time, WPS will create and initialise the database `wps.db`, plus `wps.log` and `db.log`

Check for errors in the console. Confirmation of the TCP Port is shown - check this matches the port in BPQ or Xrouter.

## Node Integration - Interfacing with BPQ or Xrouter

> [!NOTE]
> Xrouter node setup to be added

> [!WARNING]
> This section requires basic familiarity with BPQ configuration files and ideally custom application setup. Examples shown but please consult the BPQ documentation for more information

### BPQ Config Entries (abridged)
Conf
```
PORT
   PORTNUM=8
   DRIVER=TELNET
   CONFIG
   DisconnectOnClose=1       ; Ensures the client is fully disconnected if the TCP Port disconnects, not returned to the node
   CMDPORT 63001 63002       ; Port and position must match the APPLICATION entry below. HOST 0 is 63001, HOST 1 is 63002
   MAXSESSIONS=25            ; Maxmimum simultaneous connections, set to desired value
   ....
END PORT
```

### BPQ Simple Application Config
`APPLICATION 1,WPS,C 8 HOST 0 TRANS`

### BPQ Config with Callsign and NETROM
`APPLICATION 1,WPS,C 8 HOST 0 TRANS,MB7NPW-9,WTSPAC,200,WTSPAC`

```
APPLICATION 6,SYSINFO,C 9 HOST 2 NOCALL K S
                |       |    |   | |      | |
                |       |    |   | |      | Return to node upon exit (omit if giving app its own NODECALL-#)
                |       |    |   | |      Keep-alive to prevent premature exit of application
                |       |    |   | Do not pass call sign to app (omit if you want it via stdin)
                |       |    |   CMDPORT #
                |       |    Localhost
                |       Connect to Telnet PORT #
                App name entered at user prompt
```

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

## Configuring `env.json`

There is no requirement to edit `env.json`to get started - the default configuration will enable WPS to run and function. Edit env.json if you:
- Change the TCP Port
- Increase WPS Applciation or Database logging
- Configure notifications 

| Parameter | Data Type | Default | Notes |
| - | :-: | :-: | :- |
|`environment`|String|`Dev`|Historically used to suppress certain functions outside Production, but currently unused|
|`minClientVersion`|Number|`1.0`|Server configurable number that is used by client side code to prompt the user to upgrade. WPS reloads this value from env.json on every connect, meaning the value can be updated and used at runtime|
|`socketTcpPort`|Number|`63001`|TCP Port that WPS listens on. Needs to match the APPLICATION port setup in BPQ or Xrouter|
|`dbFilename`|String|`wps.db`|The filename to use for the Sqlite database. Enables a different filename to be used to differentiate between development and produciton, for example|
|`minWpsLogLevel`|String|`ERROR`|For application logging in `wps.log`, either `ERROR` for errors only, or `INFO` for everything. WPS contains a lot of INFO logging and could be optimised - beware of `wps.log` file size|
|`minDbLogLevel`|String|`ERROR`|For database logging in `db.log`, Either `ERROR` for errors only, or `INFO` for everything. When in INFO mode, `db.log` will contain every database query and the function response - beware of `db.log` file size|
|`notificationsEnabled`|Boolean|`false`|Set to `true` if `notificationsProdId` and `notificationsProdRestKey` are configured and you want to enable notifications|
|`notificationsProdId`|String|`""`|Add the Id of your OneSignal Service|
|`notificationsProdRestKey`|String|`""`|Add the REST API key of your OneSignal Service|
|`channels`|Object|`{}`|Add a key / value pair corresponding to the channel id (`cid`) and channel name. Used by notifications to include the Channel Name in the notification description - e.g. when a new post arrives in `cid` = 1, send "New Post from #packet-general"|

### Sample `env.json`

```json
{
    "environment": "Dev",
    "minClientVersion": 1.0,
    "socketTcpPort": 63001,
    "dbFilename": "wps.db",
    "minWpsLogLevel": "ERROR",
    "minDbLogLevel": "ERROR",
    "notificationsEnabled": false,
    "notificationsProdId": "",
    "notificationsProdRestKey": "",
    "channels": {
        "0": "packet-tech",
        "1": "packet-general"
    }
}
```

## WPS System and Log Files

| File | Overview |
| - | :- |
|`wps.py`|The main WPS application - includes all application logic, thread handling and TCP listener|
|`db.py`|Contains functions to handle every interaction between the WPS application and the database - e.g. `dbUserSearch`, `dbUserUpdate` or `dbGetOnlineUsers`|
|`wps.log`|Application logging, default ERROR only|
|`db.log`|Database logging, default ERROR only|
|`backup.py`|Run to create a JSON file containing every user, message and post object in the database. Reads `env.json` to determine the database filename from `dbFilename`. Any Sqlite supported backup method would be valid|
|`env.json`|Environment configuration variables|

