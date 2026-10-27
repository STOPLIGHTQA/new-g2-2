---
tags: [Security, Server]
stoplight-id: hgr8wc6jvajhe
---

# API Authorization

## Overview

Each call to System API must be authorized. This is accomplished by the use of a shared secret, which is sent with each HTTP request. System API will generate this secret for you. 

## REST API Authorization
All HTTP requests to the System API require authorization. If this is not provided, the result will be **HTTP 401 Unauthorized**, indicating that the shared secret must be included in requests:
![Request_Unauthorized.png](https://shure.widen.net/content/jmjlv9tehd/original/OpenApiSpecificationUI-HTTP401.png?u=dsog5w&download=true)

The shared secret is an API key which is generated when System API runs for the first time. This key is found at **C:\ProgramData\Shure\SystemAPI\Standalone\Security\sharedkey.txt**.
<!-- theme: info -->
> #### Shared Secret File Permissions on Windows
>
> When you install System API, the user account of the user who ran the installer will be added to a group which has read access to the shared secret file. This group will be labeled **ShureSystemAPI**, and you can manage it by running *lusrmgr* from a command prompt:![SystemAPI_Group.png](https://shure.widen.net/content/txlseh0y5y/original/LocalUsersAndGroupsManager-Windows.png?u=dsog5w&download=true)If it becomes necessary to delegate access to the file to other users, simply add their user accounts to the same group.

You will need to set the shared secret as the value for the HTTP header `x-api-key` for each API request you make:
```json http
{
  "method": "GET",
  "baseUrl": "https://127.0.0.1:10000",
  "url": "/api/v1/devices",
  "headers": {
    "accept": "application/json",
    "x-api-key": ""
  }
}
```

<!-- theme: danger -->
> #### Protect Your Shared Secret
>
> It is your responsibility to make sure the shared secret does not fall into the wrong hands. Anybody who obtains the secret key and has access to your server can use the API without restriction!
