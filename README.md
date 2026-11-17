# Room

A private chat room for two people. It is one web page with no server of its own and no accounts.

**Open the room: https://skavper.github.io/room/**

## How to use it

1. Both people open the link above.
2. Each types their own name and the **same password**.
3. Tap **Go in**. When the other person is in, the header shows "… is here".

The password is the room. Anyone who types the same password lands in the same chat, and anyone who types a different one sees nothing. Agree on it in person or over a channel you trust, and make it long: four random words in a row is a good choice.

If the other person is in but you cannot see each other, one of the passwords is different. Tap **Leave** and type it again.

### Getting back in after the page hides

**Refresh the page.** The sign-in form comes back. You can also tap the site name at the top left of the cover page **three times**, quickly. Type your name and the password again; the saved chat is still there.

## What it does

- **End-to-end encrypted.** Messages are encrypted in the browser before they leave the phone. The relays that carry them only see random-looking bytes.
- **Saved on your device.** The chat is stored in the browser, locked with the password. It is still there when you come back and type the same password.
- **Works offline.** Messages written without a connection go out when it is back.
- **Messages wait.** You can write when the other person is not in the room. The message waits, encrypted, and arrives when they open the room, also when you have left by then. A photo arrives once you are both in the room.
- **Last here, and active.** Under the name you see when the other person was last in the room. While they are in, you see how long ago they last touched the page. The page knows nothing about the rest of their phone.
- **Ask for location.** Tap **Ask for location**. The other person gets a button **Share my location**. Only when they tap it does their phone read the place and send it, encrypted, with a link that opens a map. The page never sends a place by itself and does not follow anyone.
- **Sent, Delivered, Opened.** Your last message shows how far it got: Sent (it left your phone), Delivered (the other phone has it) or Opened (it is on their screen).
- **Chat lines, not bubbles.** Each message is a line with a coloured bar and the name of who wrote it, grouped under the day.
- **Catches up.** If one phone missed messages, the two phones compare what they hold and send only what is missing.
- **Photos.** Tap the picture button, or paste a picture into the message box. The photo is made smaller before it is sent, and what the camera wrote into the file (such as the place it was taken) is removed. Photos are saved with the chat. Tap one to see it large and to save it to your device.
- **Voice notes.** Tap the microphone button next to the picture button. The phone asks once to use the microphone. While it records, tap the cross to throw it away or the arrow to send it. A voice note is 2 minutes at most. Switching to another app or tab while recording throws the recording away. Like a photo, it arrives once you are both in the room, and it is saved with the chat.
- **Delete chat.** Removes the chat and its photos for both people. This cannot be undone.
- **Typing and presence.** The first letter of the other person's name pops up at the bottom while they are in the chat, with moving dots while they type.
- **Notifications on your phone (you choose).** Tap **Notifications** in the chat and follow the steps. Your phone then tells you "New message" when the other person writes and you are not in the room. It needs the free app "ntfy". It is off until you turn it on, and you can turn it off again at any time.
- **Hides when you look away.** Switch tab, switch app or click another window and the chat signs you out and turns into a harmless-looking page: a shop sale, a recipe or a word of the day, picked at random, with its own tab title and icon.
- **Settings for this phone only.** Tap the gear at the top of the chat. Both switches change only the phone you tap them on.
  - **Stay signed in** (off at first). When it is on, this phone opens the chat without the password. **Leave** still signs out.
  - **Cover page** (on at first). When it is off, no cover page shows and the chat stays open when you look away.
- **Verse of the day.** A Bible verse shows at the top of the chat, with a few words on what it means. It is open the first time each day, then a small bar. Tap it to open or fold it. For now it is Psalm 34:6-8 (King James Version).

## How it works

Everything is in [`index.html`](index.html). There is no build step and no dependency.

| Part | Detail |
| --- | --- |
| Key | The password is stretched with PBKDF2-SHA-256 (250,000 rounds) into 96 bytes: an AES-256-GCM key, the name of the meeting place, the name of the saved chat, and a key for notification codes. |
| Transport | Encrypted packets are published over MQTT on WebSocket to three public brokers (HiveMQ, EMQX, Mosquitto) at once, so the chat keeps working if one is down. |
| Packets | 12 random bytes (nonce) + AES-GCM ciphertext, padded to 1024, 2048, 4096… bytes so the length of a message is hidden. |
| Replay guard | A packet is dropped if it does not decrypt, if its nonce was seen before, or if its clock is more than 10 minutes off. The packets a relay keeps (the notification choices and the waiting messages) have no clock check; an old one cannot undo anything newer. |
| Photos | Redrawn in the browser as a JPEG (1280 px at most), then sent encrypted in pieces of 9,000 bytes. A phone that misses pieces asks for them again. |
| Voice notes | Recorded in the browser (MP4 or WebM sound, about 32 kbit/s), then sent encrypted in the same pieces as a photo. Played back only as sound. |
| Storage | IndexedDB, one encrypted record per message and per photo or voice note. No key or password is ever stored. |
| Waiting messages | Each person has one encrypted "box" that the relays keep: the messages nobody has received yet (30 at most, no photo bytes) and which messages that person holds. A page that opens the room alone reads the boxes. The time a box was sealed is the "last here" time. |
| Location | A request and an answer are ordinary encrypted messages. The place (latitude, longitude, accuracy) is read once, after a tap, and saved with the chat like any message. |
| Notifications | Only for people who turned them on. The writer's page sends [ntfy.sh](https://ntfy.sh) one small request: a code made from the password and the name, the fixed words "New message", and the address of the page. Never a name, never the text. The on/off choices travel encrypted; the relays keep one such packet so a phone learns them when it opens the room alone. |

The comment at the top of the script in `index.html` describes the packet types, the delete mechanism and the sync protocol in full.

## Safety

- **Nothing from outside.** A content security policy lets the page load no outside script, style or picture, and lets it talk only to the three relays and, for notifications, ntfy.sh. Nothing is sent to ntfy.sh unless someone in the room turned notifications on or asked for a test.
- **No framing.** The page refuses to run inside another website.
- **Short passwords need a second tap.** A password under 12 characters shows a warning first.
- **Hides when you look away,** and also after 10 minutes without a touch.
- **Photos are shown only as pictures.** Whatever arrives is treated as a JPEG image and nothing else.
- **Text is never treated as code.** Messages and names are shown as plain text.

## Limits

- **The password is the only protection.** The page and this code are public, so a short or guessable password can be guessed. Use a long one.
- **Made for two people.** Nothing stops a third person who knows the password from joining.
- **Both phones need the right time.** Set the clock to automatic. A clock more than 10 minutes off makes the other phone ignore your messages.
- **The relays are public test brokers.** They cannot read the chat, but they can see that someone is using the room, and they make no promise to stay up.
- **Waiting messages rest on the public relays.** They are encrypted, but a relay may drop them after a while or after a restart. Then the message arrives the next time you are both in the room.
- **"Last here" is about this page only.** It says when the other person had the room open, not when they used their phone.
- **A shared location is a message.** It stays in the chat until someone uses Delete chat. Opening the map leaves the page, which signs you out. The map service then sees the place you open.
- **Clearing browser data deletes the saved chat** on that device.
- **Photos are for the new page only.** A phone that still has the old page open shows the word "Photo" until it is reloaded.
- **Voice notes are for the new page only.** A phone that still has the old page open shows the words "Voice note" until it is reloaded. An older phone may not be able to play every kind of sound; the page then says so.
- **A saved or screenshotted photo is out of the room.** Delete chat cannot reach a copy the other person has saved.
- **Notifications use an outside service.** ntfy.sh cannot read the chat, but it can see that a message was written, and when. The notification code belongs to your name, so type the same name every time. Opening the ntfy app takes you away from the page, which signs you out: turn notifications on first.
- **Stay signed in keeps the keys on the phone.** They are stored as locked browser keys that cannot be read out as text, and the password is never stored. But anyone who holds the unlocked phone can open the chat. Turn it on only on a phone that is yours alone.
- **With the cover off the chat stays on the screen.** Anyone who picks up the phone sees it.
- **The cover is a disguise, not a lock.** The address bar still shows this site, and anyone who refreshes the page or knows the three taps gets the sign-in form. The password still protects the chat.

## Run it yourself

Open `index.html` over HTTPS or from `localhost`. The page needs the Web Crypto API, which browsers only allow in a secure context.

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000/.

To host your own copy, fork the repo and turn on GitHub Pages for the `main` branch.
