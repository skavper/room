# Room

A private chat room for two people. It is one web page with no server of its own and no accounts.

**Open the room: https://skavper.github.io/room/**

## How to use it

1. Both people open the link above.
2. Each types their own name and the **same password**.
3. Tap **Go in**. When the other person is in, the header shows "… is here".

The password is the room. Anyone who types the same password lands in the same chat, and anyone who types a different one sees nothing. Agree on it in person or over a channel you trust, and make it long: four random words in a row is a good choice.

If the other person is in but you cannot see each other, one of the passwords is different. Tap **Leave** and type it again.

## What it does

- **End-to-end encrypted.** Messages are encrypted in the browser before they leave the phone. The relays that carry them only see random-looking bytes.
- **Saved on your device.** The chat is stored in the browser, locked with the password. It is still there when you come back and type the same password.
- **Works offline.** Messages written without a connection go out when it is back. Each message shows Sending, Sent, or Delivered.
- **Catches up.** If one phone missed messages, the two phones compare what they hold and send only what is missing.
- **Delete chat.** Removes the chat for both people. This cannot be undone.
- **Typing and presence.** You can see when the other person is here and when they are typing.

## How it works

Everything is in [`index.html`](index.html). There is no build step and no dependency.

| Part | Detail |
| --- | --- |
| Key | The password is stretched with PBKDF2-SHA-256 (250,000 rounds) into 64 bytes: an AES-256-GCM key, the name of the meeting place, and the name of the saved chat. |
| Transport | Encrypted packets are published over MQTT on WebSocket to three public brokers (HiveMQ, EMQX, Mosquitto) at once, so the chat keeps working if one is down. |
| Packets | 12 random bytes (nonce) + AES-GCM ciphertext, padded to 1024, 2048, 4096… bytes so the length of a message is hidden. |
| Replay guard | A packet is dropped if it does not decrypt, if its nonce was seen before, or if its clock is more than 10 minutes off. |
| Storage | IndexedDB, one encrypted record per message. No key or password is ever stored. |

The comment at the top of the script in `index.html` describes the packet types, the delete mechanism and the sync protocol in full.

## Limits

- **The password is the only protection.** The page and this code are public, so a short or guessable password can be guessed. Use a long one.
- **Made for two people.** Nothing stops a third person who knows the password from joining.
- **Both phones need the right time.** Set the clock to automatic. A clock more than 10 minutes off makes the other phone ignore your messages.
- **The relays are public test brokers.** They cannot read the chat, but they can see that someone is using the room, and they make no promise to stay up.
- **Clearing browser data deletes the saved chat** on that device.

## Run it yourself

Open `index.html` over HTTPS or from `localhost`. The page needs the Web Crypto API, which browsers only allow in a secure context.

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000/.

To host your own copy, fork the repo and turn on GitHub Pages for the `main` branch.
