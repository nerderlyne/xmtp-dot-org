---
sidebar_label: Kotlin
sidebar_position: 4
toc_max_heading_level: 4
description: "xmtp-android provides a Kotlin implementation of an XMTP message API client for use with Android apps."
---

# Quickstart for the Kotlin XMTP client SDK

The [Kotlin XMTP client SDK](https://github.com/xmtp/xmtp-android) (`xmtp-android`) provides a Kotlin implementation of an XMTP message API client for use with Android apps.

Build with this SDK to provide messaging between blockchain wallet addresses, including DMs, notifications, announcements, and more.

:::caution Important

This SDK is in **beta** status and ready for you to start building. However, we do **not** recommend using beta software in production apps. Software in this status may change based on feedback.

:::

Specifically, while push notifications should work with the current SDK, we are working on providing push notifications in the example app. We are also working on providing performance optimizations in the example app. These updates to the example app may inform changes to the SDK.

Follow along in the [tracking issue](https://github.com/xmtp/xmtp-android/issues/1) for updates.

To learn more about XMTP and get answers to frequently asked questions, see [FAQ about XMTP](/docs/concepts/faq).

## Example app

For a basic demonstration of the core concepts and capabilities of the `xmtp-android` client SDK, see the [Example app project](https://github.com/xmtp/xmtp-android/tree/main/example). This is currently a work in progress.

## Install from Maven Central

You can find the latest package version on [Maven Central](https://central.sonatype.com/artifact/org.xmtp/android).

```gradle
    implementation 'org.xmtp:android:X.X.X'
```

## Usage overview

The XMTP message API revolves around a message API client (client) that allows retrieving and sending messages to other XMTP network participants. A client must connect to a wallet app on startup. If this is the very first time the client is created, the client will generate a key bundle that is used to encrypt and authenticate messages. The key bundle persists encrypted in the network using an account signature. The public side of the key bundle is also regularly advertised on the network to allow parties to establish shared encryption keys. All of this happens transparently, without requiring any additional code.

```kotlin
// You'll want to replace this with a wallet from your application.
val account = PrivateKeyBuilder()

// Create the client with your wallet. This will connect to the XMTP `dev` network by default.
// The account is anything that conforms to the `XMTP.SigningKey` protocol.
val client = Client().create(account = account)

// Start a conversation with XMTP
val conversation = client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

// Load all messages in the conversation
val messages = conversation.messages()
// Send a message
conversation.send(text = "gm")
// Listen for new messages in the conversation
conversation.streamMessages().collect {
    print("${message.senderAddress}: ${message.body}")
}
```

## Handle conversations

Most of the time, when interacting with the network, you'll want to do it through `conversations`. Conversations are between two accounts.

```kotlin
// Create the client with a wallet from your app
val client = Client().create(account = account)
val conversations = client.conversations.list()
```

### List existing conversations

You can get a list of all conversations that have one or more messages.

```kotlin
val allConversations = client.conversations.list()

for (conversation in allConversations) {
    print("Saying GM to ${conversation.peerAddress}")
    conversation.send(text = "gm")
}
```

These conversations include all conversations for a user **regardless of which app created the conversation.** This functionality provides the concept of an [interoperable inbox](/docs/concepts/interoperable-inbox), which enables a user to access all of their conversations in any app built with XMTP.

### Listen for new conversations

You can also listen for new conversations being started in real-time. This will allow apps to display incoming messages from new contacts.

```kotlin
client.conversations.stream().collect {
    print("New conversation started with ${it.peerAddress}")
    // Say hello to your new friend
    it.send(text = "Hi there!")
}
```

### Start a new conversation

You can create a new conversation with any Ethereum address on the XMTP network.

```kotlin
val newConversation = client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")
```

### Send messages

To be able to send a message, the recipient must have already created a client at least once and consequently advertised their key bundle on the network. Messages are addressed using account addresses. In this example, the message payload is a plain string:

```kotlin
val conversation = client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")
conversation.send(text = "Hello world")
```

To learn about handling other content types, see [Handle different types of content](#handle-different-types-of-content).

### List messages in a conversation

You can receive the complete message history in a conversation by calling `conversation.messages()`.

```kotlin
for (conversation in client.conversations.list()) {
    val messagesInConversation = conversation.messages()
}
```

### List messages in a conversation with pagination

It may be helpful to retrieve and process the messages in a conversation page by page. You can do this by calling `conversation.messages(limit: Int, before: Date)` which will return the specified number of messages sent before that time.

```kotlin
val conversation = client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

val messages = conversation.messages(limit = 25)
val nextPage = conversation.messages(limit = 25, before = messages[0].sent)
```

### Listen for new messages in a conversation

You can listen for any new messages (incoming or outgoing) in a conversation by calling `conversation.streamMessages()`.

A successfully received message (that makes it through the decoding and decryption without throwing) can be trusted to be authentic. Authentic means that it was sent by the owner of the `message.senderAddress` account and that it wasn't modified in transit. The `message.sent` timestamp can be trusted to have been set by the sender.

The flow returned by the `stream` methods is an asynchronous data stream that sequentially emits values and completes normally or with an exception.

```kotlin
val conversation = client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

conversation.streamMessages().collect {
    if (it.senderAddress == client.address) {
        // This message was sent from me
    }

    print("New message from ${it.senderAddress}: ${it.body}")
}
```

### Decode a single message

You can decode a single `Envelope` from XMTP using the `decode` method:

```kotlin
val conversation = client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

// Assume this function returns an Envelope that contains a message for the above conversation
val envelope = getEnvelopeFromXMTP()

val decodedMessage = conversation.decode(envelope)
```

### Serialize/Deserialize conversations

You can save a conversation object locally using its `encodedContainer` property. This returns a `ConversationContainer` object which conforms to `Codable`.

```kotlin
// Get a conversation
val conversation = client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

// Dump it to JSON
val gson = GsonBuilder().create()
val data = gson.toJson(conversation)

// Get it back from JSON
val containerAgain = gson.fromJson(data.toString(StandardCharsets.UTF_8), ConversationV2Export::class.java)

// Get an actual Conversation object like we had above
val decodedConversation = containerAgain.decode(client)
decodedConversation.send(text = "hi")
```

### Handle different types of content

All the send functions support `SendOptions` as an optional parameter. The `contentType` option allows specifying different types of content than the default simple string, which is identified with content type identifier `ContentTypeText`.

To learn more about content types, see [Content types with XMTP](/docs/concepts/content-types).

Support for other types of content can be added by registering additional `ContentCodecs` with the `Client`. Every codec is associated with a content type identifier, `ContentTypeId`, which is used to signal to the client which codec should be used to process the content that is being sent or received.

For example, to learn about the codecs available in `xmtp-android`, see [Use content types in your app](https://github.com/xmtp/xmtp-android/blob/main/library/src/main/java/org/xmtp/android/library/codecs/README.md).

If there is a concern that the recipient may not be able to handle a non-standard content type, the sender can use the `contentFallback` option to provide a string that describes the content being sent. If the recipient fails to decode the original content, the fallback will replace it and can be used to inform the recipient what the original content was.

```kotlin
// Assuming we've loaded a fictional NumberCodec that can be used to encode numbers,
// and is identified with ContentTypeNumber, we can use it as follows.
Client.register(codec = NumberCodec())

val options = ClientOptions(api = ClientOptions.Api(contentType = ContentTypeNumber, contentFallback = "sending you a pie"))
aliceConversation.send(content = 3.14, options = options)
```

Custom codecs and content types may be proposed as interoperable standards through XRCs. To learn about the custom content type proposal process, see [XIP-5](https://github.com/xmtp/XIPs/blob/main/XIPs/xip-5-message-content-types.md).

### Compression

<!--provide kotlin details and code sample. showing swift for context of the kind of info you might want to provide. =)-->

Message content can be optionally compressed using the compression option. The value of the option is the name of the compression algorithm to use. Currently supported are gzip and deflate. Compression is applied to the bytes produced by the content codec.

Content will be decompressed transparently on the receiving end. Note that `Client` enforces maximum content size. The default limit can be overridden through the `ClientOptions`. Consequently, a message that would expand beyond that limit on the receiving end will fail to decode.

```kotlin
conversation.send(text = '#'.repeat(1000), options = ClientOptions.Api(compression = EncodedContentCompression.GZIP))
```

### Cache conversations

As a performance optimization, you may want to persist the list of conversations in your application outside of the SDK to speed up the first call to `client.conversations.list()`.

The exported conversation list contains encryption keys for any V2 conversations included in the list. As such, you should treat it with the same care that you treat private keys.

You can get a JSON serializable list of conversations by calling:

```kotlin
val client = Client().create(wallet)
val conversations = client.conversations.export()
saveConversationsSomewhere(JSON.stringify(conversations))
// To load the conversations in a new SDK instance you can run:

val client = Client.create(wallet)
val conversations = JSON.parse(loadConversationsFromSomewhere())
val client.importConversation(conversations)
```

## Breaking revisions

Because `xmtp-android` is in active development, you should expect breaking revisions that might require you to adopt the latest SDK release to enable your app to continue working as expected.

XMTP communicates about breaking revisions in the [XMTP Discord community](https://discord.gg/xmtp), providing as much advance notice as possible. Additionally, breaking revisions in an `xmtp-android` release are described on the [Releases page](https://github.com/xmtp/xmtp-android/releases).

## Deprecation

Older versions of the SDK will eventually be deprecated, which means:

1. The network will not support and eventually actively reject connections from clients using deprecated versions.
2. Bugs will not be fixed in deprecated versions.

The following table provides the deprecation schedule.

| Announced                                                            | Effective | Minimum Version | Rationale |
| -------------------------------------------------------------------- | --------- | --------------- | --------- |
| There are no deprecations scheduled for `xmtp-android` at this time. |           |                 |           |

Bug reports, feature requests, and PRs are welcome in accordance with these [contribution guidelines](https://github.com/xmtp/xmtp-android/blob/main/CONTRIBUTING.md).