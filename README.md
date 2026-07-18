# Baileys — WhatsApp Web API Library

A lightweight, full-featured WhatsApp Web API library for Node.js. Connect to WhatsApp via WebSocket, send and receive messages, manage groups, newsletters, and more — no browser required.

---

## Requirements

- Node.js **20+**
- npm

---

## Installation

```bash
npm install
```

---

## Quick Start

```js
const { makeWASocket, useMultiFileAuthState, DisconnectReason } = require('./lib/index.js');
const { Boom } = require('@hapi/boom');

async function start() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info');

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: false
    });

    sock.ev.on('creds.update', saveCreds);

    sock.ev.on('connection.update', ({ connection, lastDisconnect, qr }) => {
        if (qr) {
            console.log('QR Code:', qr); // scan with WhatsApp
        }
        if (connection === 'close') {
            const shouldReconnect =
                new Boom(lastDisconnect?.error)?.output?.statusCode !== DisconnectReason.loggedOut;
            if (shouldReconnect) start();
        } else if (connection === 'open') {
            console.log('Connected!');
        }
    });

    sock.ev.on('messages.upsert', ({ messages }) => {
        const msg = messages[0];
        if (!msg.key.fromMe) {
            console.log('New message:', msg.message);
        }
    });
}

start();
```

---

## Pair Code Login (no QR)

```js
const sock = makeWASocket({ auth: state });

// Request a pairing code for your phone number (include country code, no +)
const code = await sock.requestPairingCode('94XXXXXXXXX');
console.log('Pairing Code:', code); // Enter this in WhatsApp → Linked Devices
```

---

## Sending Messages

### Text
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', { text: 'Hello!' });
```

### Image
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    image: { url: './image.jpg' },
    caption: 'Check this out!'
});
```

### Video
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    video: { url: './video.mp4' },
    caption: 'Watch this'
});
```

### Audio / PTT (Voice Note)
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    audio: { url: './audio.mp3' },
    ptt: true // set false for regular audio
});
```

### Document
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    document: { url: './file.pdf' },
    mimetype: 'application/pdf',
    fileName: 'document.pdf'
});
```

### Reply to a Message
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', { text: 'Reply!' }, {
    quoted: msg
});
```

### React to a Message
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    react: { text: '🔥', key: msg.key }
});
```

### Poll
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    poll: {
        name: 'Your favourite?',
        values: ['Option A', 'Option B', 'Option C'],
        selectableCount: 1
    }
});
```

---

## Interactive Buttons

### Interactive Message (Native Flow)
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    interactiveMessage: {
        body: { text: 'Choose an option:' },
        footer: { text: 'Powered by Baileys' },
        nativeFlowMessage: {
            buttons: [
                {
                    name: 'quick_reply',
                    buttonParamsJson: JSON.stringify({ display_text: 'Option 1', id: 'opt1' })
                },
                {
                    name: 'quick_reply',
                    buttonParamsJson: JSON.stringify({ display_text: 'Option 2', id: 'opt2' })
                }
            ]
        }
    }
});
```

### List Message
```js
await sock.sendMessage('94XXXXXXXXX@s.whatsapp.net', {
    listMessage: {
        title: 'Choose',
        description: 'Pick one from the list',
        buttonText: 'Open List',
        listType: 1,
        sections: [
            {
                title: 'Section 1',
                rows: [
                    { title: 'Row 1', rowId: 'row1', description: 'Description' },
                    { title: 'Row 2', rowId: 'row2', description: 'Description' }
                ]
            }
        ]
    }
});
```

---

## Groups

```js
// Get group metadata
const meta = await sock.groupMetadata('GROUP_JID@g.us');
console.log(meta.subject, meta.participants);

// Send to group
await sock.sendMessage('GROUP_JID@g.us', { text: 'Hello group!' });

// Create group
const group = await sock.groupCreate('Group Name', ['94XXXXXXXXX@s.whatsapp.net']);

// Add / Remove participants
await sock.groupParticipantsUpdate('GROUP_JID@g.us', ['94XXXXXXXXX@s.whatsapp.net'], 'add');
await sock.groupParticipantsUpdate('GROUP_JID@g.us', ['94XXXXXXXXX@s.whatsapp.net'], 'remove');

// Promote / Demote to admin
await sock.groupParticipantsUpdate('GROUP_JID@g.us', ['94XXXXXXXXX@s.whatsapp.net'], 'promote');
await sock.groupParticipantsUpdate('GROUP_JID@g.us', ['94XXXXXXXXX@s.whatsapp.net'], 'demote');

// Get invite link
const link = await sock.groupInviteCode('GROUP_JID@g.us');
```

---

## Newsletters

```js
// Follow a newsletter
await sock.newsletterFollow('NEWSLETTER_JID@newsletter');

// Unfollow
await sock.newsletterUnfollow('NEWSLETTER_JID@newsletter');

// Get metadata
const info = await sock.newsletterMetadata('invite', 'INVITE_CODE');
```

---

## Events

```js
sock.ev.on('messages.upsert',     ({ messages, type }) => { /* new messages */ });
sock.ev.on('messages.update',     updates => { /* message status updates */ });
sock.ev.on('message-receipt.update', updates => { /* read receipts */ });
sock.ev.on('presence.update',     ({ id, presences }) => { /* typing / online */ });
sock.ev.on('chats.update',        updates => { /* chat metadata changes */ });
sock.ev.on('contacts.update',     updates => { /* contact changes */ });
sock.ev.on('groups.update',       updates => { /* group metadata changes */ });
sock.ev.on('group-participants.update', ({ id, participants, action }) => { /* join/leave */ });
sock.ev.on('connection.update',   update => { /* connection state */ });
sock.ev.on('creds.update',        saveCreds); // always save creds!
```

---

## Download Media

```js
const { downloadMediaMessage } = require('./lib/index.js');

sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0];
    if (msg.message?.imageMessage) {
        const buffer = await downloadMediaMessage(msg, 'buffer', {});
        require('fs').writeFileSync('received.jpg', buffer);
    }
});
```

---

## Auth State

Auth credentials are saved to a folder using `useMultiFileAuthState`. On first run it creates the folder and prompts for QR/pair code. On subsequent runs it restores the session automatically.

```js
const { state, saveCreds } = await useMultiFileAuthState('./auth_info');
```

> **Important:** Always listen to `creds.update` and call `saveCreds()` to persist your session.

---

## License

MIT — see [LICENSE](LICENSE)
