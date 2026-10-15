<div align="center">

# 🌐 @sairipedia/baileys

**WhatsApp Web API for Node.js — SairiPedia edition**

Enhanced Baileys v7 with newsletter media fixes, interactive messages, albums, and extra message types.

[![npm version](https://img.shields.io/npm/v/@sairipedia/baileys?style=for-the-badge&logo=npm&color=cb3837)](https://www.npmjs.com/package/@sairipedia/baileys)
[![downloads](https://img.shields.io/npm/dm/@sairipedia/baileys?style=for-the-badge&logo=npm&color=cb3837)](https://www.npmjs.com/package/@sairipedia/baileys)
[![node](https://img.shields.io/badge/node-%3E%3D20-339933?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![license](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![ESM](https://img.shields.io/badge/ESM-ready-f7df1e?style=for-the-badge&logo=javascript&logoColor=black)](#-import-esm--cjs)

</div>

---

> [!NOTE]
> This is a community fork maintained by **SairiPedia**. It is **not** affiliated with Meta/WhatsApp,
> and is not intended to replace upstream Baileys. Use it responsibly and at your own risk.

> [!IMPORTANT]
> **Honest fork policy.** This package builds on the work of others and keeps their credit intact (see
> [Credits](#-credits)). Unlike some re-uploads, this edition:
> - 🚫 contains **no silent auto-follow** of any channel / newsletter — your bot only follows what *you* tell it to;
> - 🔍 ships readable, un-obfuscated code you can audit;
> - 📝 states clearly what it changed and who it came from.

## ✨ What this fork adds

- 🖼️ Fix for media that could not be sent to newsletters (upstream issue).
- 🧩 Interactive messages: buttons, lists, native flows, templates, carousels.
- 🖼️ Album messages (image + video).
- 💳 Payment-related messages: requests, invites, orders, invoices.
- 📦 Sticker-pack, rich-response, code-block, inline-entity and table messages.
- 📣 `externalAdReply` without manual `contextInfo`.
- 🗄️ Reintroduced `makeInMemoryStore` (minimal ESM adaptation for v7).
- ⚙️ FFmpeg runs through `spawn` instead of `exec` for safer process handling.

## 📥 Installation

```bash
# npm
npm i @sairipedia/baileys@latest

# or pin a version
npm i @sairipedia/baileys@1.0.0
```

Or in `package.json`:

```json
"dependencies": {
   "@sairipedia/baileys": "latest"
}
```

> **Migrating from `@sairidev/baileys-new` or `@itsliaaa/baileys`?**
> The public API is the same. You can alias without touching your imports:
> ```json
> "@sairidev/baileys-new": "npm:@sairipedia/baileys@latest"
> ```

### 🧩 Import (ESM & CJS)

```javascript
// ESM
import { makeWASocket } from '@sairipedia/baileys'

// CJS
const { makeWASocket } = require('@sairipedia/baileys')
```

## 🚀 Quick start (pairing code)

```javascript
import {
   makeWASocket, delay, DisconnectReason, useMultiFileAuthState
} from '@sairipedia/baileys'
import { Boom } from '@hapi/boom'
import pino from 'pino'

const myPhoneNumber = '628xxxxxxxxxx' // bot number, no "+"

const connectToWhatsApp = async () => {
   const { state, saveCreds } = await useMultiFileAuthState('session')

   const sock = makeWASocket({
      logger: pino({ level: 'silent' }),
      auth: state
   })

   sock.ev.on('creds.update', saveCreds)

   sock.ev.on('connection.update', async (update) => {
      const { connection, lastDisconnect } = update

      if (connection === 'connecting' && !sock.authState.creds.registered) {
         await delay(1500)
         const code = await sock.requestPairingCode(myPhoneNumber)
         console.log('🔗 Pairing code :', code)
      } else if (connection === 'close') {
         const shouldReconnect =
            new Boom(lastDisconnect?.error)?.output?.statusCode !== DisconnectReason.loggedOut
         console.log('⚠️ Connection closed, reconnecting:', shouldReconnect)
         if (shouldReconnect) connectToWhatsApp()
      } else if (connection === 'open') {
         console.log('✅ Connected to WhatsApp')
      }
   })

   sock.ev.on('messages.upsert', async ({ messages }) => {
      for (const message of messages) {
         if (!message.message) continue
         await sock.sendMessage(message.key.remoteJid, { text: '👋 Hello from SairiPedia' })
      }
   })
}

connectToWhatsApp()
```

## 📚 Message features

All upstream Baileys message helpers work as documented. On top of them this fork adds the interactive,
album, payment, sticker-pack, rich-response, code-block, inline-entity and table message types listed above.
See the [upstream Baileys docs](https://github.com/WhiskeySockets/Baileys) for the shared API.

## 📦 Fork lineage

```
WhiskeySockets/Baileys   (original library)
        └── @itsliaaa/baileys   (Lia Wynn — newsletter/interactive fork)
                └── @sairipedia/baileys   (this package — rebrand, no silent auto-follow)
```

This fork **removes** the silent newsletter auto-follow that some downstream re-uploads added, and fixes a
missing `DONATE_URL` export. The rest of the behaviour matches `@itsliaaa/baileys`.

## 📣 Credits

This library is **not** original work by SairiPedia. Full respect and credit to everyone before:

- **[WhiskeySockets / Baileys](https://github.com/WhiskeySockets/Baileys)** — the original WhatsApp Web library.
- **[Lia Wynn (itsliaaa)](https://github.com/itsliaaa)** — the newsletter / interactive-message fork this package is based on.
- Original Baileys maintainers & contributors: [purpshell](https://github.com/purpshell), [jlucaso1](https://github.com/jlucaso1), [adiwajshing](https://github.com/adiwajshing).
- Protocol Buffer definitions via [WPP Connect — wa-proto](https://github.com/wppconnect-team/wa-proto).

SairiPedia's changes: rebrand, removal of silent auto-follow, restored `DONATE_URL` export.

> Please keep this credits section intact in any further fork. Forking is welcome — erasing the people
> whose work you build on is not.

## ⚖️ License

[MIT](LICENSE) — Copyright © 2025 Rajeh Taher/WhiskeySockets · © 2026 Lia Wynn · © 2026 SairiPedia.
