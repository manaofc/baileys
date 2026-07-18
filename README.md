# Baileys - WhatsApp Web API Library

WhatsApp Web සඳහා WebSocket library එකක් — button support සහ pairing code සමඟ.

## Files

```
index.js      - Bundled library (සියලු source code)
package.json  - Dependencies
README.md     - මෙම file
```

## Install

```bash
npm install
```

## Usage

```js
const { default: makeWASocket, useMultiFileAuthState, DisconnectReason } = require('./index.js')

async function startBot() {
  const { state, saveCreds } = await useMultiFileAuthState('auth_info')

  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: false
  })

  // Pairing code ගන්නා ආකාරය
  if (!sock.authState.creds.registered) {
    const phoneNumber = '94XXXXXXXXX' // ඔබේ phone number
    const code = await sock.requestPairingCode(phoneNumber)
    console.log('Pairing Code:', code) // MANAOFC6
  }

  sock.ev.on('creds.update', saveCreds)

  sock.ev.on('connection.update', ({ connection, lastDisconnect }) => {
    if (connection === 'close') {
      const shouldReconnect = lastDisconnect?.error?.output?.statusCode !== DisconnectReason.loggedOut
      if (shouldReconnect) startBot()
    } else if (connection === 'open') {
      console.log('Connected!')
    }
  })

  // Message receive කිරීම
  sock.ev.on('messages.upsert', async ({ messages }) => {
    for (const msg of messages) {
      console.log('Message:', msg)
    }
  })
}

startBot()
```

## Button Support

### Buttons Message

```js
await sock.sendMessage(jid, {
  text: 'තෝරන්න:',
  buttons: [
    { buttonId: 'id1', buttonText: { displayText: 'Option 1' }, type: 1 },
    { buttonId: 'id2', buttonText: { displayText: 'Option 2' }, type: 1 }
  ],
  headerType: 1
})
```

### Interactive Message (List)

```js
await sock.sendMessage(jid, {
  text: 'List Message',
  sections: [
    {
      title: 'Section 1',
      rows: [
        { title: 'Row 1', rowId: 'row1' },
        { title: 'Row 2', rowId: 'row2' }
      ]
    }
  ],
  buttonText: 'Click Here',
  listType: 1
})
```

## Pairing Code

Default pairing code: **MANAOFC6**

Bot connect කිරීමට:
1. `requestPairingCode(phoneNumber)` call කරන්න
2. WhatsApp → Linked Devices → Link a Device → Link with phone number
3. Code enter කරන්න

## Dependencies

```json
{
  "@hapi/boom": "^10.0.1",
  "axios": "^1.7.2",
  "lodash": "^4.x",
  "pino": "^9.x",
  "protobufjs": "^7.x",
  "ws": "^8.x"
}
```

## License

MIT © manafc / WhiskeySockets
