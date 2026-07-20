<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&height=220&color=0:ff6b6b,40:f06595,100:845ef7&text=@mr-supun-fernando/supunmd-bail&fontAlignY=40&fontSize=44&fontColor=ffffff&desc=Stable%20WhatsApp%20Web%20API%20Fork%20for%20Production%20Bots&descAlignY=60&descSize=16" alt="Header Banner" />

[![npm version](https://badge.fury.io/js/supunmd-bail.svg)](https://badge.fury.io/js/supunmd-bail)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js Version](https://img.shields.io/node/v/supunmd-bail.svg)](https://nodejs.org/)
[![Downloads](https://img.shields.io/npm/dm/supunmd-bail.svg)](https://npmjs.com/package/supunmd-bail)

</div>

---

**supunmd-bail** is an open-source library designed to help developers build automation solutions and integrations with WhatsApp efficiently and directly. Using websocket technology without the need for a browser, this library supports a wide range of features such as message management, chat handling, group administration, as well as interactive messages and action buttons for a more dynamic user experience.

Actively developed and maintained, baileys continuously receives updates to enhance stability and performance. One of the main focuses is to improve the pairing and authentication processes to be more stable and secure. Pairing features can be customized with your own codes, making the process more reliable and less prone to interruptions.

This library is highly suitable for building business bots, chat automation systems, customer service solutions, and various other communication automation applications that require high stability and comprehensive features. With a lightweight and modular design, baileys is easy to integrate into different systems and platforms.

---

### Main Features and Advantages

- Supports automatic and custom pairing processes
- Fixes previous pairing issues that often caused failures or disconnections
- Supports interactive messages, action buttons, and dynamic menus
- Efficient automatic session management for reliable operation
- Compatible with the latest multi-device features from WhatsApp
- Lightweight, stable, and easy to integrate into various systems
- Suitable for developing bots, automation, and complete communication solutions
- Comprehensive documentation and example codes to facilitate development

---

## 📦 Installation

### NPM
```bash
npm install supunmd-bail
```

### Yarn
```bash
yarn add supunmd-bail
```

### Using Different Package Name
Add to your `package.json`:
```json
{
  "dependencies": {
    "@whiskeysockets/baileys": "npm:supunmd-bail"
  }
}
```

### Import
```javascript
// ESM
import makeWASocket from 'supunmd-bail'

// CommonJS
const { default: makeWASocket } = require('supunmd-bail')
```

---

<details>

<summary><strong>Click to expand: Bailey Some Usage Things</strong></summary>


## 📖 Quick Start

1. **Initialize the Socket**:

```javascript

  import makeWASocket, { DisconnectReason, useMultiFileAuthState } from '@mr-supun-fernando/supunmd-bail'

   const startSock = async () => {
     const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys');
     const sock = makeWASocket({
       auth: state,
       browser: ['SupunMd', 'Chrome', '1.0.0'],
       printQRInTerminal: true, // Set to false for custom QR handling
       syncFullHistory: false, // Optimize for production
     });

     sock.ev.on('creds.update', saveCreds);

     sock.ev.on('connection.update', (update) => {
       const { connection, lastDisconnect } = update;
       if (connection === 'close') {
         const shouldReconnect = (lastDisconnect?.error)?.output?.statusCode !== DisconnectReason.loggedOut;
         console.log('Connection closed due to ', lastDisconnect?.error, ', reconnecting ', shouldReconnect);
         if (shouldReconnect) startSock(); // Auto-reconnect
       } else if (connection === 'open') {
         console.log('✅ Connected to WhatsApp');
       }
     });

     return sock;
   };

   const sock = await startSock();
   ```

2. **Send a Basic Message**:
   ```javascript
   const jid = 'recipient@s.whatsapp.net';
   await sock.sendMessage(jid, { text: 'Hello from SupunMd!' });
   ```

---

| Category | Description |
|---|---|
|channels | Seamlessly send messages to WhatsApp Channels. |
| 🖱️ Buttons | Create interactive messages with button options and quick replies. |
| 🖼️ Albums | Send grouped images or videos as an album (carousel-like format). |
| 👤 LID Grouping | Handle group operations using the latest @lid addressing style. |
| 🤖 AI Message Style | Add a stylized “AI” icon to messages. |
| 📷 HD Profile Pics | Upload full-size profile pictures without cropping. |
| 🔐 Pairing Code | Generate custom alphanumeric pairing codes. |
| 🛠️ Dev Experience | Reduced noise from logs with optimized libsignal printouts. |

---

## 🔧 API Reference

### Utility Functions

## 🚀 Features & Usage

### 📬 Newsletter Control
Manage WhatsApp Newsletters (Channels), from creation to message interactions.

```js
// Create a newsletter
await sock.newsletterCreate("SupunMd Update");

// Update description
await sock.newsletterUpdateDescription(
  "1234XXXX@newsletter",
  "YOO updates come daily"
);

// React to a channel message
await sock.newsletterReactMessage(
  "1234XXXX@newsletter",
  "192",
  "💜"
);
```

---

### Custom Pairing Code
```javascript
const phoneNumber = "947XXXXX"
const code = await sock.requestPairingCode(phoneNumber.trim(), "ABCDE01")
console.log("Your pairing code: " + code)
```

---

### 📌 Interactive Messaging
Send interactive messages using buttons to increase user engagement.

```js
const buttons = [
  { buttonId: "btn1", buttonText: { displayText: "Click Me" }, type: 1 },
  { buttonId: "btn2", buttonText: { displayText: "Visit Site" }, type: 1 }
];

await sock.sendMessage(id, {
  text: "Choose one:",
  footer: "Mova - Nest | Lk",
  buttons,
  headerType: 1
});
```

---

### 🖼️ Send Album
Send multiple media (images or videos) in a single album message.

```js
await sock.sendMessage(jid, { 
    albumMessage: [
        { image: { url: "https://example.com/pic1.jpg" }, caption: "Celon Memories" },
        { image: { url: "https://example.com/pic1.jpg" }, caption: "Celon Memories" }
    ] 
}, { quoted: m });

```

---

### 🔐 Custom Pairing Code
Pair a WhatsApp device using a custom code.

```js
const code = await sock.requestPairingCode("94XXXXXXXX","ABCDE123");
console.log("Pairing Code:", code);
```

---

#### Event Message
Invite users to virtual events with location and RSVP links.

```javascript
await sock.sendMessage(jid, { 
    eventMessage: { 
        isCanceled: false, 
        name: "Technology Meetup 2026", 
        description: "Join us for AI innovations!", 
        location: { 
            degreesLatitude: 0, 
            degreesLongitude: 0, 
            name: "SUPUN MD" 
        }, 
        joinLink: "https://call.whatsapp.com/video/event123", 
        startTime: "1763019000", 
        endTime: "1763026200", 
        extraGuestsAllowed: false 
    } 
}, { quoted: m });
```

#### Poll Result Message
Share poll outcomes with vote tallies.

```javascript
await sock.sendMessage(jid, { 
    pollResultMessage: { 
        name: "Hello World", 
        pollVotes: [
            {
                optionName: "TEST 1",
                optionVoteCount: "112233"
            },
            {
                optionName: "TEST 2",
                optionVoteCount: "1"
            }
        ] 
    } 
}, { quoted: m });
```

#### Simple Interactive Message
Engage with copyable CTAs.

```javascript
await sock.sendMessage(jid, {
    interactiveMessage: {
        header: 'Quick Action',
        title: 'Copy this code',
        footer: 'Powered by @SupunFernando',
        buttons: [
            {
                name: "cta_copy",
                buttonParamsJson: JSON.stringify({
                    display_text: "copy code",
                    id: "123456789",              
                    copy_code: "ABC123XYZ"
                })
            }
        ]
    }
}, { quoted: m });
```

#### Advanced Interactive Message (Native Flow)
Full native flows with lists, sheets, and offers.

```javascript
await sock.sendMessage(jid, {    
    interactiveMessage: {      
        header: "Dynamic Menu",
        title: "Explore Options",      
        footer: "Contact @SupunFernando for support",      
        image: { url: "https://example.com/image.jpg" },      
        nativeFlowMessage: {        
            messageParamsJson: JSON.stringify({          
                limited_time_offer: {            
                    text: "Exclusive deal ends soon!",            
                    url: "https://whatsapp.com/channel/0029Vb55cv41nozBTTIw1y07",            
                    copy_code: "DEAL2026",            
                    expiration_time: Date.now() * 86400000          
                },          
                bottom_sheet: {            
                    in_thread_buttons_limit: 2,            
                    divider_indices: [1, 2, 3, 4, 5, 999],            
                    list_title: "Categories",            
                    button_title: "View All"          
                },          
                tap_target_configuration: {            
                    title: "Learn More",            
                    description: "Advanced automation tips",            
                    canonical_url: "https://www.npmjs.com/package/mr-supun-fernando",            
                    domain: "supunmd.vercel.com",            
                    button_index: 0          
                }        
            }),        
            buttons: [          
                {            
                    name: "single_select",            
                    buttonParamsJson: JSON.stringify({              
                        has_multiple_buttons: true            
                    })          
                },          
                {            
                    name: "call_permission_request",            
                    buttonParamsJson: JSON.stringify({              
                        has_multiple_buttons: true            
                    })          
                },          
                {            
                    name: "single_select",            
                    buttonParamsJson: JSON.stringify({              
                        title: "Hello World",              
                        sections: [                
                            {                  
                                title: "title",                  
                                highlight_label: "label",                  
                                rows: [                    
                                    {                      
                                        title: "@SupunFernando",                      
                                        description: "love you",                      
                                        id: "row_2"                    
                                    }                  
                                ]                
                            }              
                        ],              
                        has_multiple_buttons: true            
                    })          
                },          
                {            
                    name: "cta_copy",            
                    buttonParamsJson: JSON.stringify({              
                        display_text: "copy code",              
                        id: "123456789",              
                        copy_code: "ABC123XYZ"            
                    })          
                }        
            ]      
        }    
    }  
}, { quoted: m });
```

#### Interactive Message with Thumbnail
Rich previews for products/services.

```javascript
await sock.sendMessage(jid, {
  interactiveMessage: {
    header: { title: 'Featured Item' },
    title: 'Supun Md',
    footer: 'Upgrade today',
    image: { url: 'https://example.com/product-thumb.jpg' },
    buttons: [
      {
        name: 'cta_copy',
        buttonParamsJson: JSON.stringify({
          display_text: 'copy code',
          id: '123456789',
          copy_code: 'ABC123XYZ'
        })
      }
    ]
  }
}, { quoted: m });
```

#### Product Message
E-commerce ready catalogs.

```javascript
await sock.sendMessage(jid, {
    productMessage: {
        title: "Produk Contoh",
        description: "Ini adalah deskripsi produk",
        thumbnail: { url: "https://example.com/image.jpg" },
        productId: "PROD001",
        retailerId: "RETAIL001",
        url: "https://example.com/product",
        body: "Detail produk",
        footer: "Harga spesial",
        priceAmount1000: 50000,
        currencyCode: "USD",
        buttons: [
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "Beli Sekarang",
                    url: "https://example.com/buy"
                })
            }
        ]
    }
}, { quoted: m });
```

### Interactive Message with Document Buffer
Send interactive messages with document from buffer (file system) - **Note: Documents only support buffer**:

```javascript
await sock.sendMessage(jid, {
    interactiveMessage: {
        header: "Hello World",
        title: "Hello World",
        footer: "Powered by @SupunFernando",
        document: fs.readFileSync("./package.json"),
        mimetype: "application/pdf",
        fileName: "pantatBegetar.pdf",
        jpegThumbnail: fs.readFileSync("./document.jpeg"),
        contextInfo: {
            mentionedJid: [jid],
            forwardingScore: 777,
            isForwarded: false
        },
        externalAdReply: {
            title: "Wabot",
            body: "Z team",
            mediaType: 3,
            thumbnailUrl: "https://example.com/image.jpg",
            mediaUrl: " X ",
            sourceUrl: "https://t.me/pantatBegetar",
            showAdAttribution: true,
            renderLargerThumbnail: false         
        },
        buttons: [
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "Telegram",
                    url: "https://t.me/pantatBegetar",
                    merchant_url: "https://t.me/pantatBegetar"
                })
            }
        ]
    }
}, { quoted: m });
```


### Interactive Message with Document Buffer (Simple)
Send interactive messages with document from buffer (file system) without contextInfo and externalAdReply - **Note: Documents only support buffer**:

```javascript
await sock.sendMessage(jid, {
    interactiveMessage: {
        header: "Hello World",
        title: "Hello World",
        footer: "Powered by @SupunFernando",
        document: fs.readFileSync("./package.json"),
        mimetype: "application/pdf",
        fileName: "pantatBegetar.pdf",
        jpegThumbnail: fs.readFileSync("./document.jpeg"),
        buttons: [
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "Telegram",
                    url: "https://t.me/pantatBegetar",
                    merchant_url: "https://t.me/pantatBegetar"
                })
            }
        ]
    }
}, { quoted: m });
```

### Request Payment Message
Send payment request messages with custom background and sticker:

```javascript
let quotedType = m.quoted?.mtype || '';
let quotedContent = JSON.stringify({ [quotedType]: m.quoted }, null, 2);

await sock.sendMessage(jid, {
    requestPaymentMessage: {
        currency: "IDR",
        amount: 10000000,
        from: m.sender,
        sticker: JSON.parse(quotedContent),
        background: {
            id: "100",
            fileLength: "0",
            width: 1000,
            height: 1000,
            mimetype: "image/webp",
            placeholderArgb: 0xFF00FFFF,
            textArgb: 0xFFFFFFFF,     
            subtextArgb: 0xFFAA00FF   
        }
    }
}, { quoted: m });
```

---

</details>

## Why Choose WhatsApp Baileys?

Because this library offers high stability, full features, and an actively improved pairing process. It is ideal for developers aiming to create professional and secure WhatsApp automation solutions. Support for the latest WhatsApp features ensures compatibility with platform updates.

---

### Technical Notes

- Supports custom pairing codes that are stable and secure
- Fixes previous issues related to pairing and authentication
- Features interactive messages and action buttons for dynamic menu creation
- Automatic and efficient session management for long-term stability
- Compatible with the latest multi-device features from WhatsApp
- Easy to integrate and customize based on your needs
- Perfect for developing bots, customer service automation, and other communication applications

---

## 📞 Support

- **Issues**: [Whatsapp Channel](https://whatsapp.com/channel/0029Vb55cv41nozBTTIw1y07)
- **Discussions**: [Whatsapp Channel](https://whatsapp.com/channel/0029Vb55cv41nozBTTIw1y07)
- **NPM**: [@mr-supun-fernando/baileyz](https://www.npmjs.com/package/mr-supun-fernando/supunmd-bail)

**Built with ❤️ for the WhatsApp dev community. Let's automate the future!** 🚀
