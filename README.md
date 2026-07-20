<div align="center">

<img src="https://files.catbox.moe/10v1uj.png" alt="Header Banner" />
</div>

---

**manaofc-baileys** is an open-source library designed to help developers build automation solutions and integrations with WhatsApp efficiently and directly. Using websocket technology without the need for a browser, this library supports a wide range of features such as message management, chat handling, group administration, as well as interactive messages and action buttons for a more dynamic user experience.

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



### Using Different Package Name
Add to your `package.json`:
```json
{
  "dependencies": {
    "manaofc-baileys": "github:manaofc/manaofc-baileys"
  }
}
```

### Import
```javascript
// ESM
import makeWASocket from 'manaofc-baileys'

// CommonJS
const { default: makeWASocket } = require('manaofc-baileys')
```

---

<details>

<summary><strong>Click to expand: Bailey Some Usage Things</strong></summary>


## 📖 Quick Start

1. **Initialize the Socket**:

```javascript

  import makeWASocket, { DisconnectReason, useMultiFileAuthState } from 'manaofc-baileys'

   const startSock = async () => {
     const { state, saveCreds } = await useMultiFileAuthState('manaofc_baileys');
     const sock = makeWASocket({
       auth: state,
       browser: ['manaofc', 'Chrome', '1.0.0'],
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
   const jid = 'xxx@s.whatsapp.net';
   await sock.sendMessage(jid, { text: 'Hello from manaofc!' });
   ```

---

| Category | Description |
|---|---|
|channels | Seamlessly send messages to WhatsApp Channels. |
| 🖱️ Buttons | Create interactive messages with button options and quick replies. |
| 📌 list | Create interactive messages with list options and quick replies. |
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



---

### 🖱️ Interactive Messaging
Send interactive messages using buttons to increase user engagement.

```js
const buttons = [
  { buttonId: "btn1", buttonText: { displayText: "Click Me" }, type: 1 },
  { buttonId: "btn2", buttonText: { displayText: "Visit Site" }, type: 1 }
];

await sock.sendMessage(id, {
  text: "Choose one:",
  footer: "Powered by manaofc",
  buttons,
  headerType: 1
});
```
### 📌 Interactive Messaging
Send interactive messages using list to increase user engagement. 

```js
const list = [
            {
                title: "hi",
                rows: [
                    { title: "list1", rowId: prefix + "set prefix ." },
                    { title: "list 2", rowId: prefix + "set prefix !" },
                ],
            },
         ];

await sock.sendMessage(id, {
  text: "Choose one:",
  footer: "Powered by manaofc",
  buttonText: "manaofc",
  list,
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
const code = await sock.requestPairingCode("9475XXXXXXXX","MANAOFC6");
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
        description: "join whatsapp", 
        location: { 
            degreesLatitude: 0, 
            degreesLongitude: 0, 
            name: "manaofc" 
        }, 
        joinLink: "https://call.whatsapp.com/xxxxxxx", 
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
        footer: 'Powered by manaofc',
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
        header: "Menu",
        title: "manaofc",      
        footer: "Powered by manaofc",      
        image: { url: "https://example.com/image.jpg" },      
        nativeFlowMessage: {        
            messageParamsJson: JSON.stringify({          
                limited_time_offer: {            
                    text: "Exclusive deal ends soon!",            
                    url: "https://whatsapp.com/channel/xxxxxxxxxxxxx",            
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
                    canonical_url: "https://example.com",            
                    domain: "https://example.com",            
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
                                        title: "manaofc",                      
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
    header: { title: 'manaofc' },
    title: 'manaofc',
    footer: 'Powered by manaofc',
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
        title: "manaofc",
        description: "welcome",
        thumbnail: { url: "https://example.com/image.jpg" },
        productId: "PROD001",
        retailerId: "RETAIL001",
        url: "https://example.com/product",
        body: "manaofc",
        footer: "Powered by manaofc",
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
        footer: "Powered by manaofc",
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
            body: "manaofc",
            mediaType: 3,
            thumbnailUrl: "https://example.com/image.jpg",
            mediaUrl: "https://example.com",
            sourceUrl: "https://example.com",
            showAdAttribution: true,
            renderLargerThumbnail: false         
        },
        buttons: [
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "Telegram",
                    url: "https://example.com",
                    merchant_url: "https://example.com"
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
        footer: "Powered by manaofc",
        document: fs.readFileSync("./package.json"),
        mimetype: "application/pdf",
        fileName: "pantatBegetar.pdf",
        jpegThumbnail: fs.readFileSync("./document.jpeg"),
        buttons: [
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "Telegram",
                    url: "https://example.com",
                    merchant_url: "https://example.com"
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

**Built with ❤️ for the WhatsApp dev community. Let's automate the future!** 🚀
