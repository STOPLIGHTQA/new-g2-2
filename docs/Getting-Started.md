---
tags: [Server, Devices, Subscriptions, Capabilities, Installation]
stoplight-id: zxmrlx3g0buuj
---

# Getting Started With System API Server
## How to Use the Docssdsds
Browse the API documentation directly via the web-hosted copycvxvxvx here on Stoplight or on your server using the interactive API documentation provided with System API Server to further explore the API surface. Each has their own benefits which are summarized below, but are otherwise equivalent in functionality.

### Stoplight Docs
Unique features:
1. Read markdown (like this) which describes how to interact with the API more broadly. In particular, step-by-step guides, [UML sequence diagrams](https://wikipedia.org/wiki/Sequence_diagram), and more will be available here to explain how to use more advanced features of the API.
2. Generate example code snippets for making HTTP requests in your preferred language in order to try things out in your application with minimal code adjustment.

### Interactive API Docs
The interactive API documentation (a.k.a "Swagger", "Open API Specification browser") is available on the server where System API Server is installed. By default, it is available at `https://127.0.0.1:10000/rest`.

Unique features:
1. Call APIs against your own devices running on your local device network.
2. Browse the API documentation on the server when Internet access is not available.

<!-- theme: warning -->
> You may need to substitute the server's IP address and port number depending on how you have configured your server instance, or if you are accessing the server from a separate machine. This is also true of all subsequent requests in the documentation.

## System Requirements
### OS Support
System API Server requires one of the following operating systems in 64-bit version:
- [x] Windows 10
- [x] Windows 11
- [x] Windows Server 2016
- [x] Windows Server 2019
- [x] Windows Server 2022
- [x] Windows Server 2025

### Hardware Requirements
The following minimum and recommended requirements apply to either the physical machine, or to the virtual machine
running the software, if applicable.

**Minimum** | **Recommended**
---------|----------
 **Dual Core** 64-bit Intel or AMD processor | **Quad Core** 64-bit Intel or AMD processor
 **2 GB** RAM | **8 GB** RAM

*ARM processors are not supported at this time on Windows.*

## Your First API Request - Get Device Info
With the server installed, you can now make requests to monitor and control Shure devices.

To see information about all discovered Shure devices on your network, call the [`GET /v1/devices`](../API/System-API.json/paths/~1v1~1devices/get) endpoint.

```json http
{
  "method": "GET",
  "baseUrl": "https://127.0.0.1:10000",
  "url": "/api/v1/devices",
  "headers": {
    "x-api-key": "",
    "Content-Type": "application/json"
  }
}
```

Here is an example of what the response may look like if there were only one device on the network.

```json
[
  {
    "compatibility": "COMPATIBLE_LATEST",
    "deviceState": "ONLINE",
    "hardwareIdentity": {
      "deviceId": "DDFF8B8D-0000-11DD-A000-000EDDCCCCCC",
      "serialNumber": "2TJ06681671"
    },
    "softwareIdentity": {
      "firmwareVersion": "6.0.24",
      "model": "MXA710",
      "firmwareValid": true
    },
    "communicationProtocol": {
      "name": "ESTA.DMP",
      "address": "192.168.178.10"
    },
    "capabilities": [
      "audio-mute",
      "audio-network",
      "control-network",
      "name",
      "user-presets",
      ...
    ]
  }
]
```

`deviceState` describes the level of interactability that the server is able to maintain with the device at any moment. The different states, as well as the state machine and its impacts on your software, are described at length in [Device Discovery Lifecycle Guide](../docs/Device-Discovery-Lifecycle.md).

`hardwareIdentity` is a collection of read-only device information used for identifying a *particular* instance of a device.  `hardwareIdentity.deviceId` is used to uniquely address an individual device in the API endpoints. `hardwareIdentity.serialNumber`, on the other hand, is useful for asset tracking, registering your product, and creating a warranty claim.

`softwareIdentity` is used to describe the device's firmware, as well as the device's `model`. Note that there can be multiple product combinations for one `model`, as is the case for [Shure's MXA710 Linear Array Microphone.](https://www.shure.com/en-US/products/microphones/mxa710)

The `capabilities` array describes all of the features which a device is capable of performing. **It should be used for all logical checks for whether or not a device is capable of performing the behaviors of a given feature.**

<!-- theme: success -->
> #### Capabilities make your code more maintainable!
>
> As an app developer, `capabilities` allow you to write code that is *feature-focused instead of device-focused*! When you use `capabilities` to check for device features, your app code will meet the [Open-Closed Principle](https://wikipedia.org/wiki/Open%E2%80%93closed_principle) of [SOLID](https://wikipedia.org/wiki/SOLID) when Shure adds support for more devices (or newer firmware versions).
>
> This works on the same principle as [interfaces in object-oriented programming](https://wikipedia.org/wiki/Interface_\(computing\)#In_object-oriented_languages); the interaction with the capability is considered interchangeable when...
> - Two or more devices implement the *same* capability
> - Two or more `firmwareVersion`s of the same  `model` of device implement the *same* capability
>
> Simply put, your code will be better equipped to work with new firmware updates, and the devices of tomorrow.

<!-- theme: info -->
> #### The `capabilities` array does not update in realtime!
>
> Note that while *inspired* by the concept of [HATEOAS](https://wikipedia.org/wiki/HATEOAS), your app will still be responsible for state management.
>
> For example, devices which have not yet entered the `deviceState` of `ONLINE` will not accept being updated via the API, except for a few minor exceptions noted in [Device Discovery States](../docs/Device-Discovery-Lifecycle.md).

`compatibility` describes the status of compatibility between this software, and the discovered device's firmware. Devices must be compatible in order to interact with their features via the API.
Compatibility  | What It Means | Recommended Action(s) to Maximize Experience
---------|----------|---------
 `COMPATIBLE_LATEST` | The device firmware is the latest version tested with this software version. | N/A
 `COMPATIBLE_OLD` | The device firmware is behind the latest available, but can be completely interacted with. | Firmware Update Recommended
 `COMPATIBLE_NEW` | The device firmware is ahead of the latest version this software was tested with, but can be completely interacted with. | System API Server Update Recommended
 `INCOMPATIBLE_TOO_OLD` | The device firmware is not supported, and therefore no capability interaction is supported. | Firmware Update Required
 `INCOMPATIBLE_TOO_NEW` | The device firmware is not supported, and therefore no capability interaction is supported. | System API Server Update Required

## Working With Device Capabilities
In order to discover which APIs are available to work with for a given device, use the steps described below.

1. **Examine the `capabilities` array of the device** to determine what features it is capable of.

2. **Find APIs that apply to each device capability** using APIs ending in `/capabilities`, such as [`GET /v1/devices/{deviceId}/capabilities`](../API/System-API.json/paths/~1v1~1devices~1{deviceId}~1capabilities/get).

3. **Search the API documentation** for the `operationId` that best describes the functionality of interest.

### Example - Saving and Loading Custom Presets
To demonstrate, let's walk through how to find the APIs in the documentation which describe saving, loading, and retrieving custom preset settings.

<!--
type: tab
title: 1. Examine the Capabilities
-->
The [`GET /v1/devices`](../API/System-API.json/paths/~1v1~1devices/get) API response above returns a `capabilities` array along with the device. Notice that `"user-presets"` is a capability of our [MXA710 Linear Array Microphone](https://www.shure.com/en-US/products/microphones/mxa710).
```json
  "capabilities": [
    "audio-mute",
    "audio-network",
    "control-network",
    "name",
    "user-presets",
    ...
  ]
```

<!--
type: tab
title: 2. Find APIs for Capabilities
-->
The [`GET /v1/devices/{deviceId}/capabilities`](../API/System-API.json/paths/~1v1~1devices~1{deviceId}~1capabilities/get) API can be used to find the APIs which correspond to the `"user-presets"` capability. For the sake of brevity, only the `"user-presets"` entry has been copied below.

```json title="Example Response"
[
	{
		"name": "user-presets",
		"operationIds": [
			"userPresets",
			"subscribeUserPresets",
			"unsubscribeUserPresets",
			"renameUserPreset",
			"saveUserPreset",
			"loadUserPreset",
			"clearUserPreset"
		]
	}
}
```

```json http
{
  "method": "GET",
  "baseUrl": "https://127.0.0.1:10000",
  "url": "/api/v1/devices/{deviceId}/capabilities",
  "headers": {
    "x-api-key": "",
    "Content-Type": "application/json"
  }
}
```

<!--
type: tab
title: 3. Search the Docs
-->
Using the search bar, we can search for the `operationIds` which are relevant to the interaction we are trying to perform.

![Stoplight-SearchBar.png](https://shure.widen.net/content/izupcebwgo/original/Stoplight-SearchBar.png?u=dsog5w&download=true)

![Stoplight-SearchOID.png](https://shure.widen.net/content/znhsi3nsix/original/Stoplight-SearchOID.png?u=dsog5w&download=true)

<!-- type: tab-end -->

## Your First API Subscription
Subscriptions provide a real-time data stream from the device containing information that has previously been subscribed to. This can be used to avoid needing to repeatedly call the API for receiving updates.

### Open a WebSocket
1. Use a tool for interacting with [WebSockets](https://wikipedia.org/wiki/WebSocket) such as [Hoppscotch](https://hoppscotch.io/realtime).
2. Open a new WebSocket at the following location `wss://127.0.0.1:10000/api/v1/subscriptions/websocket/create`. Note that the shared secret used for HTTP requests is not required.
    ![Hoppscotch.gif](https://shure.widen.net/content/6lccpelkhu/web/Hoppscotch.gif?u=dsog5w&animate=true)
3. The connection will immediately return a message containing a `transportId`, which is a unique ID for the newly opened WebSocket. This will be required for subsequent subscription requests to determine which WebSocket you would like to receive updates over.
    ```json
    {
      "transportId": "9d56208c-8085-433c-ae21-6f668302806e"
    }
    ```

### Subscribe to Data
After creating the WebSocket as described in the previous section, we can subscribe to events via the REST API. In turn, this will allow System API Server to deliver asynchronous updates over the WebSocket.

The example below will demonstrate how to subscribe to whether or not the device is currently identifying (flashing the lights on the device), but the process is nearly identical for other capabilities that can be subscribed to.

1. Subscribe to device identify using the information from previous steps by calling the [`POST /v1/devices/{deviceId}/identify/subscription/{transportId}`](../API/System-API.json/paths/~1v1~1devices~1{deviceId}~1identify~1subscription~1{transportId}/post) API.
    ```json http
    {
      "method": "POST",
      "baseUrl": "https://127.0.0.1:10000",
      "url": "/api/v1/devices/{deviceId}/identify/subscription/{transportId}",
      "headers": {
        "x-api-key": "",
        "Content-Type": "application/json"
      }
    }
    ```
2. The WebSocket is now subscribed to the identify state of the device. To illustrate this, identify the device using the [`POST /v1/devices/{deviceId}/identify/start`](../API/System-API.json/paths/~1v1~1devices~1{deviceId}~1identify~1start/post) API.
    ```json http
    {
      "method": "POST",
      "baseUrl": "https://127.0.0.1:10000",
      "url": "/api/v1/devices/{deviceId}/identify/start",
      "headers": {
        "x-api-key": "",
        "Content-Type": "application/json"
      }
    }
    ```
3. The WebSocket connection opened earlier should now show a newly received message.
    ```json
    {
      "envelope": {
        "eventName": "DEVICE_IDENTIFY",
        "deviceId": "{deviceId}" // Your device's ID will be returned here
      },
      "body": {
        "identifying": true
      }
    }
    ```

4. The device will naturally stop identifying after some time, or manually stop identifying the device  using the [`POST /v1/devices/{deviceId}/identify/stop`](../API/System-API.json/paths/~1v1~1devices~1{deviceId}~1identify~1stop/post) API.
    ```json http
    {
      "method": "POST",
      "baseUrl": "https://127.0.0.1:10000",
      "url": "/api/v1/devices/{deviceId}/identify/stop",
      "headers": {
        "x-api-key": "",
        "Content-Type": "application/json"
      }
    }
    ```

    This will result in another WebSocket message:

    ```json
    {
      "envelope": {
        "eventName": "DEVICE_IDENTIFY",
        "deviceId": "{deviceId}" // Your device's ID will be returned here
      },
      "body": {
        "identifying": false
      }
    }
    ```
5. To unsubscribe from identify information on this WebSocket, call the [`DELETE /{deviceId}/identify/subscription/{transportId}`](../API/System-API.json/paths/~1v1~1devices~1{deviceId}~1identify~1subscription~1{transportId}/delete) API.
    ```json http
    {
      "method": "DELETE",
      "baseUrl": "https://127.0.0.1:10000",
      "url": "/api/v1/devices/{deviceId}/identify/subscription/{transportId}",
      "headers": {
        "x-api-key": "",
        "Content-Type": "application/json"
      }
    }
    ```
    You are no longer subscribed to the device's identify state change information at this point, so you will no longer see any identifying state change event messages over the WebSocket.

<!-- theme: info -->
> #### One WebSocket Connection Can Be Used for Multiple Subscriptions
> One WebSocket connection is sufficient for handling multiple subscriptions. You can distinguish different types of events from one another via the `envelope.eventName` field present in each event message sent through the WebSocket. The `envelope.eventName` field value will be unique for each type of event.

<!-- theme: info -->
> #### Multiple WebSockets Can Be Used
> It is also possible to use multiple WebSockets to receive different types of data. Choosing which WebSocket to receive which data is based on which `transportId` is provided in each subscription request.
