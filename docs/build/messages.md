---
sidebar_label: Messages
sidebar_position: 4
description: Learn how to send and list messages
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Send and list messages with XMTP

The message payload can be a plain string, but you can configure custom content types. To learn more, see [Content types](/docs/content-types/introduction).

## Send messages

To send a message, the recipient must have already started their client at least once and consequently advertised their key bundle on the network.

You might want to consider [optimistically sending messages](/docs/tutorials/optimistic-sending).

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript"  attributes={{className: "js_tab"}}>

```ts
const conversation = await xmtp.conversations.newConversation(
  "0x3F11b27F323b62B159D2642964fa27C46C841897",
);
await conversation.send("Hello world");
```

</TabItem>
<TabItem value="swift" label="Swift"  attributes={{className: "swift_tab"}}>

```swift
let conversation = try await client.conversations.newConversation(
  with: "0x3F11b27F323b62B159D2642964fa27C46C841897")
try await conversation.send(content: "Hello world")
```

</TabItem>
<TabItem value="dart" label="Dart"  attributes={{className: "dart_tab"}}>

```dart
var convo = await client.newConversation("0x...");
await client.sendMessage(convo, 'gm');
```

</TabItem>
<TabItem value="kotlin" label="Kotlin"  attributes={{className: "kotlin_tab"}}>

```kotlin
val conversation =
    client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

conversation.send(text = "Hello world")
```

</TabItem>
<TabItem value="react" label="React"  attributes={{className: "react_tab"}}>

```tsx
import { useSendMessage } from "@xmtp/react-sdk";

const sendMessage = useSendMessage(conversation);

await sendMessage(message);
```

</TabItem>
<TabItem value="rn" label="React Native"  attributes={{className: "rn_tab"}}>

```tsx
const conversation = await xmtp.conversations.newConversation(
  "0x3F11b27F323b62B159D2642964fa27C46C841897",
);
await conversation.send("Hello world");
```

</TabItem>
</Tabs>

## List messages in a conversation

You can receive the complete message history in a conversation.

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript"  attributes={{className: "js_tab"}}>

```ts
for (const conversation of await xmtp.conversations.list()) {
  // All parameters are optional and can be omitted
  const opts = {
    // Only show messages from last 24 hours
    startTime: new Date(new Date().setDate(new Date().getDate() - 1)),
    endTime: new Date(),
  };
  const messagesInConversation = await conversation.messages(opts);
}
```

</TabItem>
<TabItem value="swift" label="Swift"  attributes={{className: "swift_tab"}}>

```swift
for conversation in client.conversations.list() {
  let messagesInConversation = try await conversation.messages()
}
```

</TabItem>
<TabItem value="dart" label="Dart"  attributes={{className: "dart_tab"}}>

```dart
// Only show messages from the last 24 hours.
var messages = await alice.listMessages(convo,
    start: DateTime.now().subtract(const Duration(hours: 24)));
```

</TabItem>
<TabItem value="kotlin" label="Kotlin"  attributes={{className: "kotlin_tab"}}>

```kotlin
for (conversation in client.conversations.list()) {
    val messagesInConversation = conversation.messages()
}
```

</TabItem>
<TabItem value="react" label="React"  attributes={{className: "react_tab"}}>

```tsx
import { useMessages } from "@xmtp/react-sdk";

const [conversation, setConversation] = useState(null);
const { messages } = useMessages(conversation);

useEffect(() => {
  if (messages) {
    console.log("Loaded message history:", messages.length);
  }
}, [messages]);
```

</TabItem>
<TabItem value="rn" label="React Native"  attributes={{className: "rn_tab"}}>

```tsx
for (const conversation of await xmtp.conversations.list()) {
  const messagesInConversation = await conversation.messages(before: new Date(
    new Date().setDate(new Date().getDate() - 1)), after: new Date())
}
```

</TabItem>
</Tabs>

## List messages in a conversation with pagination

If a conversation has a lot of messages, it's more performant to retrieve and process the messages page by page instead of handling all of the messages at once.

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript"  attributes={{className: "js_tab"}}>

Call `conversation.messagesPaginated()`, which will return an [AsyncGenerator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncGenerator) yielding one page of results at a time. `conversation.messages()` uses this under the hood internally to gather all messages.

```ts
const conversation = await xmtp.conversations.newConversation(
  "0x3F11b27F323b62B159D2642964fa27C46C841897",
);

for await (const page of conversation.messagesPaginated({ pageSize: 25 })) {
  for (const msg of page) {
    // Breaking from the outer loop will stop the client from requesting any further pages
    if (msg.content === "gm") {
      return;
    }
    console.log(msg.content);
  }
}
```

</TabItem>
<TabItem value="swift" label="Swift"  attributes={{className: "swift_tab"}}>

Call `conversation.messages(limit: Int, before: Date)`, which will return the specified number of messages sent before that time.

```swift
let conversation = try await client.conversations.newConversation(
  with: "0x3F11b27F323b62B159D2642964fa27C46C841897")

let messages = try await conversation.messages(limit: 25)
let nextPage = try await conversation.messages(limit: 25, before: messages[0].sent)
```

</TabItem>
<TabItem value="dart" label="Dart"  attributes={{className: "dart_tab"}}>

Specify `limit` and `end`, which will return the specified number
of messages sent before that time.

```dart
var messages = await alice.listMessages(convo, limit: 10);
var nextPage = await alice.listMessages(
    convo, limit: 10, end: messages.last.sentAt);
```

</TabItem>
<TabItem value="kotlin" label="Kotlin"  attributes={{className: "kotlin_tab"}}>

Call `conversation.messages(limit: Int, before: Date)`, which will return the specified number of messages sent before that time.

```kotlin
val conversation =
    client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

val messages = conversation.messages(limit = 25)
val nextPage = conversation.messages(limit = 25, before = messages[0].sent)
```

</TabItem>
<TabItem value="react" label="React"  attributes={{className: "react_tab"}}>

```tsx
import { useMessages } from "@xmtp/react-sdk";

const { error, isLoading, messages, next } = useMessages(
  conversation,
  options: {
    limit: 20,
  },
);
const handleClick = useCallback(() => {
  // fetch next page of messages
  next();
}, [next]);


return (

    <button type="button" onClick={handleClick}>
      Load more messages
    </button>

);
```

</TabItem>
<TabItem value="rn" label="React Native"  attributes={{className: "rn_tab"}}>

```tsx
const conversation = await xmtp.conversations.newConversation(
  '0x3F11b27F323b62B159D2642964fa27C46C841897'
)

for await (const page of conversation.messages(limit: 25)) {
  for (const msg of page) {
    // Breaking from the outer loop will stop the client from requesting any further pages
    if (msg.content === 'gm') {
      return
    }
    console.log(msg.content)
  }
}
```

</TabItem>
</Tabs>

## Decode a single message

You can decode a single `Envelope` from XMTP using the `decode` method:

<Tabs groupId="sdk-langs">
<TabItem value="swift" label="Swift"  attributes={{className: "swift_tab"}}>

```swift
let conversation = try await client.conversations.newConversation(
  with: "0x3F11b27F323b62B159D2642964fa27C46C841897")

// Assume this function returns an Envelope that contains a message for the above conversation
let envelope = getEnvelopeFromXMTP()

let decodedMessage = try conversation.decode(envelope)
```

</TabItem>
<TabItem value="kotlin" label="Kotlin"  attributes={{className: "kotlin_tab"}}>

```kotlin
val conversation =
    client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

// Assume this function returns an Envelope that contains a message for the above conversation
val envelope = getEnvelopeFromXMTP()

val decodedMessage = conversation.decode(envelope)
```

</TabItem>
</Tabs>
