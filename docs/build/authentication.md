---
sidebar_label: Authentication
sidebar_position: 2
description: Learn how to create and configure an XMTP client
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Build authentication with XMTP

The XMTP message API revolves around a network client that allows retrieving and sending messages to other network participants. A client must be connected to a wallet on startup. The client will request a wallet signature in two cases:

1. To sign the newly generated key bundle. This happens only the very first time when key bundle is not found in storage.
2. To sign a random salt used to encrypt the key bundle in storage. This happens every time the client is started (including the very first time).

## Create a client

A client is created that requires passing in a connected wallet that implements the [Signer](https://github.com/xmtp/xmtp-js/blob/main/src/types/Signer.ts) interface.
[Use client configuration options](#configure-the-client) to change parameters of a client's network connection.

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript"  attributes={{className: "js_tab"}}>

```ts
import { Client } from "@xmtp/xmtp-js";
// Create the client with a `Signer` from your application
const xmtp = await Client.create(wallet, { env: "dev" });
```

</TabItem>
<TabItem value="swift" label="Swift"  attributes={{className: "swift_tab"}}>

```swift
import XMTP

// Create the client with a `SigningKey` from your app
let client = try await Client.create(
  account: account, options: .init(api: .init(env: .production)))
```

</TabItem>
<TabItem value="dart" label="Dart"  attributes={{className: "dart_tab"}}>

```dart
/*The client has two constructors: `createFromWallet` and `createFromKeys`.

The first time a user uses a new device, they should call `createFromWallet`. This will prompt them
to sign a message to do one of the following: Create a new identity (if they're new) or enable their existing identity (if they've used XMTP before)

When this succeeds, it configures the client with a bundle of `keys` that can be stored securely on
the device.*/

var api = xmtp.Api.create();
var client = await Client.createFromWallet(api, wallet);
await mySecureStorage.save(client.keys.writeToBuffer());

//The second time a user launches the app they should call `createFromKeys` using the stored `keys` from their previous session.

var stored = await mySecureStorage.load();
var keys = xmtp.PrivateKeyBundle.fromBuffer(stored);
var api = xmtp.Api.create();
var client = await Client.createFromKeys(api, keys);
```

</TabItem>
<TabItem value="kotlin" label="Kotlin"  attributes={{className: "kotlin_tab"}}>

```kotlin
// Create the client with a `SigningKey` from your app
val options =
    ClientOptions(
        api = ClientOptions.Api(env = XMTPEnvironment.PRODUCTION, isSecure = true)
    )
val client = Client().create(account = account, options = options)
```

</TabItem>
<TabItem value="react" label="React"  attributes={{className: "react_tab"}}>

```tsx
const { client, error, isLoading, initialize } = useClient();

const handleConnect = useCallback(async () => {
  await initialize({ signer });
}, [initialize]);

if (error) {
  return "An error occurred while initializing the client";
}

if (isLoading) {
  return "Awaiting signatures...";
}
```

</TabItem>
<TabItem value="rn" label="React Native"  attributes={{className: "rn_tab"}}>

```tsx
import { Client } from "@xmtp/xmtp-react-native";
// Create the client with a `Signer` from your application
const xmtp = await Client.create(wallet);
```

</TabItem>
</Tabs>

## Saving keys

You can export the unencrypted key bundle using the static method `getKeys`, save it somewhere secure, and then provide those keys at a later time to initialize a new client using the exported XMTP identity.

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript"  attributes={{className: "js_tab"}}>

```jsx
// Get the keys using a valid Signer. Save them somewhere secure.
import { loadKeys, storeKeys } from "./helpers";

let keys = loadKeys(address);
if (!keys) {
  keys = await Client.getKeys(signer, {
    ...clientOptions,
    // we don't need to publish the contact here since it
    // will happen when we create the client later
    skipContactPublishing: true,
    // we can skip persistence on the keystore for this short-lived
    // instance
    persistConversations: false,
  });
  storeKeys(address, keys);
}
const client = await Client.create(null, { privateKeyOverride: keys });
```

We are using the following helper funtions to store and retrieve the keys

```ts
// Create a client using keys returned from getKeys
const ENCODING = "binary";

export const buildLocalStorageKey = (walletAddress: string) =>
  walletAddress ? `xmtp:${"dev"}:keys:${walletAddress}` : "";

export const loadKeys = (walletAddress: string): Uint8Array | null => {
  const val = localStorage.getItem(buildLocalStorageKey(walletAddress));
  return val ? Buffer.from(val, ENCODING) : null;
};

export const storeKeys = (walletAddress: string, keys: Uint8Array) => {
  localStorage.setItem(
    buildLocalStorageKey(walletAddress),
    Buffer.from(keys).toString(ENCODING),
  );
};

export const wipeKeys = (walletAddress: string) => {
  // This will clear the conversation cache + the private keys
  localStorage.removeItem(buildLocalStorageKey(walletAddress));
};
```

</TabItem>
<TabItem value="swift" label="Swift"  attributes={{className: "swift_tab"}}>

```swift
// Create the client with a `SigningKey` from your app
let client = try await Client.create(
  account: account, options: .init(api: .init(env: .production)))

// Get the key bundle
let keys = client.privateKeyBundle

// Serialize the key bundle and store it somewhere safe
let keysData = try keys.serializedData()
```

Once you have those keys, you can create a new client with `Client.from`:

```swift
let keys = try PrivateKeyBundle(serializedData: keysData)
let client = try Client.from(bundle: keys, options: .init(api: .init(env: .production)))
```

</TabItem>
<TabItem value="kotlin" label="Kotlin"  attributes={{className: "kotlin_tab"}}>

```kotlin
// Create the client with a `SigningKey` from your app
val options =
    ClientOptions(
        api = ClientOptions.Api(env = XMTPEnvironment.PRODUCTION, isSecure = true)
    )
val client = Client().create(account = account, options = options)

// Get the key bundle
val keys = client.privateKeyBundleV1

// Serialize the key bundle and store it somewhere safe
val serializedKeys = PrivateKeyBundleV1Builder.encodeData(v1)
```

Once you have those keys, you can create a new client with `Client().buildFrom()`:

```kotlin
val keys = PrivateKeyBundleV1Builder.fromEncodedData(serializedKeys)
val client = Client().buildFrom(bundle = keys, options = options)
```

</TabItem>
<TabItem value="react" label="React"  attributes={{className: "react_tab"}}>

```tsx
import { Client, useClient } from "@xmtp/react-sdk";

const { initialize } = useClient();
// get the keys using a valid Signer
const keys = await Client.getKeys(signer);
// create a client using keys returned from getKeys
await initialize({ keys, signer });
```

</TabItem>
<TabItem value="rn" label="React Native"  attributes={{className: "rn_tab"}}>

```js
import { Client } from "@xmtp/xmtp-react-native";
// Get the keys using a valid Signer. Save them somewhere secure.
const keys = await Client.exportKeyBundle();
// Create a client using keys returned from getKeys
const client = await Client.createFromKeyBundle(keys, "dev");
```

</TabItem>
</Tabs>

## Configure the client

Configure a client's network connection and other options using these client creation parameters:

Set the `env` client option to `dev` while developing. Set it to `production` before you launch.

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript"  attributes={{className: "js_tab"}}>

| Parameter                 | Default                                                                           | Description                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| appVersion                | `undefined`                                                                       | Add a client app version identifier that's included with API requests. **Production apps are strongly encouraged to set this value.**<br/>For example, you can use the following format: `appVersion: APP_NAME + '/' + APP_VERSION`.<br/>Setting this value provides telemetry that shows which apps are using the XMTP client SDK. This information can help XMTP developers provide app support, especially around communicating important SDK updates, including deprecations and required upgrades. |
| env                       | `dev`                                                                             | Connect to the specified XMTP network environment. Valid values include `dev`, `production`, or `local`. For important details about working with these environments, see [XMTP `production` and `dev` network environments](#xmtp-production-and-dev-network-environments).                                                                                                                                                             |
| apiUrl                    | `undefined`                                                                       | Manually specify an API URL to use. If specified, value of `env` will be ignored.                                                                                                                                                                                                                                                                                                                                                        |
| keystoreProviders         | `[StaticKeystoreProvider, NetworkKeystoreProvider, KeyGeneratorKeystoreProvider]` | Override the default behavior of how the Client creates a Keystore with a custom provider. This can be used to get the user's private keys from a different storage mechanism.                                                                                                                                                                                                                                                           |
| persistConversations      | `true`                                                                            | Maintain a cache of previously seen V2 conversations in the storage provider (defaults to `LocalStorage`).                                                                                                                                                                                                                                                                                                                               |
| skipContactPublishing     | `false`                                                                           | Do not publish the user's contact bundle to the network on Client creation. Designed to be used in cases where the Client session is short-lived (for example, decrypting a push notification), and where it is known that a Client instance has been instantiated with this flag set to false at some point in the past.                                                                                                                |
| codecs                    | `[TextCodec]`                                                                     | Add codecs to support additional content types.                                                                                                                                                                                                                                                                                                                                                                                          |
| maxContentSize            | `100M`                                                                            | Maximum message content size in bytes.                                                                                                                                                                                                                                                                                                                                                                                                   |
| preCreateIdentityCallback | `undefined`                                                                       | `preCreateIdentityCallback` is a function that will be called immediately before a [Create Identity](/docs/concepts/account-signatures#sign-to-create-an-xmtp-identity) wallet signature is requested from the user.                                                                                                                                                                                                                     |
| preEnableIdentityCallback | `undefined`                                                                       | `preEnableIdentityCallback` is a function that will be called immediately before an [Enable Identity](/docs/concepts/account-signatures#sign-to-enable-an-xmtp-identity) wallet signature is requested from the user.                                                                                                                                                                                                                    |

</TabItem>
<TabItem value="swift" label="Swift"  attributes={{className: "swift_tab"}}>

| Parameter | Default | Description                                                                                                                                                                                                                                                                     |
| --------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| appVersion                | `undefined`                                                                       | Add a client app version identifier that's included with API requests. **Production apps are strongly encouraged to set this value.**<br/>For example, you can use the following format: `appVersion: APP_NAME + '/' + APP_VERSION`.<br/>Setting this value provides telemetry that shows which apps are using the XMTP client SDK. This information can help XMTP developers provide app support, especially around communicating important SDK updates, including deprecations and required upgrades. |
| env       | `dev`   | Connect to the specified XMTP network environment. Valid values include `.dev`, `.production`, or `.local`. For important details about working with these environments, see [XMTP `production` and `dev` network environments](#xmtp-production-and-dev-network-environments). |

**Configure `env`**

```swift
// Configure the client to use the `production` network
let clientOptions = ClientOptions(api: .init(env: .production))
let client = try await Client.create(account: account, options: clientOptions)
```

</TabItem>
<TabItem value="dart" label="Dart"  attributes={{className: "dart_tab"}}>

You can configure the client environment when you call `Api.create()`.

By default, it will connect to a `local` XMTP network.

For important details about connecting to environments, see [XMTP `production` and `dev` network environments](#xmtp-production-and-dev-network-environments).

</TabItem>
<TabItem value="kotlin" label="Kotlin"  attributes={{className: "kotlin_tab"}}>

| Parameter  | Default     | Description                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| appVersion | `undefined` | Add a client app version identifier that's included with API requests. **Production apps are strongly encouraged to set this value.**<br/>For example, you can use the following format: `appVersion: APP_NAME + '/' + APP_VERSION`.<br/>Setting this value provides telemetry that shows which apps are using the XMTP client SDK. This information can help XMTP developers provide app support, especially around communicating important SDK updates, including deprecations and required upgrades. |
| env        | `DEV`       | Connect to the specified XMTP network environment. Valid values include `DEV`, `.PRODUCTION`, or `LOCAL`. For important details about working with these environments, see [XMTP `production` and `dev` network environments](#xmtp-production-and-dev-network-environments).                                                                                                                                                            |

**Configure `env`**

```kotlin
// Configure the client to use the `production` network
val options =
    ClientOptions(
        api = ClientOptions.Api(env = XMTPEnvironment.PRODUCTION, isSecure = true)
    )
val client = Client().create(account = account, options = options)
```

:::note

The `apiUrl`, `keyStoreType`, `codecs`, and `maxContentSize` parameters from the XMTP client SDK for JavaScript (xmtp-js) are not yet supported.

:::

</TabItem>
<TabItem value="react" label="React"  attributes={{className: "react_tab"}}>

| Parameter                 | Default                                                                           | Description                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| appVersion                | `undefined`                                                                       | Add a client app version identifier that's included with API requests. **Production apps are strongly encouraged to set this value.**<br/>For example, you can use the following format: `appVersion: APP_NAME + '/' + APP_VERSION`.<br/>Setting this value provides telemetry that shows which apps are using the XMTP client SDK. This information can help XMTP developers provide app support, especially around communicating important SDK updates, including deprecations and required upgrades. |
| env                       | `dev`                                                                             | Connect to the specified XMTP network environment. Valid values include `dev`, `production`, or `local`. For important details about working with these environments, see [XMTP `production` and `dev` network environments](#xmtp-production-and-dev-network-environments).                                                                                                                                                             |
| apiUrl                    | `undefined`                                                                       | Manually specify an API URL to use. If specified, value of `env` will be ignored.                                                                                                                                                                                                                                                                                                                                                        |
| keystoreProviders         | `[StaticKeystoreProvider, NetworkKeystoreProvider, KeyGeneratorKeystoreProvider]` | Override the default behavior of how the Client creates a Keystore with a custom provider. This can be used to get the user's private keys from a different storage mechanism.                                                                                                                                                                                                                                                           |
| persistConversations      | `true`                                                                            | Maintain a cache of previously seen V2 conversations in the storage provider (defaults to `LocalStorage`).                                                                                                                                                                                                                                                                                                                               |
| skipContactPublishing     | `false`                                                                           | Do not publish the user's contact bundle to the network on Client creation. Designed to be used in cases where the Client session is short-lived (for example, decrypting a push notification), and where it is known that a Client instance has been instantiated with this flag set to false at some point in the past.                                                                                                                |
| codecs                    | `[TextCodec]`                                                                     | Add codecs to support additional content types.                                                                                                                                                                                                                                                                                                                                                                                          |
| maxContentSize            | `100M`                                                                            | Maximum message content size in bytes.                                                                                                                                                                                                                                                                                                                                                                                                   |
| preCreateIdentityCallback | `undefined`                                                                       | `preCreateIdentityCallback` is a function that will be called immediately before a [Create Identity](/docs/concepts/account-signatures#sign-to-create-an-xmtp-identity) wallet signature is requested from the user.                                                                                                                                                                                                                     |
| preEnableIdentityCallback | `undefined`                                                                       | `preEnableIdentityCallback` is a function that will be called immediately before an [Enable Identity](/docs/concepts/account-signatures#sign-to-enable-an-xmtp-identity) wallet signature is requested from the user.                                                                                                                                                                                                                    |

</TabItem>
<TabItem value="rn" label="React Native"  attributes={{className: "rn_tab"}}>

| Parameter  | Default     | Description                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| appVersion | `undefined` | Add a client app version identifier that's included with API requests. **Production apps are strongly encouraged to set this value.**<br/>For example, you can use the following format: `appVersion: APP_NAME + '/' + APP_VERSION`.<br/>Setting this value provides telemetry that shows which apps are using the XMTP client SDK. This information can help XMTP developers provide app support, especially around communicating important SDK updates, including deprecations and required upgrades. |
| env        | `dev`       | Connect to the specified XMTP network environment. Valid values include `dev`, `production`, or `local`. For important details about working with these environments, see [XMTP `production` and `dev` network environments](#xmtp-production-and-dev-network-environments).                                                                                                                                                             |

</TabItem>
</Tabs>

## Environments

XMTP identity on `dev` network is completely distinct from its XMTP identity on the `production` network, as are the messages associated with these identities.

- `production`: This network is used in production and is configured to store messages indefinitely.

- `dev`: XMTP may occasionally delete messages and keys from this network

- `local`: Use to have a client communicate with an XMTP node you are running locally. For example, an XMTP node developer can set `env` to `local` to generate client traffic to test a node running locally.
