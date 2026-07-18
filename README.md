# Baileys — Patched (manaofc.js Compatible)

මේ Baileys `index.js` patch කරලා තියෙන්නේ `manaofc.js` bot එකෙ `NON_BUTTON: false` mode එකෙදී buttons සහ lists හරියට work කරන්න.

---

## පැච් කරලා fix කරපු දේවල්

| Problem | Fix |
|---|---|
| `buttons` field send කළාම crash | `buttonsMessage` proto properly handle |
| Image URL crash (`getStream` error) | `{ url: "..." }` format correctly pass |
| `sections` (list) crash | `listMessage` proto properly handle |

---

## Deploy කරන හැටි (Heroku)

### 1. baileys.zip download කරන්න

### 2. zip extract කරන්න — `index.js` සහ `package.json` ගන්න

### 3. Heroku ගෙ Baileys file replace කරන්න

```bash
# Heroku CLI use කරනවා නම්
heroku ps:exec --app YOUR_APP_NAME

# Server ඇතුළෙ
cp /path/to/index.js /app/node_modules/baileys/index.js
```

**හෝ** Heroku Dashboard → Resources → ඔයාගෙ dyno → Run console:

```bash
# patched index.js upload කරලා replace කරන්න
```

### 4. Bot restart කරන්න

```bash
heroku restart --app YOUR_APP_NAME
```

---

## manaofc.js Setup

### NON_BUTTON Config

`manaofc.js` ගෙ top ගාව `defaultConfig` ඇතුළෙ:

```js
const defaultConfig = {
    NON_BUTTON: false,  // ← false = real WhatsApp buttons/lists
                        // ← true  = text + image fallback
    // ... other config
};
```

| Value | Behaviour |
|---|---|
| `false` | Real WhatsApp buttons සහ list panels show වෙනවා ✅ |
| `true` | Text + numbered list + image fallback (old-style) |

---

## Button Message Format (reference)

`manaofc.js` මේ format එකෙන් buttons send කරනවා:

```js
const buttonMessage = {
  image: "https://example.com/image.jpg",  // header image (optional)
  caption: "Message body text",
  footer: "Footer text",
  headerType: 4,                            // 4 = IMAGE header
  buttons: [
    { buttonId: ".command", buttonText: { displayText: "BUTTON TEXT" }, type: 1 },
    { buttonId: ".other",   buttonText: { displayText: "OTHER BUTTON" }, type: 1 },
  ]
};

await socket.buttonMessage(from, buttonMessage, mek);
```

## List Message Format (reference)

```js
const listMessage = {
  image: "https://example.com/image.jpg",
  text: "Select an option",
  footer: "Footer text",
  buttonText: "OPEN LIST",
  sections: [
    {
      title: "Section Title",
      rows: [
        { rowId: ".command1", title: "Option 1", description: "Description" },
        { rowId: ".command2", title: "Option 2", description: "Description" },
      ]
    }
  ]
};

await socket.listMessage(from, listMessage, mek);
```

---

## File Structure

```
baileys.zip
├── index.js       ← patched Baileys library
└── package.json   ← Baileys package info

Deploy path: /app/node_modules/baileys/index.js
```

---

## Patch ගෙ Technical Details

`index.js` ගෙ `generateWAMessageContent` function එකට handlers add කරලා තියෙනවා:

```
message.buttons exist?
  └─ image ද? → prepareWAMessageMedia({ image: { url: "..." } })
              → buttonsMessage + imageMessage (headerType 4) ✅
  └─ image නෑ? → buttonsMessage text-only (headerType 1) ✅

message.sections exist?
  └─ listMessage proto build කරනවා ✅

else
  └─ original Baileys prepareWAMessageMedia (unchanged)
```

---

## Troubleshooting

**Buttons show වෙන්නේ නෑ:**
- `NON_BUTTON: false` set කරලා තියෙනවද check කරන්න
- Heroku restart කරලා නැවත try කරන්න
- WhatsApp update කරන්න (older versions render නකරන්න පුළුවන්)

**Crash / Error log:**
- Heroku logs check කරන්න: `heroku logs --tail --app YOUR_APP_NAME`
- Error paste කරන්න → fix කරන්නම්

**Image show වෙන්නේ නෑ:**
- `IMAGE_PATH` valid public URL එකක් දැම්මද check කරන්න
- URL directly browser එකෙ open වෙනවද verify කරන්න
