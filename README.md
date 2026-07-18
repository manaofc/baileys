# Baileys — WhatsApp Web API

WhatsApp Web සඳහා bundled library — button support සහ pairing code **MANAOFC6** සමඟ.

## Files

```
index.js      — සියලු source code (single bundled file)
package.json  — dependencies
README.md     — මෙම file
```

## Install

```bash
npm install
```

## Basic Usage

```js
const {
  default: makeWASocket,
  useMultiFileAuthState,
  DisconnectReason
} = require('./index.js')

async function startBot() {
  const { state, saveCreds } = await useMultiFileAuthState('auth_info')

  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: false
  })

  // Pairing code ගන්නා ආකාරය
  if (!sock.authState.creds.registered) {
    const phoneNumber = '94XXXXXXXXX' // country code සමඟ
    const code = await sock.requestPairingCode(phoneNumber)
    console.log('Pairing Code:', code) // MANAOFC6
  }

  sock.ev.on('creds.update', saveCreds)

  sock.ev.on('connection.update', ({ connection, lastDisconnect }) => {
    if (connection === 'close') {
      const shouldReconnect =
        lastDisconnect?.error?.output?.statusCode !== DisconnectReason.loggedOut
      if (shouldReconnect) startBot()
    } else if (connection === 'open') {
      console.log('Connected to WhatsApp!')
    }
  })

  sock.ev.on('messages.upsert', async ({ messages }) => {
    for (const msg of messages) {
      console.log('Message received:', msg)
    }
  })

  return sock
}

startBot()
```

## Button Support

### Buttons Message

```js
await sock.sendMessage(jid, {
  text: 'තෝරන්න:',
  buttons: [
    { buttonId: 'btn1', buttonText: { displayText: 'Option 1' }, type: 1 },
    { buttonId: 'btn2', buttonText: { displayText: 'Option 2' }, type: 1 },
    { buttonId: 'btn3', buttonText: { displayText: 'Option 3' }, type: 1 }
  ],
  headerType: 1
})
```

### Interactive List Message

```js
await sock.sendMessage(jid, {
  text: 'List message',
  sections: [
    {
      title: 'Section 1',
      rows: [
        { title: 'Item 1', rowId: 'item1', description: 'Description 1' },
        { title: 'Item 2', rowId: 'item2', description: 'Description 2' }
      ]
    }
  ],
  buttonText: 'Click Here',
  listType: 1
})
```

### Template Buttons

```js
await sock.sendMessage(jid, {
  text: 'Template message',
  templateButtons: [
    { index: 1, urlButton: { displayText: 'Visit Site', url: 'https://example.com' } },
    { index: 2, callButton: { displayText: 'Call Now', phoneNumber: '+94XXXXXXXXX' } },
    { index: 3, quickReplyButton: { displayText: 'Quick Reply', id: 'qr1' } }
  ]
})
```

## Pairing Code

Default pairing code: **MANAOFC6**

WhatsApp connect ලෙස කිරීමට:
1. `requestPairingCode(phoneNumber)` call කරන්න
2. WhatsApp app → Linked Devices → Link a Device → Link with phone number
3. Code enter කරන්න: **MANAOFC6**

## License

MIT © Rajeh Taher / WhiskeySockets
