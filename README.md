# WhatsApp Baileys — Dokumentasi Lengkap

> **Modified Baileys** by [sairidev](https://github.com/sairidev) — Fork modifikasi dari Baileys dengan dukungan pesan interaktif, native flow, pairing kustom, dan berbagai fitur WhatsApp terbaru.

---

## Daftar Isi

- [Instalasi](#instalasi)
- [Persyaratan Sistem](#persyaratan-sistem)
- [Koneksi & Autentikasi](#koneksi--autentikasi)
- [Tipe Pesan yang Didukung](#tipe-pesan-yang-didukung)
  - [Pesan Teks](#1-pesan-teks)
  - [Pesan Gambar](#2-pesan-gambar)
  - [Pesan Video](#3-pesan-video)
  - [Pesan Audio](#4-pesan-audio)
  - [Pesan Dokumen / File](#5-pesan-dokumen--file)
  - [Pesan Sticker](#6-pesan-sticker)
  - [Pesan Lokasi](#7-pesan-lokasi)
  - [Pesan Kontak](#8-pesan-kontak)
  - [Pesan Poll](#9-pesan-poll)
  - [Pesan Reaksi](#10-pesan-reaksi)
  - [Album Message](#11-album-message-multiple-images)
  - [Order Message](#12-order-message)
  - [Event Message](#13-event-message)
  - [Poll Result Message](#14-poll-result-message)
  - [Product Message](#15-product-message)
  - [Request Payment Message](#16-request-payment-message)
  - [Review and Pay Message](#17-review-and-pay-message)
  - [Group Status Message](#18-group-status-message)
  - [Interactive Message](#19-interactive-message)
  - [Interactive dengan Native Flow](#20-interactive-message-dengan-native-flow)
  - [Interactive dengan Document Buffer](#21-interactive-message-dengan-document)
  - [Interactive dengan External Ad Reply](#22-interactive-message-dengan-external-ad-reply)
- [Manajemen Grup](#manajemen-grup)
- [Manajemen Chat & Kontak](#manajemen-chat--kontak)
- [Fitur Presence & Status](#fitur-presence--status)
- [Manajemen Sesi & Store](#manajemen-sesi--store)
- [Event System](#event-system)
- [Dependencies](#dependencies)
- [Catatan Teknis](#catatan-teknis)

---

## Instalasi

```bash
# Menggunakan npm
npm install github:sairidev/baileys

# Menggunakan yarn
yarn add github:sairidev/baileys
```

---

## Persyaratan Sistem

| Kebutuhan | Versi Minimum |
|-----------|---------------|
| Node.js | `>= 20.0.0` |
| Package Manager | `yarn 1.22.19` atau `npm` |
| OS | Linux / macOS / Windows |

### Peer Dependencies (Opsional)

```bash
# Untuk pemrosesan gambar
npm install sharp
# atau
npm install jimp

# Untuk preview link
npm install link-preview-js

# Untuk QR Code di terminal
npm install qrcode-terminal
```

---

## Koneksi & Autentikasi

### Koneksi Dasar dengan QR Code

```javascript
const { makeWASocket, useMultiFileAuthState, DisconnectReason } = require('@sairidev/baileys2');
const { Boom } = require('@hapi/boom');

async function connectToWhatsApp() {
  const { state, saveCreds } = await useMultiFileAuthState('auth_info');

  const client = makeWASocket({
    auth: state,
    printQRInTerminal: true,
  });

  client.ev.on('connection.update', async (update) => {
    const { connection, lastDisconnect } = update;

    if (connection === 'close') {
      const shouldReconnect =
        (lastDisconnect.error instanceof Boom)?.output?.statusCode !== DisconnectReason.loggedOut;

      if (shouldReconnect) {
        connectToWhatsApp();
      } else {
        console.log('Terputus. Silakan scan ulang QR.');
      }
    } else if (connection === 'open') {
      console.log('Terhubung ke WhatsApp!');
    }
  });

  client.ev.on('creds.update', saveCreds);
  return client;
}

connectToWhatsApp();
```

### Koneksi dengan Custom Pairing Code

```javascript
const { makeWASocket, useMultiFileAuthState } = require('@sairidev/baileys2');

async function connectWithPairingCode() {
  const { state, saveCreds } = await useMultiFileAuthState('auth_info');

  const client = makeWASocket({
    auth: state,
    printQRInTerminal: false,
  });

  // Gunakan nomor tanpa + dan tanpa spasi, contoh: "628123456789"
  if (!client.authState.creds.registered) {
    const phoneNumber = '628123456789';
    const code = await client.requestPairingCode(phoneNumber);
    console.log('Pairing Code:', code); // Masukkan kode ini di WhatsApp > Perangkat Tertaut
  }

  client.ev.on('creds.update', saveCreds);
  return client;
}

connectWithPairingCode();
```

---

## Tipe Pesan yang Didukung

### 1. Pesan Teks

```javascript
// Teks biasa
await client.sendMessage(jid, { text: 'Halo dunia!' });

// Dengan mention
await client.sendMessage(jid, {
  text: '@628123456789 Halo!',
  mentions: ['628123456789@s.whatsapp.net']
});

// Sebagai reply / quoted
await client.sendMessage(jid, { text: 'Ini balasan' }, { quoted: m });
```

---

### 2. Pesan Gambar

```javascript
const fs = require('fs');

// Dari buffer
await client.sendMessage(jid, {
  image: fs.readFileSync('./gambar.jpg'),
  caption: 'Caption gambar'
});

// Dari URL
await client.sendMessage(jid, {
  image: { url: 'https://example.com/gambar.jpg' },
  caption: 'Caption gambar dari URL'
});

// Sebagai view once (sekali lihat)
await client.sendMessage(jid, {
  image: { url: 'https://example.com/gambar.jpg' },
  viewOnce: true,
  caption: 'Pesan sekali lihat'
});
```

---

### 3. Pesan Video

```javascript
// Video biasa
await client.sendMessage(jid, {
  video: fs.readFileSync('./video.mp4'),
  caption: 'Caption video'
});

// Video sebagai GIF
await client.sendMessage(jid, {
  video: fs.readFileSync('./animasi.mp4'),
  gifPlayback: true,
  caption: 'Ini GIF'
});

// Video note (PTV / Picture-in-Picture)
await client.sendMessage(jid, {
  video: { url: './video.mp4' },
  ptv: true
});
```

---

### 4. Pesan Audio

```javascript
// Audio biasa
await client.sendMessage(jid, {
  audio: fs.readFileSync('./audio.mp3'),
  mimetype: 'audio/mp4'
});

// Voice note (pesan suara)
await client.sendMessage(jid, {
  audio: fs.readFileSync('./voice.ogg'),
  mimetype: 'audio/ogg; codecs=opus',
  ptt: true // push-to-talk = voice note
});
```

> **Catatan:** Agar audio berfungsi di semua perangkat, konversi menggunakan ffmpeg:
> ```bash
> ffmpeg -i input.mp3 -c:a libopus -ac 1 -avoid_negative_ts make_zero output.ogg
> ```

---

### 5. Pesan Dokumen / File

```javascript
// Dokumen dari buffer
await client.sendMessage(jid, {
  document: fs.readFileSync('./file.pdf'),
  mimetype: 'application/pdf',
  fileName: 'dokumen.pdf'
});

// Dokumen dari URL
await client.sendMessage(jid, {
  document: { url: 'https://example.com/file.pdf' },
  mimetype: 'application/pdf',
  fileName: 'dokumen.pdf'
});
```

---

### 6. Pesan Sticker

```javascript
// Sticker dari buffer (webp)
await client.sendMessage(jid, {
  sticker: fs.readFileSync('./sticker.webp')
});

// Sticker animasi
await client.sendMessage(jid, {
  sticker: fs.readFileSync('./sticker-animated.webp'),
  isAnimated: true
});
```

---

### 7. Pesan Lokasi

```javascript
// Lokasi biasa
await client.sendMessage(jid, {
  location: {
    degreesLatitude: -6.2088,
    degreesLongitude: 106.8456,
    name: 'Jakarta',
    address: 'DKI Jakarta, Indonesia'
  }
});

// Live location (lokasi real-time)
await client.sendMessage(jid, {
  liveLocation: {
    degreesLatitude: -6.2088,
    degreesLongitude: 106.8456,
    accuracyInMeters: 10,
    speedInMps: 0,
    degreesClockwiseFromMagneticNorth: 0,
    name: 'Posisi Saya',
    address: 'Jakarta'
  },
  caption: 'Ini lokasi saya sekarang'
});
```

---

### 8. Pesan Kontak

```javascript
// Kirim satu kontak
await client.sendMessage(jid, {
  contacts: {
    displayName: 'Nama Kontak',
    contacts: [
      {
        vcard: `BEGIN:VCARD\nVERSION:3.0\nFN:Nama Kontak\nTEL;type=CELL;type=VOICE;waid=628123456789:+62 812-3456-789\nEND:VCARD`
      }
    ]
  }
});

// Kirim beberapa kontak
await client.sendMessage(jid, {
  contacts: {
    displayName: '2 Kontak',
    contacts: [
      { vcard: `BEGIN:VCARD\nVERSION:3.0\nFN:Kontak Satu\nTEL;waid=628111111111:+62 811-1111-111\nEND:VCARD` },
      { vcard: `BEGIN:VCARD\nVERSION:3.0\nFN:Kontak Dua\nTEL;waid=628222222222:+62 822-2222-222\nEND:VCARD` }
    ]
  }
});
```

---

### 9. Pesan Poll

```javascript
// Buat polling
await client.sendMessage(jid, {
  poll: {
    name: 'Pilihan Warna Favorit',
    values: ['Merah', 'Hijau', 'Biru', 'Kuning'],
    selectableCount: 1 // 1 = single choice, >1 = multiple choice
  }
});
```

---

### 10. Pesan Reaksi

```javascript
// Kirim reaksi ke pesan
await client.sendMessage(jid, {
  react: {
    text: '❤️', // emoji reaksi
    key: m.key   // key dari pesan yang ingin direaksi
  }
});

// Hapus reaksi
await client.sendMessage(jid, {
  react: {
    text: '', // string kosong = hapus reaksi
    key: m.key
  }
});
```

---

### 11. Album Message (Multiple Images)

Mengirim banyak gambar sekaligus dalam satu album.

```javascript
await client.sendMessage(jid, {
  albumMessage: [
    { image: fs.readFileSync('./foto1.jpg'), caption: 'Foto pertama' },
    { image: fs.readFileSync('./foto2.jpg'), caption: 'Foto kedua' },
    { image: { url: 'https://example.com/foto3.jpg' }, caption: 'Foto ketiga' }
  ]
}, { quoted: m });
```

---

### 12. Order Message

Mengirim pesan order/pesanan seperti di WhatsApp Business.

```javascript
await client.sendMessage(jid, {
  orderMessage: {
    orderId: '7778',
    thumbnail: await (await fetch('https://example.com/produk.jpg')).buffer(),
    itemCount: 1,
    status: 'INQUIRY',       // INQUIRY | ACCEPTED | DECLINED | EXPIRED
    surface: 'CATALOG',
    message: 'Pesan produk ini',
    orderTitle: 'Judul Order',
    sellerJid: '0@s.whatsapp.net',
    token: Buffer.from('777777'),
    totalAmount1000: 50000,  // Harga dalam satuan 1/1000 (50000 = Rp 50)
    currencyCode: 'IDR',
    messageVersion: 2
  }
}, { quoted: m });
```

---

### 13. Event Message

Membuat dan mengirim undangan event WhatsApp.

```javascript
await client.sendMessage(jid, {
  eventMessage: {
    isCanceled: false,
    name: 'Nama Event',
    description: 'Deskripsi event lengkap di sini',
    location: {
      degreesLatitude: -6.2088,
      degreesLongitude: 106.8456,
      name: 'Gedung Serbaguna Jakarta'
    },
    joinLink: 'https://call.whatsapp.com/video/xxxxx',
    startTime: '1763019000', // Unix timestamp
    endTime: '1763026200',   // Unix timestamp
    extraGuestsAllowed: true
  }
}, { quoted: m });
```

---

### 14. Poll Result Message

Menampilkan hasil polling beserta jumlah vote tiap opsi.

```javascript
await client.sendMessage(jid, {
  pollResultMessage: {
    name: 'Hasil Vote: Warna Favorit',
    pollVotes: [
      { optionName: 'Merah', optionVoteCount: '42' },
      { optionName: 'Hijau', optionVoteCount: '15' },
      { optionName: 'Biru',  optionVoteCount: '88' }
    ]
  }
}, { quoted: m });
```

---

### 15. Product Message

Mengirim pesan katalog produk dengan tombol beli.

```javascript
await client.sendMessage(jid, {
  productMessage: {
    title: 'Nama Produk',
    description: 'Deskripsi produk lengkap',
    thumbnail: { url: 'https://example.com/produk.jpg' },
    productId: 'PROD001',
    retailerId: 'RETAIL001',
    url: 'https://example.com/produk',
    body: 'Detail produk',
    footer: 'Harga spesial hari ini',
    priceAmount1000: 150000, // Harga dalam satuan 1/1000
    currencyCode: 'IDR',
    buttons: [
      {
        name: 'cta_url',
        buttonParamsJson: JSON.stringify({
          display_text: 'Beli Sekarang',
          url: 'https://example.com/beli',
          merchant_url: 'https://example.com'
        })
      }
    ]
  }
}, { quoted: m });
```

---

### 16. Request Payment Message

Mengirim permintaan pembayaran dengan background dan sticker kustom.

```javascript
await client.sendMessage(jid, {
  requestPaymentMessage: {
    currency: 'IDR',
    amount: 150000,
    from: m.sender,
    background: {
      id: '100',
      fileLength: '0',
      width: 1000,
      height: 1000,
      mimetype: 'image/webp',
      placeholderArgb: 0xFF00FFFF,
      textArgb: 0xFFFFFFFF,
      subtextArgb: 0xFFAA00FF
    }
  }
}, { quoted: m });
```

---

### 17. Review and Pay Message

Mengirim pesan pembayaran native WhatsApp dengan rincian order.

```javascript
await client.sendMessage(jid, {
  nativeFlowMessage: {
    buttons: [
      {
        name: 'review_and_pay',
        buttonParamsJson: JSON.stringify({
          currency: 'IDR',
          total_amount: { value: 150000, offset: 100 },
          reference_id: 'ORDER-12345',
          type: 'payment_request',
          payment_status: 'pending',
          payment_timestamp: Date.now(),
          order: {
            status: 'pending',
            subtotal: { value: 150000, offset: 100 },
            order_type: 'PAYMENT_REQUEST',
            items: [
              {
                retailer_id: 'ITEM-' + Math.floor(Math.random() * 1e9),
                name: 'Nama Produk',
                amount: { value: 150000, offset: 100 },
                quantity: 1
              }
            ]
          },
          additional_note: 'Terima kasih telah berbelanja',
          share_payment_status: true
        })
      }
    ]
  }
}, { quoted: m });
```

---

### 18. Group Status Message

Mengirim pesan ke status grup (mirip broadcast ke semua member).

```javascript
const groupId = 'XXXXXXXXXX@g.us'; // JID grup

// Status teks
await client.sendMessage(groupId, {
  groupStatusMessage: {
    text: 'Pengumuman penting untuk semua member!'
  }
});

// Status gambar
const buffer = await quoted.download();
await client.sendMessage(groupId, {
  groupStatusMessage: {
    image: buffer,
    caption: 'Caption gambar status grup'
  }
});

// Status video
await client.sendMessage(groupId, {
  groupStatusMessage: {
    video: buffer,
    caption: 'Caption video status grup'
  }
});

// Status audio
await client.sendMessage(groupId, {
  groupStatusMessage: {
    audio: buffer
  }
});
```

---

### 19. Interactive Message

Pesan interaktif dasar dengan berbagai tipe tombol.

#### Tombol Copy Code
```javascript
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'Header Pesan',
    title: 'Judul Interaktif',
    footer: 'Footer teks',
    buttons: [
      {
        name: 'cta_copy',
        buttonParamsJson: JSON.stringify({
          display_text: 'Salin Kode',
          id: '123456789',
          copy_code: 'DISKON50'
        })
      }
    ]
  }
}, { quoted: m });
```

#### Tombol URL
```javascript
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'Kunjungi Website Kami',
    title: 'Info Lebih Lanjut',
    footer: 'Klik tombol di bawah',
    buttons: [
      {
        name: 'cta_url',
        buttonParamsJson: JSON.stringify({
          display_text: 'Buka Website',
          url: 'https://example.com',
          merchant_url: 'https://example.com'
        })
      }
    ]
  }
}, { quoted: m });
```

#### Tombol Call
```javascript
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'Hubungi Kami',
    title: 'Customer Service',
    footer: '24 jam siap melayani',
    buttons: [
      {
        name: 'cta_call',
        buttonParamsJson: JSON.stringify({
          display_text: 'Telepon Sekarang',
          phone_number: '+628123456789'
        })
      }
    ]
  }
}, { quoted: m });
```

#### Tombol dengan Gambar / Thumbnail
```javascript
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'Promo Spesial',
    title: 'Diskon 50% Hari Ini',
    footer: 'Berlaku sampai tengah malam',
    image: { url: 'https://example.com/promo.jpg' },
    buttons: [
      {
        name: 'cta_url',
        buttonParamsJson: JSON.stringify({
          display_text: 'Klaim Promo',
          url: 'https://example.com/promo'
        })
      },
      {
        name: 'cta_copy',
        buttonParamsJson: JSON.stringify({
          display_text: 'Salin Kode',
          id: '999',
          copy_code: 'PROMO50'
        })
      }
    ]
  }
}, { quoted: m });
```

---

### 20. Interactive Message dengan Native Flow

Pesan interaktif lengkap dengan dropdown list, multi-button, dan konfigurasi lanjutan.

```javascript
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'Menu Utama',
    title: 'Pilih Layanan',
    footer: 'Powered by Bot',
    image: { url: 'https://example.com/banner.jpg' },
    nativeFlowMessage: {
      messageParamsJson: JSON.stringify({
        limited_time_offer: {
          text: 'Penawaran terbatas!',
          url: 'https://example.com',
          copy_code: 'KODE123',
          expiration_time: Date.now() + 86400000
        },
        bottom_sheet: {
          in_thread_buttons_limit: 2,
          divider_indices: [1, 3],
          list_title: 'Pilihan Menu',
          button_title: 'Lihat Menu'
        }
      }),
      buttons: [
        // Dropdown pilihan tunggal
        {
          name: 'single_select',
          buttonParamsJson: JSON.stringify({
            title: 'Pilih Kategori',
            sections: [
              {
                title: 'Layanan Utama',
                highlight_label: 'Populer',
                rows: [
                  { title: 'Layanan A', description: 'Deskripsi layanan A', id: 'LA' },
                  { title: 'Layanan B', description: 'Deskripsi layanan B', id: 'LB' },
                  { title: 'Layanan C', description: 'Deskripsi layanan C', id: 'LC' }
                ]
              },
              {
                title: 'Layanan Tambahan',
                rows: [
                  { title: 'Layanan D', description: 'Deskripsi layanan D', id: 'LD' }
                ]
              }
            ],
            has_multiple_buttons: true
          })
        },
        // Tombol copy
        {
          name: 'cta_copy',
          buttonParamsJson: JSON.stringify({
            display_text: 'Salin Kode Promo',
            id: '001',
            copy_code: 'PROMO2025'
          })
        },
        // Tombol URL
        {
          name: 'cta_url',
          buttonParamsJson: JSON.stringify({
            display_text: 'Kunjungi Website',
            url: 'https://example.com',
            merchant_url: 'https://example.com'
          })
        },
        // Izin panggilan
        {
          name: 'call_permission_request',
          buttonParamsJson: JSON.stringify({
            has_multiple_buttons: true
          })
        }
      ]
    }
  }
}, { quoted: m });
```

---

### 21. Interactive Message dengan Document

Pesan interaktif dengan lampiran dokumen sebagai media header.

> **Penting:** Header dokumen hanya mendukung **buffer**, bukan URL.

```javascript
const fs = require('fs');

// Lengkap dengan contextInfo dan externalAdReply
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'Dokumen Penting',
    title: 'Laporan Bulanan',
    footer: 'Klik untuk membuka',
    document: fs.readFileSync('./laporan.pdf'),
    mimetype: 'application/pdf',
    fileName: 'laporan-bulanan.pdf',
    jpegThumbnail: fs.readFileSync('./thumbnail.jpeg'),
    contextInfo: {
      mentionedJid: [m.chat],
      forwardingScore: 0,
      isForwarded: false
    },
    externalAdReply: {
      title: 'Laporan Penting',
      body: 'Klik untuk lihat detail',
      mediaType: 3,
      thumbnailUrl: 'https://example.com/thumbnail.jpg',
      mediaUrl: 'https://example.com',
      sourceUrl: 'https://example.com',
      showAdAttribution: true,
      renderLargerThumbnail: false
    },
    buttons: [
      {
        name: 'cta_url',
        buttonParamsJson: JSON.stringify({
          display_text: 'Lihat Online',
          url: 'https://example.com/laporan',
          merchant_url: 'https://example.com'
        })
      }
    ]
  }
}, { quoted: m });

// Versi sederhana (tanpa contextInfo/externalAdReply)
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'File PDF',
    title: 'Dokumen',
    footer: 'Unduh dokumen',
    document: fs.readFileSync('./dokumen.pdf'),
    mimetype: 'application/pdf',
    fileName: 'dokumen.pdf',
    jpegThumbnail: fs.readFileSync('./thumb.jpeg'),
    buttons: [
      {
        name: 'cta_url',
        buttonParamsJson: JSON.stringify({
          display_text: 'Download',
          url: 'https://example.com/download',
          merchant_url: 'https://example.com'
        })
      }
    ]
  }
}, { quoted: m });
```

---

### 22. Interactive Message dengan External Ad Reply

Pesan interaktif dengan tampilan preview iklan/artikel eksternal.

```javascript
await client.sendMessage(jid, {
  interactiveMessage: {
    header: 'Artikel Terbaru',
    title: 'Berita Hari Ini',
    footer: 'Baca selengkapnya',
    image: { url: 'https://example.com/artikel.jpg' },
    contextInfo: {
      externalAdReply: {
        title: 'Judul Artikel',
        body: 'Ringkasan artikel singkat',
        mediaType: 1, // 1=IMAGE, 3=VIDEO
        thumbnailUrl: 'https://example.com/thumb.jpg',
        mediaUrl: 'https://example.com/artikel',
        sourceUrl: 'https://example.com',
        showAdAttribution: true,
        renderLargerThumbnail: true
      }
    },
    buttons: [
      {
        name: 'cta_url',
        buttonParamsJson: JSON.stringify({
          display_text: 'Baca Artikel',
          url: 'https://example.com/artikel'
        })
      }
    ]
  }
}, { quoted: m });
```

---

## Manajemen Grup

```javascript
// Buat grup baru
const group = await client.groupCreate('Nama Grup Baru', [
  '628111111111@s.whatsapp.net',
  '628222222222@s.whatsapp.net'
]);
console.log('Grup dibuat:', group.id);

// Tambah member
await client.groupParticipantsUpdate(groupId, ['628333333333@s.whatsapp.net'], 'add');

// Keluarkan member
await client.groupParticipantsUpdate(groupId, ['628333333333@s.whatsapp.net'], 'remove');

// Jadikan admin
await client.groupParticipantsUpdate(groupId, ['628111111111@s.whatsapp.net'], 'promote');

// Copot admin
await client.groupParticipantsUpdate(groupId, ['628111111111@s.whatsapp.net'], 'demote');

// Update deskripsi grup
await client.groupUpdateDescription(groupId, 'Deskripsi grup baru');

// Update nama/subject grup
await client.groupUpdateSubject(groupId, 'Nama Grup Baru');

// Ubah foto grup
await client.updateProfilePicture(groupId, fs.readFileSync('./foto-grup.jpg'));

// Dapatkan info grup
const groupMetadata = await client.groupMetadata(groupId);
console.log(groupMetadata);

// Dapatkan invite link
const inviteCode = await client.groupInviteCode(groupId);
console.log('Link:', `https://chat.whatsapp.com/${inviteCode}`);

// Reset invite link
await client.groupRevokeInvite(groupId);

// Keluar dari grup
await client.groupLeave(groupId);

// Atur hanya admin yang bisa kirim pesan
await client.groupSettingUpdate(groupId, 'announcement');

// Kembalikan semua member bisa kirim pesan
await client.groupSettingUpdate(groupId, 'not_announcement');

// Kunci info grup (hanya admin yang bisa edit)
await client.groupSettingUpdate(groupId, 'locked');
await client.groupSettingUpdate(groupId, 'unlocked');
```

---

## Manajemen Chat & Kontak

```javascript
// Hapus pesan
await client.sendMessage(jid, { delete: m.key });

// Edit pesan (teks saja)
await client.sendMessage(jid, {
  text: 'Teks yang sudah diedit',
  edit: m.key
});

// Archive chat
await client.chatModify({ archive: true }, jid);
await client.chatModify({ archive: false }, jid);

// Pin chat
await client.chatModify({ pin: true }, jid);
await client.chatModify({ pin: false }, jid);

// Tandai sudah dibaca
await client.readMessages([m.key]);

// Mute chat (dalam ms, 86400000 = 24 jam)
await client.chatModify({ mute: 86400000 }, jid);
await client.chatModify({ mute: null }, jid); // unmute

// Update nama profil
await client.updateProfileName('Nama Baru');

// Update status/bio
await client.updateProfileStatus('Status baru saya');

// Update foto profil
await client.updateProfilePicture(client.user.id, fs.readFileSync('./foto.jpg'));

// Hapus foto profil
await client.removeProfilePicture(client.user.id);

// Cek apakah nomor terdaftar di WA
const result = await client.onWhatsApp('628123456789');
console.log(result);

// Dapatkan foto profil
const ppUrl = await client.profilePictureUrl('628123456789@s.whatsapp.net', 'image');
console.log(ppUrl);

// Blokir kontak
await client.updateBlockStatus('628123456789@s.whatsapp.net', 'block');
await client.updateBlockStatus('628123456789@s.whatsapp.net', 'unblock');
```

---

## Fitur Presence & Status

```javascript
// Update presence di chat
await client.sendPresenceUpdate('available', jid);       // Online
await client.sendPresenceUpdate('unavailable', jid);     // Offline
await client.sendPresenceUpdate('composing', jid);       // Sedang mengetik...
await client.sendPresenceUpdate('recording', jid);       // Merekam audio...
await client.sendPresenceUpdate('paused', jid);          // Berhenti mengetik

// Subscribe untuk menerima presence update kontak
await client.presenceSubscribe(jid);

// Kirim pesan ke status WA (Story)
await client.sendMessage('status@broadcast', {
  text: 'Status baru saya!',
  backgroundColor: '#FF5733',
  font: 3
});

await client.sendMessage('status@broadcast', {
  image: { url: 'https://example.com/foto.jpg' },
  caption: 'Caption status',
  backgroundColor: '#000000'
});
```

---

## Manajemen Sesi & Store

### Setup Session dengan File

```javascript
const { useMultiFileAuthState } = require('@sairidev/baileys2');

const { state, saveCreds } = await useMultiFileAuthState('./sesi');
// state.creds berisi credentials
// state.keys berisi encryption keys
// Panggil saveCreds() setiap kali ada update
client.ev.on('creds.update', saveCreds);
```

### In-Memory Store

```javascript
const { makeInMemoryStore } = require('@sairidev/baileys2');
const store = makeInMemoryStore({});

// Simpan ke file setiap 10 detik
store.readFromFile('./baileys_store.json');
setInterval(() => store.writeToFile('./baileys_store.json'), 10_000);

// Bind ke socket
store.bind(client.ev);

// Akses data yang tersimpan
const chats = store.chats.all();
const contacts = store.contacts;
const messages = store.messages;
```

---

## Event System

```javascript
// Koneksi update
client.ev.on('connection.update', (update) => {
  console.log('Status koneksi:', update.connection);
  console.log('QR Code:', update.qr);
});

// Pesan masuk / keluar
client.ev.on('messages.upsert', ({ messages, type }) => {
  for (const msg of messages) {
    if (!msg.message) continue;
    console.log('Pesan baru:', msg);
  }
});

// Pesan diupdate (dibaca, dikirim, dll)
client.ev.on('messages.update', (updates) => {
  for (const update of updates) {
    console.log('Update pesan:', update.key, update.update);
  }
});

// Pesan dihapus
client.ev.on('messages.delete', (item) => {
  console.log('Pesan dihapus:', item);
});

// Chat update
client.ev.on('chats.upsert', (chats) => {
  console.log('Chat baru/update:', chats);
});

client.ev.on('chats.update', (updates) => {
  console.log('Update chat:', updates);
});

// Kontak update
client.ev.on('contacts.upsert', (contacts) => {
  console.log('Kontak baru/update:', contacts);
});

// Group update
client.ev.on('groups.upsert', (groups) => {
  console.log('Grup baru:', groups);
});

client.ev.on('groups.update', (updates) => {
  console.log('Update grup:', updates);
});

// Member masuk/keluar grup
client.ev.on('group-participants.update', ({ id, participants, action }) => {
  console.log(`Grup ${id}: ${participants} => ${action}`);
  // action: 'add' | 'remove' | 'promote' | 'demote'
});

// Presence update
client.ev.on('presence.update', ({ id, presences }) => {
  console.log('Presence update:', id, presences);
});

// Credentials update (harus disimpan!)
client.ev.on('creds.update', saveCreds);

// Call update
client.ev.on('call', (calls) => {
  for (const call of calls) {
    console.log('Panggilan masuk:', call.from, call.status);
  }
});
```

---

## Download Media

```javascript
const { downloadMediaMessage } = require('@sairidev/baileys2');

client.ev.on('messages.upsert', async ({ messages }) => {
  const msg = messages[0];
  if (!msg.message) return;

  const type = Object.keys(msg.message)[0];

  if (['imageMessage', 'videoMessage', 'audioMessage', 'documentMessage', 'stickerMessage'].includes(type)) {
    const buffer = await downloadMediaMessage(
      msg,
      'buffer',
      {},
      {
        logger,
        reuploadRequest: client.updateMediaMessage
      }
    );

    fs.writeFileSync(`./download.${type === 'imageMessage' ? 'jpg' : 'mp4'}`, buffer);
    console.log('Media berhasil diunduh!');
  }
});
```

---

## Dependencies

| Package | Versi | Fungsi |
|---------|-------|--------|
| `ws` | ^8.13.0 | WebSocket client — koneksi ke server WA |
| `protobufjs` | ^7.2.4 | Parse protokol protobuf WhatsApp |
| `libsignal` (`@dapadev/libsignal-node`) | latest | Signal Protocol — enkripsi E2E |
| `music-metadata` | ^7.12.3 | Ekstrak metadata file audio |
| `audio-decode` | ^2.1.3 | Decode file audio |
| `libphonenumber-js` | ^1.10.20 | Validasi & parsing nomor telepon |
| `axios` | ^1.3.3 | HTTP requests |
| `node-fetch` | ^2.6.1 | Fetch API untuk Node.js |
| `pino` | ^7.0.0 | Logger performa tinggi |
| `cache-manager` | 4.0.1 | Manajemen cache |
| `node-cache` | ^5.1.2 | In-memory cache |
| `@cacheable/node-cache` | ^1.4.0 | Cache layer tambahan |
| `async-mutex` | ^0.5.0 | Mutex untuk mencegah race condition |
| `futoin-hkdf` | ^1.5.1 | Derivasi kunci kriptografi (HKDF) |
| `uuid` | ^9.0.0 | Generate ID unik |
| `lodash` | ^4.17.21 | Utility functions |
| `chalk` | ^4.1.2 | Output terminal berwarna |
| `@adiwajshing/keyed-db` | ^0.2.4 | Database berorientasi kunci |
| `@hapi/boom` | ^9.1.3 | HTTP-friendly error handling |

### Peer Dependencies (Opsional)

| Package | Fungsi |
|---------|--------|
| `sharp` | Resize & manipulasi gambar (performa tinggi) |
| `jimp` | Manipulasi gambar (pure JS) |
| `link-preview-js` | Generate preview untuk URL link |
| `qrcode-terminal` | Tampilkan QR Code di terminal |

---

## Catatan Teknis

- Library ini adalah **fork modifikasi** dari [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys), bukan produk resmi WhatsApp.
- Menggunakan **WebSocket** langsung — tidak membutuhkan browser atau Selenium.
- Semua komunikasi dienkripsi end-to-end menggunakan **Signal Protocol**.
- Media tidak pernah di-load sepenuhnya ke memori — diproses sebagai **readable stream** terenkripsi.
- Kompatibel dengan arsitektur **multi-device** WhatsApp terbaru.
- Pairing code dapat dikustomisasi untuk proses autentikasi yang lebih stabil.
- Untuk produksi, disarankan menggunakan **process manager** seperti PM2 agar koneksi tetap hidup.
- Pastikan mematuhi [Ketentuan Layanan WhatsApp](https://www.whatsapp.com/legal/terms-of-service). Library ini tidak untuk spam atau penyalahgunaan.

---

## Lisensi

MIT License — © [sairidev](https://github.com/sairidev)
