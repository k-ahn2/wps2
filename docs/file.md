## Type C - Connect
This is the first data exchange after connect - the client sends a Type ‘c’ object to the server, which uses this data to determine the client state and what to return

| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |

### Schematic
| Friendly Name | Key | Sample Values | Data Type | Notes |
|-------------|-------------|-------------|-------------|-------------|

| Friendly Name | Key | Sample Values | Data | Notes |
| ------------- | ------------- | ------------- | ------------- | ------------- |
|Type|`t`|`c`|String|Always type c|
|Name|`n`|`Kevin`|String|Name as entered in WhatsPac Client|
|Callsign|`c`|`M0AHN`|String|Callsign as entered in WhatsPac Client, minus the SSID if added|
|Last Message|`lm`|`1740299150`|Number|Timestamp of last message - seconds since epoch|
|Last Emoji|`le`|`1740266497`|Number|Timestamp of last emoji - seconds since epoch|
|Last Edit|`led`|`1739318078`|Number|Timestamp of last edit - seconds since epoch|
|Version|`v`|`0.44`|Number|Version of client, used for logging only|
|Channel Connect|`cc`|`[]`|Array of Objects|Contains one JSON object per channel subscribed|


|Channel Connect Objects|-|-|-|-|
|Channel Id|`cid`|`2`|Number|The Channel Id|
|Last Post|`lp`|`1740251305826`|Number|Timestamp of last message - milliseconds since epoch|
|Last Emoji|`le`|`1740252223588`|Number|Timestamp of last emoji - milliseconds since epoch|
|Last Edit|`led`|`1738869165936`|Number|Timestamp of last edit - milliseconds since epoch|
### JSON Example
```
{
   "t":"c",
   "n":"Kevin",
   "c":"M0AHN",
   "lm":1740299150,
   "le":1740266497,
   "led":1739318078,
   "v":0.44,
   "cc":[
      {
         "cid":3,
         "lp":1740242101095,
         "le":1740247089774,
         "led":1740250727684
      },
      {
         "cid":4,
         "lp":1740251305826,
         "le":1740252223588,
         "led":1738869165936
      }
   ]
}

```
