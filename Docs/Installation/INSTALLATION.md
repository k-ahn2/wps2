# Installation

## Table of Contents 

1. [WPS Installation and Prereqs](#wps-installation-and-prereqs)
2. [Sending a JSON object to WPS (Javascript Example)](#sending-a-json-object-to-wps---a-javascript-example)
3. [Node Integration - Interfacing with BPQ or Xrouter](#node-integration---interfacing-with-bpq-or-xrouter)
4. Configuring `env.json`
5. Files

## WPS Installation and Prereqs

> [!NOTE]
> WPS has only been tested on a Raspberry Pi running Raspbian. There is no known reason it shouldn't run in any Python environment. Please share your feedback so we can update the docs for others

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