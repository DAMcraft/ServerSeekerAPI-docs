# API documentation
All types are strings if not specified otherwise.
A successful request will always return a `200` status code.
If it failed, it will return a json object with the following structure:
```json
{
    "error": "The error message"
}
```

### Getting your api_key
Obtaining your api_key is easy. 
Just open the following url in your browser and login with your discord account. 
You will be redirected to the callback url which will automatically generate your api_key.

```
https://discord.com/api/oauth2/authorize?client_id=1087083964432404590&redirect_uri=https%3A%2F%2Fserverseeker.damcraft.de%2Fapi%2Fv1%2Fdiscord_callback&response_type=code&scope=identify
```
---

If you want to get the api_key locally, simply start a server on the port 7637 .
```
https://discord.com/api/oauth2/authorize?client_id=1087083964432404590&redirect_uri=http%3A%2F%2F127.0.0.1%3A7637%2F&response_type=code&scope=identify
```
This link will redirect to the callback on 127.0.0.1:7637, and you can get the `code` as a GET parameter.

This can then be sent using a POST request to the following url to get your api_key.
```
https://serverseeker.damcraft.de/api/v1/get_token
```
The POST request should look like this:
```json
{
    "code": "The code you got from the callback",
    "usage": "(optional) How the api_key will be used." 
}
```
---
## Using the api_key to find servers

Great! Now that you got your api_key, you can start using the api!
#### The base url is __**`https://serverseeker.damcraft.de`**__.

### /api/v1/whereis

| Parameter | Required            | Type   | Description                             |
|-----------|---------------------|--------|-----------------------------------------|
| api_key   | yes                 | string | Your api_key                            |
| name      | either name or uuid | string | The name of the player you want to find |
| uuid      | either name or uuid | string | The uuid of the player you want to find |

example:
```json5
{
    "api_key": "...", // Your api_key
    "name": "DAMcraft" // The name of the player you want to find. If you choose name, you can't choose uuid and vice versa
}
```
returns:
```json5
{
  "data": [ // An array of servers the player was seen on. Limited to 1000
    {
      "server": "109.123.240.84:25565", // The ip and port of the server
      "uuid": "68af4d98-24a2-41b6-96bc-a9c2ef9b397b", // The uuid of the player 
      "name": "DAMcraft", // The name of the player
      "last_seen": 1683790506 // The last time the player was seen on the server (unix timestamp)
    }, ...
  ]
}
```
---
### /api/v1/server_info

| Parameter | Required           | Type   | Description                                              |
|-----------|--------------------|--------|----------------------------------------------------------|
| api_key   | yes                | string | Your api_key                                             |
| ip        | yes                | string | The ip of the server you want to get information about   |
| port      | no (default=25565) | int    | The port of the server you want to get information about |

example:
```json5
{
  "api_key": "...", // Your api_key
  "ip": "109.123.240.84", // The ip of the server
  "port": 25565  // The port of the server (defaults to 25565)
}
```
returns:
```json5
{
  "cracked": null, // Boolean. Whether the server is cracked or not. null if unknown
  "description": "A Minecraft Server", // The description aka motd of the server
  "last_seen": 1683760904, // The last time the server was seen (unix timestamp)
  "max_players": 20, // The maximum amount of players the server can hold
  "online_players": 2, // The amount of players currently online
  "protocol": 762, // The protocol version of the server
  "version": "Spigot 1.19.4", // The version of the server
  "players": [ // An array of when which players were seen on the server. Limited to 1000
    {
      "last_seen": 1683790506, // The last time the player was seen on the server (unix timestamp)
      "name": "DAMcraft", // The name of the player
      "uuid": "68af4d98-24a2-41b6-96bc-a9c2ef9b397b" // The uuid of the player
    }, ...
  ]
}
```
---
### /api/v1/servers
(if a param is not given, any value is accepted, except for online_after, which defaults to the last scan time)

| Parameter      | Required                                    | Type   | Description                                                                                                                                                |
|----------------|---------------------------------------------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| api_key        | yes                                         | string | Your api_key                                                                                                                                               |
| online_players | no                                          | int    | The minimum amount of online players the server should have                                                                                                |
| max_players    | no                                          | int    | The maximum amount of players the server should hold                                                                                                       |
| cracked        | no                                          | bool   | Whether the server should be cracked or not                                                                                                                |
| description    | no                                          | string | The description aka motd of the servers should have                                                                                                        |
| protocol       | no                                          | int    | The protocol version of the server                                                                                                                         |
| online_after   | no (default=last_scan!!!, set to 0 for all) | int    | The server should have been online after this unix timestamp. Defaults to showing all servers which were online at last ping. Set to 0 to show all servers |

example:
```json5 
{
    "api_key": "...", // Your api_key
    "online_players": 1, // The amount of online players the servers should have
    "max_players": 20, // The maximum amount of players the servers should hold
    "cracked": true, // Whether the servers should be cracked or not
    "description": "A Minecraft Server\u00a7r", // The description aka motd of the servers should have
    "protocol": 754, // The protocol version the servers should have
    // leave out online_after to show all servers which were online at last ping
}
```
returns:
```json5
{
    "data": [ // An array of servers matching the criteria. Limited to 128
        {
            "cracked": false, // Boolean. Whether the server is cracked or not. null if unknown
            "description": "\u00a7rA Minecraft Server\u00a7r", // The description aka motd of the server
            "last_seen": 1686105138, // The last time the server was seen (unix timestamp)
            "max_players": 20, // The maximum amount of players the server can hold
            "online_players": 1, // The amount of players currently online
            "protocol": 760, // The protocol version of the server
            "version": "Paper 1.19.2", // The version of the server
            "server": "181.75.128.9:8185" // The ip and port of the server
        } ...
    ]
}
```
