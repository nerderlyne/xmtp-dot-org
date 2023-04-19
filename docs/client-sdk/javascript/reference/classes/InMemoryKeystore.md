<!---->
# Class: InMemoryKeystore

A Keystore is responsible for holding the user's XMTP private keys and using them to encrypt/decrypt/sign messages.
Keystores are instantiated using a `KeystoreProvider`

## Implements

- [`Keystore`](../interfaces/Keystore.md)

## Constructors

### constructor

**new InMemoryKeystore**(`keys`, `inviteStore`)

#### Parameters

| Name | Type |
| :------ | :------ |
| `keys` | `PrivateKeyBundleV1` |
| `inviteStore` | `default` |

#### Defined in

[keystore/InMemoryKeystore.ts:41](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L41)

## Properties

### accountAddress

 `Private` **accountAddress**: `undefined` \| `string`

#### Defined in

[keystore/InMemoryKeystore.ts:39](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L39)

___

### authenticator

 `Private` **authenticator**: `default`

#### Defined in

[keystore/InMemoryKeystore.ts:38](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L38)

___

### inviteStore

 `Private` **inviteStore**: `default`

#### Defined in

[keystore/InMemoryKeystore.ts:37](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L37)

___

### v1Keys

 `Private` **v1Keys**: `PrivateKeyBundleV1`

#### Defined in

[keystore/InMemoryKeystore.ts:35](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L35)

___

### v2Keys

 `Private` **v2Keys**: `PrivateKeyBundleV2`

#### Defined in

[keystore/InMemoryKeystore.ts:36](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L36)

## Methods

### createAuthToken

**createAuthToken**(`«destructured»`): `Promise`<`Token`\>

Create an XMTP auth token to be used as a header on XMTP API requests

#### Parameters

| Name | Type |
| :------ | :------ |
| `«destructured»` | `CreateAuthTokenRequest` |

#### Returns

`Promise`<`Token`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[createAuthToken](../interfaces/Keystore.md#createauthtoken)

#### Defined in

[keystore/InMemoryKeystore.ts:153](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L153)

___

### createInvite

**createInvite**(`req`): `Promise`<`CreateInviteResponse`\>

Create a sealed/encrypted invite and store the Topic keys in the Keystore for later use.
The returned invite payload must be sent to the network for the other party to be able to communicate.

#### Parameters

| Name | Type |
| :------ | :------ |
| `req` | `CreateInviteRequest` |

#### Returns

`Promise`<`CreateInviteResponse`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[createInvite](../interfaces/Keystore.md#createinvite)

#### Defined in

[keystore/InMemoryKeystore.ts:265](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L265)

___

### decryptV1

**decryptV1**(`req`): `Promise`<`DecryptResponse`\>

Decrypt a batch of V1 messages

#### Parameters

| Name | Type |
| :------ | :------ |
| `req` | `DecryptV1Request` |

#### Returns

`Promise`<`DecryptResponse`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[decryptV1](../interfaces/Keystore.md#decryptv1)

#### Defined in

[keystore/InMemoryKeystore.ts:52](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L52)

___

### decryptV2

**decryptV2**(`req`): `Promise`<`DecryptResponse`\>

Decrypt a batch of V2 messages

#### Parameters

| Name | Type |
| :------ | :------ |
| `req` | `DecryptV2Request` |

#### Returns

`Promise`<`DecryptResponse`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[decryptV2](../interfaces/Keystore.md#decryptv2)

#### Defined in

[keystore/InMemoryKeystore.ts:83](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L83)

___

### encryptV1

**encryptV1**(`req`): `Promise`<`EncryptResponse`\>

Encrypt a batch of V1 messages

#### Parameters

| Name | Type |
| :------ | :------ |
| `req` | `EncryptV1Request` |

#### Returns

`Promise`<`EncryptResponse`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[encryptV1](../interfaces/Keystore.md#encryptv1)

#### Defined in

[keystore/InMemoryKeystore.ts:121](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L121)

___

### encryptV2

**encryptV2**(`req`): `Promise`<`EncryptResponse`\>

Encrypt a batch of V2 messages

#### Parameters

| Name | Type |
| :------ | :------ |
| `req` | `EncryptV2Request` |

#### Returns

`Promise`<`EncryptResponse`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[encryptV2](../interfaces/Keystore.md#encryptv2)

#### Defined in

[keystore/InMemoryKeystore.ts:161](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L161)

___

### getAccountAddress

**getAccountAddress**(): `Promise`<`string`\>

Get the account address of the wallet used to create the Keystore

#### Returns

`Promise`<`string`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[getAccountAddress](../interfaces/Keystore.md#getaccountaddress)

#### Defined in

[keystore/InMemoryKeystore.ts:356](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L356)

___

### getPrivateKeyBundle

**getPrivateKeyBundle**(): `Promise`<`PrivateKeyBundleV1`\>

Export the private keys. May throw an error if the keystore implementation does not allow this operation

#### Returns

`Promise`<`PrivateKeyBundleV1`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[getPrivateKeyBundle](../interfaces/Keystore.md#getprivatekeybundle)

#### Defined in

[keystore/InMemoryKeystore.ts:352](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L352)

___

### getPublicKeyBundle

**getPublicKeyBundle**(): `Promise`<[`PublicKeyBundle`](PublicKeyBundle.md)\>

Get the `PublicKeyBundle` associated with the Keystore's private keys

#### Returns

`Promise`<[`PublicKeyBundle`](PublicKeyBundle.md)\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[getPublicKeyBundle](../interfaces/Keystore.md#getpublickeybundle)

#### Defined in

[keystore/InMemoryKeystore.ts:348](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L348)

___

### getV2Conversations

**getV2Conversations**(): `Promise`<`ConversationReference`[]\>

Get a list of V2 conversations

#### Returns

`Promise`<`ConversationReference`[]\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[getV2Conversations](../interfaces/Keystore.md#getv2conversations)

#### Defined in

[keystore/InMemoryKeystore.ts:335](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L335)

___

### lookupTopic

**lookupTopic**(`topic`): `undefined` \| `WithoutUndefined`<`TopicMap_TopicData`\>

#### Parameters

| Name | Type |
| :------ | :------ |
| `topic` | `string` |

#### Returns

`undefined` \| `WithoutUndefined`<`TopicMap_TopicData`\>

#### Defined in

[keystore/InMemoryKeystore.ts:367](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L367)

___

### saveInvites

**saveInvites**(`req`): `Promise`<`SaveInvitesResponse`\>

Take a batch of invite messages and store the `TopicKeys` for later use in decrypting messages

#### Parameters

| Name | Type |
| :------ | :------ |
| `req` | `SaveInvitesRequest` |

#### Returns

`Promise`<`SaveInvitesResponse`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[saveInvites](../interfaces/Keystore.md#saveinvites)

#### Defined in

[keystore/InMemoryKeystore.ts:200](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L200)

___

### signDigest

**signDigest**(`req`): `Promise`<`Signature`\>

Sign the provided digest with either the `IdentityKey` or a specified `PreKey`

#### Parameters

| Name | Type |
| :------ | :------ |
| `req` | `SignDigestRequest` |

#### Returns

`Promise`<`Signature`\>

#### Implementation of

[Keystore](../interfaces/Keystore.md).[signDigest](../interfaces/Keystore.md#signdigest)

#### Defined in

[keystore/InMemoryKeystore.ts:300](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L300)

___

### create

`Static` **create**(`keys`, `persistence?`): `Promise`<[`InMemoryKeystore`](InMemoryKeystore.md)\>

#### Parameters

| Name | Type |
| :------ | :------ |
| `keys` | `PrivateKeyBundleV1` |
| `persistence?` | [`Persistence`](../interfaces/Persistence.md) |

#### Returns

`Promise`<[`InMemoryKeystore`](InMemoryKeystore.md)\>

#### Defined in

[keystore/InMemoryKeystore.ts:48](https://github.com/xmtp/xmtp-js/blob/ff53c33/src/keystore/InMemoryKeystore.ts#L48)