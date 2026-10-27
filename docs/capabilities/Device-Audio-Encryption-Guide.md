---
stoplight-id: 0mlvch9cxt893
tags: [Capabilities, DeviceAudioEncryption, Devices, Server]
---

# Device Audio Encryption Guide

## Introduction

Many of Shure's conferencing devices support encrypting audio between two or more Shure devices using the Advanced Encryption Standard (AES-256), as specified by the US Government National Institute of Standards and Technology (NIST) publication FIPS-197. This feature allows audio to be securely transported across a Dante or AES67 network, preventing any unauthorized wiretapping or snooping of conversations. Shure devices that support encryption require a symmetric key (passphrase) for the devices to encrypt the audio they send, and decrypt the audio they receive. Encryption is not supported with third-party devices.

System API Server provides APIs which satisfy the following use cases:

- Enabling audio encryption
- Changing / rotating the passphrases used for encrypting and decrypting audio
- Disabling audio encryption
- Getting audio encryption state of a device
- Subscribing to and unsubscribing from the encryption state of a device

<!-- theme: info -->

> #### Encrypted Audio Privacy Notice
>
> The API neither listens to nor records any audio signal passing through the network between Shure devices.

## Prerequisites

Before attempting to setup audio encryption between Shure devices, ensure that Dante audio routes are properly configured via either Shure Designer or Dante Controller in order for audio signals to pass between devices as intended.

- To create Dante audio routes using Shure Designer Software, refer to instructions on [How to Install Shure Designer Software and Create Dante Audio Routes](https://pubs.shure.com/guide/designer).
- To create Dante audio routes using Dante Controller, refer to instructions on [Audinate's Guide to Use Dante Controller to Create Dante Audio Routes](https://dev.audinate.com/GA/dante-controller/userguide/pdf/latest/).

## Supported Operations

### Enable Audio Encryption

The API supports enabling audio encryption on a specified device by setting an audio encryption passphrase for the device (which acts as its symmetric encryption and/or decrpytion key). Audio will pass between devices, if and only if, all the following conditions are met:

- [x] Dante audio routes exists between those devices. Refer to [Prerequisites](./x1c4f0776tion-device-audio-encryption-guide#prerequisites) section above to learn more about configuring Dante audio routes.
- [x] All routed devices' audio are unmuted.
- [x] Audio encryption is enabled on all of the routed devices.
- [x] Audio encryption passphrases match on all of the routed devices.

To enable audio encryption, call [`POST /api/v1/devices/{deviceId}/audio-encryption/enable`](https://shure.stoplight.io/docs/shure-system-api-specification/a9baabbecdbac-enable-device-audio-encryption) API with a request body containing a valid audio encryption passphrase.

```json http
{
  "method": "POST",
  "baseUrl": "https://127.0.0.1:10000",
  "url": "/api/v1/devices/{deviceId}/audio-encryption/enable",
  "headers": {
    "x-api-key": "",
    "Content-Type": "application/json"
  },
  "body": {
    "passphrase": "valid-passphrase-to-pass-to-device"
  }
}
```

<!-- theme: info -->

> #### Changing / rotating the passphrases used for encrypting and decrypting audio
> This API makes it easy to perform passphrase rotation on devices. Simply calling [`POST /api/v1/devices/{deviceId}/audio-encryption/enable`](https://shure.stoplight.io/docs/shure-system-api-specification/a9baabbecdbac-enable-device-audio-encryption) API with the new passphrase on each routed device will overwrite the existing encryption passphrase with the newer passphrase, if encryption was already enabled on the device. If encryption was previously disabled on the device, encryption will be enabled with the new passphrase.
>
> Refer to [NIST Publication of Digital Authentication Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html#memsecretver) for more information about the latest recommendations for setting passphrases.

### Disable Audio Encryption

The API supports disabling audio encryption on a specified device. Once encryption is disabled, the unencrypted audio will pass between devices, if and only if, the following conditions are met:

- [x] Dante audio routes exists between those devices. Refer to [Prerequisites](./x1c4f0776tion-device-audio-encryption-guide#prerequisites) section above to learn more about configuring Dante audio routes.
- [x] All routed devices' audio are unmuted.
- [x] Audio encryption is disabled on all of the routed devices.

To disable audio encryption, call [`POST /api/v1/devices/{deviceId}/audio-encryption/disable`](https://shure.stoplight.io/docs/shure-system-api-specification/103d04b645788-disable-device-audio-encryption) API.

```json http
{
  "method": "POST",
  "baseUrl": "https://127.0.0.1:10000",
  "url": "/api/v1/devices/{deviceId}/audio-encryption/disable",
  "headers": {
    "x-api-key": "",
    "Content-Type": "application/json"
  }
}
```

### Get Audio Encryption State

The API supports fetching the current audio encryption state on a specified device.

To get the current audio encryption state of a device, call [`GET /api/v1/devices/{deviceId}/audio-encryption/`](https://shure.stoplight.io/docs/shure-system-api-specification/cf24caf6d14a7-get-device-audio-encryption-information) API.

```json http
{
  "method": "GET",
  "baseUrl": "https://127.0.0.1:10000",
  "url": "/api/v1/devices/{deviceId}/audio-encryption/",
  "headers": {
    "x-api-key": "",
    "Content-Type": "application/json"
  }
}
```

Here is an example of how a successful JSON response may look like for a device which has audio encryption enabled:

```json
{
  "enable": true
}
```

### Subscribe to Audio Encryption State

The API supports subscribing to audio encryption state changes on a specified device. To learn more about creating WebSocket connections for making subscriptions, refer to the [Getting Started Guide](../../docs/Getting-Started.md#your-first-api-subscription).

To subscribe to audio encryption state changes on a device, call [`POST /api/v1/devices/{deviceId}/audio-encryption/subscription/{transportId}`](https://shure.stoplight.io/docs/shure-system-api-specification/f63f1bbca0e1e-subscribe-device-audio-encryption-state) API.

```json http
{
  "method": "POST",
  "baseUrl": "https://127.0.0.1:10000",
  "url": "/api/v1/devices/{deviceId}/audio-encryption/subscription/{transportId}",
  "headers": {
    "x-api-key": "",
    "Content-Type": "application/json"
  }
}
```

Here's an example of how audio encryption state change event message may look like over the WebSocket when audio encryption gets enabled on a device:

```json
{
  "envelope": {
    "eventName": "DEVICE_AUDIO_ENCRYPTION",
    "deviceId": "DD123456-0000-11DD-A000-000EDDCCCCCC"
  },
  "body": {
    "enable": true
  }
}
```

### Unsubscribe from Audio Encryption State

The API supports unsubscribing from audio encryption state changes on a specified device.

To unsubscribe from audio encryption state changes on a device, call [`DELETE /api/v1/devices/{deviceId}/audio-encryption/subscription/{transportId}`](https://shure.stoplight.io/docs/shure-system-api-specification/69a7068331354-unsubscribe-device-audio-encryption-state) API.

```json http
{
  "method": "DELETE",
  "url": "/api/v1/devices/{deviceId}/audio-encryption/subscription/{transportId}",
  "baseUrl": "https://127.0.0.1:10000"
  "headers": {
    "x-api-key": "",
    "Content-Type": "application/json"
  }
}
```
