---
sidebar_label: Streams
sidebar_position: 5
description: Learn how to stream new conversations and messages
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Stream conversations and messages with XMTP

XMTP supports real-time message delivery and retrieval. Once you initially retrieve existing conversations, you can listen for a real-time stream of new conversations and messages.

## Listen for new conversations

You can listen for new conversations being started in real-time. This enables apps to display incoming messages from new contacts.

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript" default>

**Caution**: This stream will continue infinitely. To end the stream, you can either break from the loop, or call `await stream.return()`.

```ts
const stream = await xmtp.conversations.stream();
for await (const conversation of stream) {
  console.log(`New conversation started with ${conversation.peerAddress}`);
  // Say hello to your new friend
  await conversation.send("Hi there!");
  // Break from the loop to stop listening
  break;
}
```

</TabItem>
<TabItem value="swift" label="Swift" default>

**Caution**: This stream will continue infinitely. To end the stream, break from the loop.

```swift
for try await conversation in client.conversations.stream() {
  print("New conversation started with \(conversation.peerAddress)")

  // Say hello to your new friend
  try await conversation.send(content: "Hi there!")

  // Break from the loop to stop listening
  break
}
```

</TabItem>
<TabItem value="dart" label="Dart" default>

```dart
var listening = client.streamConversations().listen((convo) {
  debugPrint('Got a new conversation with ${convo.peer}');
});
// When you want to stop listening:
await listening.cancel();
```

</TabItem>
<TabItem value="kotlin" label="Kotlin - beta" default>

```kotlin
client.conversations.stream().collect {
    print("New conversation started with ${it.peerAddress}")
    // Say hello to your new friend
    it.send(text = "Hi there!")
}
```

</TabItem>
<TabItem value="react" label="React - beta" default>

The `useStreamConversations` hook listens for new conversations in real-time and calls the passed callback when a new conversation is created. It also exposes an error state.

**Type**

```ts
import type { Conversation } from "@xmtp/react-sdk";

const useStreamConversations: (
  onConversation: (conversation: Conversation) => void
) => {
  error: unknown;
};
```

**Example**

```tsx
import { useCallback, useState } from "react";
import { useStreamConversations } from "@xmtp/react-sdk";
import type { Conversation } from "@xmtp/react-sdk";

export const NewConversations: React.FC = () => {
  // track streamed conversations
  const [streamedConversations, setStreamedConversations] = useState<
    Conversation[]
  >([]);

  // callback to handle incoming conversations
  const onConversation = useCallback(
    (conversation: Conversation) => {
      setStreamedConversations((prev) => [...prev, conversation]);
    },
    [],
  );
  const { error } = useStreamConversations(onConversation);

  if (error) {
    return "An error occurred while streaming conversations";
  }

  return (
    ...
  );
};
```

</TabItem>
</Tabs>

## Listen for new messages in a conversation

Listen for any new incoming or outgoing messages in a conversation.

<Tabs groupId="sdk-langs">
<TabItem value="js" label="JavaScript" default>

You can listen for any new messages (incoming or outgoing) in a conversation by calling `conversation.streamMessages()`.

A successfully received message (that makes it through the decoding and decryption without throwing) can be trusted to be authentic, i.e. that it was sent by the owner of the `message.senderAddress` wallet and that it wasn't modified in transit. The `message.sent` timestamp can be trusted to have been set by the sender.

The Stream returned by the `stream` methods is an asynchronous iterator and as such usable by a for-await-of loop. Note however that it is by its nature infinite, so any looping construct used with it will not terminate, unless the termination is explicitly initiated (by breaking the loop or by an external call to `Stream.return()`).

```ts
const conversation = await xmtp.conversations.newConversation(
  "0x3F11b27F323b62B159D2642964fa27C46C841897"
);
for await (const message of await conversation.streamMessages()) {
  if (message.senderAddress === xmtp.address) {
    // This message was sent from me
    continue;
  }
  console.log(`New message from ${message.senderAddress}: ${message.content}`);
}
```

</TabItem>
<TabItem value="swift" label="Swift" default>

You can listen for any new messages (incoming or outgoing) in a conversation by calling `conversation.streamMessages()`.

A successfully received message (that makes it through the decoding and decryption without throwing) can be trusted to be authentic. Authentic means that it was sent by the owner of the `message.senderAddress` account and that it wasn't modified in transit. The `message.sent` timestamp can be trusted to have been set by the sender.

The stream returned by the `stream` methods is an asynchronous iterator and as such is usable by a for-await-of loop. Note however that it is by its nature infinite, so any looping construct used with it will not terminate, unless the termination is explicitly initiated (by breaking the loop).

```swift
let conversation = try await client.conversations.newConversation(
  with: "0x3F11b27F323b62B159D2642964fa27C46C841897")

for try await message in conversation.streamMessages() {
  if message.senderAddress == client.address {
    // This message was sent from me
    continue
  }

  print("New message from \(message.senderAddress): \(message.body)")
}
```

</TabItem>
<TabItem value="dart" label="Dart" default>

You can listen for any new messages (incoming or outgoing) in a conversation by calling
`client.streamMessages(convo)`.

A successfully received message (that makes it through the decoding and decryption) can be trusted
to be authentic. Authentic means that it was sent by the owner of the `message.sender` account and
that it wasn't modified in transit. The `message.sentAt` time can be trusted to have been set by
the sender.

```dart
var listening = client.streamMessages(convo).listen((message) {
  debugPrint('${message.sender}> ${message.content}');
});
// When you want to stop listening:
await listening.cancel();
```

</TabItem>
<TabItem value="kotlin" label="Kotlin - beta" default>

You can listen for any new messages (incoming or outgoing) in a conversation by calling `conversation.streamMessages()`.

A successfully received message (that makes it through the decoding and decryption without throwing) can be trusted to be authentic. Authentic means that it was sent by the owner of the `message.senderAddress` account and that it wasn't modified in transit. The `message.sent` timestamp can be trusted to have been set by the sender.

The flow returned by the `stream` methods is an asynchronous data stream that sequentially emits values and completes normally or with an exception.

```kotlin
val conversation =
    client.conversations.newConversation("0x3F11b27F323b62B159D2642964fa27C46C841897")

conversation.streamMessages().collect {
    if (it.senderAddress == client.address) {
        // This message was sent from me
    }

    print("New message from ${it.senderAddress}: ${it.body}")
}
```

</TabItem>
<TabItem value="react" label="React - beta" default>

The `useStreamMessages` hook streams new conversation messages on mount and exposes an error state.

**Type**

```ts
import type { Conversation, DecodedMessage } from "@xmtp/react-sdk";

const useStreamMessages: (
  conversation: Conversation,
  onMessage: (message: DecodedMessage) => void
) => {
  error: unknown;
};
```

**Example**

```tsx
import { useStreamMessages } from "@xmtp/react-sdk";
import type { Conversation, DecodedMessage } from "@xmtp/react-sdk";
import { useCallback, useEffect, useState } from "react";

export const StreamMessages: React.FC<{
  conversation: Conversation;
}> = ({
  conversation,
}) => {
  // track streamed messages
  const [streamedMessages, setStreamedMessages] = useState<DecodedMessage[]>(
    [],
  );

  // callback to handle incoming messages
  const onMessage = useCallback(
    (message: DecodedMessage) => {
      setStreamedMessages((prev) => [...prev, message]);
    },
    [streamedMessages],
  );

  useStreamMessages(conversation, onMessage);

  useEffect(() => {
    setStreamedMessages([]);
  }, [conversation]);

  return (
    ...
  );
};
```

</TabItem>
</Tabs>

## Listen for new messages in all conversations

:::info

There is a chance this stream can miss messages if multiple new conversations are received in the time it takes to update the stream to include a new conversation.

:::

<Tabs>
<TabItem value="js" label="JavaScript" default>

To listen for any new messages from all conversations, use `conversations.streamAllMessages()`.

```ts
for await (const message of await xmtp.conversations.streamAllMessages()) {
  if (message.senderAddress === xmtp.address) {
    // This message was sent from me
    continue;
  }
  console.log(`New message from ${message.senderAddress}: ${message.content}`);
}
```

</TabItem>
<TabItem value="react" label="React - beta" default>

The `useStreamAllMessages` hook streams new messages from all conversations on mount and exposes an error state.

**Type**

```ts
import type { DecodedMessage } from "@xmtp/react-sdk";

const useStreamAllMessages: (onMessage: (message: DecodedMessage) => void) => {
  error: unknown;
};
```

**Example**

```tsx
import { useStreamAllMessages } from "@xmtp/react-sdk";
import type { DecodedMessage } from "@xmtp/react-sdk";
import { useCallback, useState } from "react";

export const StreamAllMessages: React.FC = () => {
  // track streamed messages
  const [streamedMessages, setStreamedMessages] = useState<DecodedMessage[]>(
    [],
  );

  // callback to handle incoming messages
  const onMessage = useCallback(
    (message: DecodedMessage) => {
      setStreamedMessages((prev) => [...prev, message]);
    },
    [streamedMessages],
  );

  useStreamAllMessages(onMessage);

  return (
    ...
  );
};
```

</TabItem>
</Tabs>