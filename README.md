# 🤖 manaofc-baileys

<p align="center">
   WhatsApp Web API — Baileys fork with button/list click fix and interactive message support for manaofc bots.
   <br><br>
   <img src="https://img.shields.io/badge/version-0.0.6-blue?style=for-the-badge"/>
   <img src="https://img.shields.io/badge/license-MIT-blue?style=for-the-badge"/>
   <img src="https://img.shields.io/badge/node-%3E%3D20-339933?logo=node.js&labelColor=green&logoColor=white&style=for-the-badge"/>
</p>

---

### ✨ What's Fixed in This Fork

- 🔘 **Button click fix** — When `NON_BUTTON = false`, native WhatsApp buttons and list selections now correctly trigger bot commands. Previously clicking a button did nothing.
- 📋 **List selection fix** — List `rowId` is now properly received as a command body.
- ⚡ **Interactive response fix** — `interactiveResponseMessage` / nativeFlow button replies are now normalized correctly.

#### How the fix works

When a user taps a native button or selects a list item, WhatsApp sends a `buttonsResponseMessage` or `listResponseMessage`. This fork's `normalizeMessageContent` converts those into a plain `conversation` message containing the `buttonId` / `rowId`, so your bot's command system processes them automatically — no extra code needed.

| User Action | Before Fix | After Fix |
|---|---|---|
| Button tap | `buttonsResponseMessage` → ignored | → `conversation: ".ping"` → command runs ✅ |
| List select | `listResponseMessage` → ignored | → `conversation: ".help"` → command runs ✅ |
| Interactive tap | `interactiveResponseMessage` → ignored | → `conversation: "<id>"` → command runs ✅ |

---

### 📥 Installation

```bash
# Copy index.js and package.json to your project, then:
npm install
```

Or add as a local dependency in your bot's `package.json`:

```json
"dependencies": {
   "manaofc-baileys": "file:./manaofc-baileys"
}
```

#### 🧩 Import (CJS)

```javascript
const {
  makeWASocket,
  useMultiFileAuthState,
  DisconnectReason,
  getContentType,
  normalizeMessageContent,
  generateWAMessage,
  prepareWAMessageMedia
} = require('manaofc-baileys')
```

---

### 🌐 Connect to WhatsApp

```javascript
const { makeWASocket, useMultiFileAuthState, DisconnectReason } = require('manaofc-baileys')
const { Boom } = require('@hapi/boom')
const pino = require('pino')

const logger = pino({ level: 'silent' })

const connectToWhatsApp = async () => {
   const { state, saveCreds } = await useMultiFileAuthState('session')

   const sock = makeWASocket({
      logger,
      auth: state
   })

   sock.ev.on('creds.update', saveCreds)

   sock.ev.on('connection.update', async (update) => {
      const { connection, lastDisconnect } = update

      if (connection === 'close') {
         const shouldReconnect = new Boom(lastDisconnect?.error)?.output?.statusCode !== DisconnectReason.loggedOut
         if (shouldReconnect) connectToWhatsApp()
      } else if (connection === 'open') {
         console.log('✅ Connected to WhatsApp')
      }
   })
}

connectToWhatsApp()
```

---

### 🔐 Auth State

```javascript
const { useMultiFileAuthState } = require('manaofc-baileys')

const { state, saveCreds } = await useMultiFileAuthState('session')
```

---

### ✉️ Sending Messages

#### 🔠 Text

```javascript
sock.sendMessage(jid, { text: '👋 Hello' }, { quoted: message })
```

#### 😁 Reaction

```javascript
sock.sendMessage(jid, {
   react: { key: message.key, text: '✨' }
})
```

#### 📌 Pin Message

```javascript
sock.sendMessage(jid, {
   pin: message.key,
   time: 86400, // 1d = 86400, 7d = 604800, 30d = 2592000
   type: 1      // 2 to unpin
})
```

#### ➡️ Forward Message

```javascript
sock.sendMessage(jid, { forward: message, force: true })
```

#### 📍 Location

```javascript
sock.sendMessage(jid, {
   location: {
      degreesLatitude: 24.121231,
      degreesLongitude: 55.1121221,
      name: '📍 I am here'
   }
}, { quoted: message })
```

#### 📊 Poll

```javascript
sock.sendMessage(jid, {
   poll: {
      name: '🔥 Voting',
      values: ['Yes', 'No'],
      selectableCount: 1
   }
}, { quoted: message })
```

---

### 📁 Sending Media Messages

> For media, pass a `Buffer`, `{ url: 'https://...' }`, or `{ url: './local/path' }`.

#### 🖼️ Image

```javascript
sock.sendMessage(jid, {
   image: { url: './path/to/image.jpg' },
   caption: '🔥 Caption here'
}, { quoted: message })
```

#### 🎥 Video

```javascript
sock.sendMessage(jid, {
   video: { url: './path/to/video.mp4' },
   caption: '🎬 Caption here',
   gifPlayback: false
}, { quoted: message })
```

#### 💽 Audio

```javascript
sock.sendMessage(jid, {
   audio: { url: './path/to/audio.mp3' },
   ptt: false // true = voice note
}, { quoted: message })
```

#### 🗂️ Document

```javascript
sock.sendMessage(jid, {
   document: { url: './path/to/file.pdf' },
   mimetype: 'application/pdf',
   caption: '📄 My file'
}, { quoted: message })
```

#### 📃 Sticker

```javascript
sock.sendMessage(jid, {
   sticker: { url: './path/to/sticker.webp' }
}, { quoted: message })
```

---

### 👉 Sending Interactive Messages

#### 🔘 Buttons (NON_BUTTON = false)

```javascript
// Text + buttons
sock.sendMessage(jid, {
   text: '👆 Choose an option:',
   footer: 'manaofc-baileys',
   buttons: [{
      buttonId: '.ping',
      buttonText: { displayText: '🏓 Ping' },
      type: 1
   }, {
      buttonId: '.menu',
      buttonText: { displayText: '📋 Menu' },
      type: 1
   }]
}, { quoted: message })

// Image + buttons
sock.sendMessage(jid, {
   image: { url: 'https://example.com/image.jpg' },
   caption: '👆 Choose an option:',
   footer: 'manaofc-baileys',
   headerType: 4,
   buttons: [{
      buttonId: '.help',
      buttonText: { displayText: '❓ Help' },
      type: 1
   }]
}, { quoted: message })
```

> ✅ **With this fix**, when a user taps a button with `buttonId: '.ping'`, the bot receives it as `body = ".ping"` and runs the `.ping` command automatically.

#### 📋 List (NON_BUTTON = false)

> Only works in private chat (`@s.whatsapp.net`).

```javascript
sock.sendMessage(jid, {
   text: '📋 Select an option:',
   footer: 'manaofc-baileys',
   buttonText: '📋 Open Menu',
   title: '🤖 Bot Menu',
   sections: [{
      title: '🚀 Main Menu',
      rows: [{
         title: '🏓 Ping',
         description: 'Check bot speed',
         rowId: '.ping'
      }, {
         title: '📋 Menu',
         description: 'Show full menu',
         rowId: '.menu'
      }]
   }]
}, { quoted: message })
```

> ✅ **With this fix**, when a user selects a row with `rowId: '.ping'`, the bot receives it as `body = ".ping"` and runs the command automatically.

#### 💭 Button Reply (manual)

```javascript
// buttonsResponseMessage
sock.sendMessage(jid, {
   type: 'plain',
   buttonReply: {
      id: '.menu',
      displayText: '📋 Menu'
   }
}, { quoted: message })

// listResponseMessage
sock.sendMessage(jid, {
   listReply: {
      title: '📄 Selected',
      description: '📋 Menu',
      id: '.menu'
   }
}, { quoted: message })
```

---

### ♻️ Modify Messages

#### 🗑️ Delete

```javascript
sock.sendMessage(jid, { delete: message.key })
```

#### ✏️ Edit

```javascript
sock.sendMessage(jid, {
   text: '✨ Edited text!',
   edit: message.key
})
```

---

### 👥 Group Management

```javascript
// Get group info
const metadata = await sock.groupMetadata(jid)

// Add participants
sock.groupParticipantsUpdate(jid, ['628123456789@s.whatsapp.net'], 'add')

// Remove participants
sock.groupParticipantsUpdate(jid, ['628123456789@s.whatsapp.net'], 'remove')

// Promote to admin
sock.groupParticipantsUpdate(jid, ['628123456789@s.whatsapp.net'], 'promote')

// Demote from admin
sock.groupParticipantsUpdate(jid, ['628123456789@s.whatsapp.net'], 'demote')

// Change group name
sock.groupUpdateSubject(jid, '📦 Group Name')

// Change group description
sock.groupUpdateDescription(jid, 'Updated description')

// Change group photo
sock.updateProfilePicture(jid, { url: 'path/to/image.jpg' })

// Set admin-only chat
sock.groupSettingUpdate(jid, 'announcement')

// Set open chat
sock.groupSettingUpdate(jid, 'not_announcement')

// Get invite code
const inviteCode = await sock.groupInviteCode(jid)
```

---

### 👤 Profile Management

```javascript
// Get profile picture
const url = await sock.profilePictureUrl(jid, 'image')

// Update name
sock.updateProfileName('My Bot Name')

// Update status
sock.updateProfileStatus('Available 24/7')

// Send presence
sock.sendPresenceUpdate('available', jid)

// Read messages
sock.readMessages([message.key])

// Block / Unblock
sock.updateBlockStatus(jid, 'block')
sock.updateBlockStatus(jid, 'unblock')
```

---

### 🔐 Privacy Management

```javascript
sock.updateLastSeenPrivacy('all')         // 'all' | 'contacts' | 'nobody'
sock.updateOnlinePrivacy('all')           // 'all' | 'match_last_seen'
sock.updateProfilePicturePrivacy('contacts')
sock.updateReadReceiptsPrivacy('all')     // 'all' | 'none'
sock.updateGroupsAddPrivacy('contacts')   // 'all' | 'contacts'
```

---

### 📡 Events

```javascript
sock.ev.on('connection.update', (update) => {})
sock.ev.on('creds.update', (update) => {})
sock.ev.on('messages.upsert', ({ messages }) => {})
sock.ev.on('messages.update', (update) => {})
sock.ev.on('messages.delete', (update) => {})
sock.ev.on('messages.reaction', (update) => {})
sock.ev.on('groups.upsert', (update) => {})
sock.ev.on('group-participants.update', (update) => {})
sock.ev.on('contacts.upsert', (update) => {})
sock.ev.on('presence.update', (update) => {})
```

---
