> Read this in English: [README.md](README.md)

# Qssentials

Addon *server tool* untuk Minecraft Bedrock Edition: ekonomi, home, warp, TPA, land claim, reward harian/mingguan, scorehud, rank, chat custom, dan banyak Custom UI. Cocok untuk server SMP/Realm.

| | |
|---|---|
| **Paket** | `Qssentials [BP]` + `Qssentials [RP]` (satu file `.mcaddon`) |
| **Versi** | v2.0.0 (build 241026) |
| **Namespace perintah** | `es` (contoh: `/es:home`) |
| **Min. engine** | 1.26.60.4 |
| **Modul Script** | `@minecraft/server` (beta), `@minecraft/server-ui` (beta), `@minecraft/server-admin` (1.0.0-beta) |
| **Bahasa bawaan** | English (`en_US`) dan Indonesia (`id_ID`) |

---

## Daftar Isi

1. [Fitur](#fitur)
2. [Persyaratan](#persyaratan)
3. [Instalasi](#instalasi)
4. [Pengaturan Awal](#pengaturan-awal)
5. [Daftar Perintah](#daftar-perintah)
6. [Rank dan Izin Akses](#rank-dan-izin-akses)
7. [Konfigurasi Utama (`config.js`)](#konfigurasi-utama-configjs)
8. [Konfigurasi Per-Modul](#konfigurasi-per-modul)
9. [Panduan Fitur](#panduan-fitur)
10. [Resource Pack dan Tampilan UI](#resource-pack-dan-tampilan-ui)
11. [ScriptEvent dan Integrasi](#scriptevent-dan-integrasi)
12. [Struktur Folder](#struktur-folder)
13. [Troubleshooting](#troubleshooting)
14. [Changelog](#changelog)
15. [Kontributor dan Lisensi](#kontributor-dan-lisensi)

---

## Fitur

- **Teleportasi**: `/home`, `/sethome`, `/delhome`, `/warp`, `/setwarp`, `/tpa`, `/rtp`, `/back` (kembali ke lokasi mati terakhir). Teleport dibatalkan jika pemain bergerak saat hitung mundur.
- **Ekonomi**: saldo `money`, `/pay`, `/topmoney`, `/setmoney`.
- **Land claim**: beli lahan dengan klik dua titik, proteksi build/interaksi/ledakan/farmland, jual kembali lahan.
- **Reward**: `/daily` (per hari dalam seminggu) dan `/weekly` (tier `free`, `rare`, `mythical`) dengan hadiah item, uang, atau acak.
- **Scorehud**: papan info di layar, template bisa diubah lewat config atau perintah.
- **Rank & chat**: rank berbasis tag, format chat dan nametag custom, cooldown chat.
- **Sistem server**: ClearLag + ClearChat otomatis, broadcast otomatis, lobby protect, interaksi NPC satu klik.
- **Custom UI**: Home, Warp, Ranklist, Rules, konfirmasi land, toast notifikasi, dan lainnya.

---

## Persyaratan

- Minecraft Bedrock **1.21.100** atau lebih baru.
- Eksperimen **Beta APIs** aktif di world/server (manifest memakai modul beta).
- **Behavior Pack dan Resource Pack harus aktif keduanya** (BP bergantung pada RP).
- **Cheats aktif** di world selama `command_required_cheats: true` (default). Lihat [Konfigurasi Utama](#konfigurasi-utama-configjs).
- Addon ini bergantung pada modul `@minecraft/server-admin` (dipakai oleh `/goto`). Modul tersebut pada umumnya hanya tersedia di Bedrock Dedicated Server, jadi uji coba di world lokal bisa berbeda dari server.

---

## Instalasi

### World lokal / Realm

1. Buka file `.mcaddon` dengan Minecraft. `Qssentials [BP]` dan `Qssentials [RP]` akan terimpor otomatis.
2. Buat atau edit world, lalu di bagian **Behavior Packs** dan **Resource Packs** aktifkan **Qssentials**.
3. Di bagian **Experiments** aktifkan **Beta APIs**.
4. Aktifkan **Allow Cheats** (atau ubah `command_required_cheats` menjadi `false`).
5. Masuk ke world. Pesan `[Qssentials] >> World Has Been Load...` menandakan addon sudah berjalan.

### Dedicated Server (BDS)

1. Ekstrak `.mcaddon` (format ZIP), salin folder `Qssentials [BP]` ke `development_behavior_packs/` dan `Qssentials [RP]` ke `development_resource_packs/`.
2. Daftarkan keduanya pada `world_behavior_packs.json` dan `world_resource_packs.json` milik world Anda (UUID ada di `manifest.json` masing-masing).
3. Aktifkan beta API pada world, lalu jalankan ulang server.

> Setelah mengubah `config.js` atau file `ess.config.js`, simpan dan **muat ulang world/server** agar perubahan berlaku.

---

## Pengaturan Awal

1. **Tentukan Owner**: buka `Qssentials [BP]/scripts/config.js` dan isi `owners` dengan gamertag persis (huruf besar/kecil berpengaruh):

   ```js
   owners: ["NamaOwnerKamu", "NamaOwnerLain"],
   ```

2. **Tentukan Admin**: pemain dianggap *Admin* jika Owner, Operator (`/op`), atau memiliki tag `Admin`:

   ```mcfunction
   /tag NamaPemain add Admin
   ```

3. **Atur tautan beli rank** (tombol "contact" di `/ranklist`) pada `buy_rank_link`.
4. **Atur lobby** (opsional): lihat [Lobby Protect](#lobby-protect).
5. **Buat warp pertama**: berdiri di lokasi tujuan lalu jalankan `/setwarp spawn`.

Saat pertama dijalankan, addon otomatis membuat scoreboard `money`, `kills`, dan `deaths`.

---

## Daftar Perintah

Semua perintah terdaftar dengan namespace `es:` (misalnya `/es:home`). Pada tabel di bawah awalan `es:` dihilangkan agar ringkas; tulis lengkap `/es:nama` bila perintah bentrok dengan addon lain. Namespace dapat diubah lewat `config.namespace`.

Notasi: `<param>` = wajib, `[param]` = opsional.

**Level akses**: *Semua* = semua pemain · *Tag* = butuh tag tertentu · *Admin* = Owner / Operator / tag `Admin` · *Owner* = gamertag di `config.owners`.

### General

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/help` | `[category]` | Semua | Daftar perintah. Kategori: `general`, `tpa`, `home`, `mode`, `economy`, `owner`. |
| `/about` | | Semua | Info namespace, versi, dan kontributor. |
| `/plugins` (alias `/pl`, `/plugin`) | `[pluginName]` | Admin* | Daftar plugin terpasang. *Dibatasi untuk admin jika `hide_plugin_command_for_member: true`. |
| `/rules` | | Semua | Menampilkan aturan server (dari `config.rules`). |
| `/news` | | Semua | Menampilkan berita (dari `config.news`). |
| `/ranklist` | | Semua | UI daftar rank dan fiturnya. |
| `/ping` | | Semua | Melihat ping Anda. |
| `/tps` | | Semua | Estimasi TPS server. |
| `/iditem` | | Semua | Melihat ID item di tangan. |
| `/clearchat` | | Semua | Membersihkan chat milik sendiri (cooldown 30 dtk). |
| `/scorehud` | | Semua | Menyalakan/mematikan scorehud pribadi. |
| `/goto` | `<ip-address> [port]` | Semua | Pindah ke server lain (port bawaan `19132`). |
| `/rtp` | `[dimension]` | Semua | Teleport acak. Dimensi: `overworld`, `nether`, `the_end`. |
| `/back` | | Semua | Kembali ke lokasi mati terakhir. |
| `/feed` | | Tag `feed.access` / `dev` | Mengisi lapar penuh (cooldown 10 dtk). |
| `/heal` | | Tag `heal.access` / `dev` | Mengisi darah penuh (cooldown 30 dtk). |
| `/fly` | | Tag `fly.access` | Mengaktifkan kemampuan terbang (`mayfly`). |

### Home, Warp, dan TPA

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/home` | `[homeName]` | Semua | Tanpa parameter membuka UI daftar home. Dengan nama = teleport langsung. |
| `/sethome` | `[homeName]` | Semua | Menyimpan home di posisi sekarang. Tanpa nama membuka form. |
| `/delhome` | `[homeName]` | Semua | Menghapus home. Tanpa nama membuka UI + konfirmasi. |
| `/warp` | `[warpName]` | Semua | Tanpa parameter membuka UI warp (urut A–Z atau terdekat). |
| `/setwarp` | `<warpName>` | Admin | Menyimpan warp di posisi sekarang (nama otomatis huruf kecil). |
| `/tpa` | `<TargetPlayer>` | Semua | Mengirim permintaan teleport ke pemain lain. |
| `/tpaaccept` | `[player]` | Semua | Menerima permintaan (pengirim akan diteleport ke Anda). |
| `/tpadeny` | | Semua | Menolak permintaan. |

### Land

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/land here` | | Semua | Cek apakah posisi Anda berada di lahan seseorang. |
| `/land create` | | Semua | Mulai membeli lahan: klik blok titik 1 lalu titik 2, lalu konfirmasi. |
| `/land list` | | Semua | Daftar lahan milik Anda (ID, koordinat tengah, dimensi). |
| `/land sell` | `[id]` | Semua | Menjual lahan. Berdiri di dalam lahan Anda agar ID terdeteksi otomatis. |

### Ekonomi

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/money` (alias `/balance`) | | Semua | Melihat saldo. |
| `/pay` | `<TargetPlayer> <Amount>` | Semua | Mengirim uang (cooldown 5 dtk, minimum dari `pay.min`). |
| `/topmoney` | | Semua | Papan peringkat saldo. |
| `/setmoney` (alias `/setbalance`) | `<Amount> [TargetPlayer]` | Admin | Mengatur saldo. |

### Reward

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/daily` | | Semua | Klaim reward sesuai hari ini (cooldown 24 jam). |
| `/weekly` | `[free\|rare\|mythical]` | Semua / Tag | Klaim reward mingguan (cooldown 7 hari per tier). `rare` butuh tag `weekly.rare`, `mythical` butuh tag `weekly.myth`. |

### Mode

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/gms` | | Semua | Mode survival. |
| `/gmc` | | Admin | Mode creative. |
| `/spectator` | | Admin | Mode spectator. |
| `/invis` (alias `/ghost`) | | Tag `mvp` / `admin` / `dev` / `owner` | Menyalakan/mematikan invisibility. |

### Admin

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/ctitle` | `<title-text> [subtitle-text] [broadcast]` | Admin | Menampilkan title custom. `\n` = baris baru, `@s` = nama pemain. `broadcast=true` untuk semua pemain. |
| `/invsee` | `<target>` | Admin | Melihat inventori pemain lain lewat UI chest. |
| `/setscorehud` | `[title] [body] [footer]` | Admin | Mengubah template scorehud. Gunakan `@ln` untuk baris baru. Kosongkan `title`/`body` untuk reset ke template config. |
| `/cdialog` | | Admin | **Nonaktif secara default.** Butuh plugin `DynamicDialogV2`. |

### Owner

| Perintah | Parameter | Akses | Keterangan |
|---|---|---|---|
| `/sethomelimit` | `[limit] [targetName]` | Owner | Mengatur batas home per pemain. `-1` = tanpa batas. |
| `/clearrank` | `[Target]` | Owner | Menghapus semua tag rank milik target (atau diri sendiri). |
| `/dailyreset` | | Owner | Reset cooldown daily milik sendiri (untuk pengujian). |
| `/weeklyreset` | | Owner | Reset cooldown weekly milik sendiri (untuk pengujian). |
| `/notif` | `[title] [message] [icon] [background] [broadcast]` | Owner | Mengirim toast notifikasi custom. `icon` bisa path tekstur atau `aux:<id>`. |

---

## Rank dan Izin Akses

Qssentials memakai **tag pemain** untuk rank dan izin.

### Memberi rank (tampilan chat dan nametag)

Rank dibaca dari tag berawalan `rank:` (bisa diubah di `rank_prefix`). Tag terakhir yang dimiliki pemain dipakai sebagai rank aktif.

```mcfunction
/tag NamaPemain add rank:VIP
/tag NamaPemain add "rank:§bVIP"
```

- Teks setelah `rank:` ditampilkan apa adanya (kode warna `§` boleh dipakai) dan dibungkus tanda kurung, misalnya `[VIP]`.
- Jika rank berupa karakter glyph (mis. dari `font/glyph_E*.png`), rank ditampilkan tanpa tanda kurung.
- Tanpa tag rank, nilai `rank_default` dipakai.
- Hapus rank: `/clearrank <pemain>` (Owner).

> Isi `ranklist` di `config.js` hanya dipakai untuk **tampilan** UI `/ranklist`. Hak akses perintah ditentukan oleh tag di bawah.

### Tag izin

| Tag | Efek |
|---|---|
| `Admin` | Dianggap Admin (setara OP untuk fitur Qssentials). |
| `feed.access` | Boleh memakai `/feed`. |
| `heal.access` | Boleh memakai `/heal`. |
| `fly.access` | Boleh memakai `/fly`. |
| `mvp` | Boleh memakai `/invis`. |
| `dev` | Boleh memakai `/feed`, `/heal`, `/invis`. |
| `weekly.rare` | Boleh klaim `/weekly rare`. |
| `weekly.myth` | Boleh klaim `/weekly mythical`. |
| `disable:scorehud` | Dibuat otomatis saat pemain mematikan scorehud. |
| `daily.cd.*`, `weekly.cd.*` | Dibuat otomatis untuk menyimpan cooldown reward. Jangan dihapus manual. |

Contoh memberi akses ala VIP/MVP:

```mcfunction
/tag NamaPemain add rank:VIP
/tag NamaPemain add feed.access
/tag NamaPemain add heal.access
/tag NamaPemain add fly.access
```

---

## Konfigurasi Utama (`config.js`)

Lokasi: `Qssentials [BP]/scripts/config.js`

### Umum

| Opsi | Default | Keterangan |
|---|---|---|
| `namespace` | `"es"` | Awalan perintah (`/es:home`). |
| `language` | `"en"` | Bahasa bawaan (saat ini belum dipakai). |
| `owners` | – | Daftar gamertag Owner. |
| `command_required_cheats` | `true` | Jika `true`, perintah hanya berjalan saat cheats aktif. Ubah ke `false` agar jalan tanpa cheats. |
| `rank_prefix` | `"rank:"` | Awalan tag rank. |
| `rank_default` | `"§7"` | Tampilan rank untuk pemain tanpa tag rank. |
| `buy_rank_link` | – | Tautan pada tombol "contact" di `/ranklist`. |
| `ranklist` | – | Isi tampilan `/ranklist`: `{ NAMA: { icon, feature: [...] } }`. |
| `news` | `[...]` | Array teks untuk `/news` (satu elemen = satu baris). |
| `hide_plugin_command_for_member` | `true` | `true` = `/plugins` hanya untuk Admin. |
| `teleport_show_dynamic_countdown` | `true` | Tampilkan hitung mundur teleport di actionbar. |
| `pay.min` | `1000` | Minimal `/pay`. Nilai dibatasi di rentang **500–20000**. |

### Chat dan Nametag

```js
chat: {
  message_max_length: 500,   // panjang maksimal pesan
  cooldown: 3,               // detik, jeda antar pesan
  message_template: "@rank @player_name§r @faction §7§l>§r @msg"
},
nametag: {
  template: "@rank§r @faction §r@player_name",
  enabled: true              // false = nametag bawaan Minecraft
},
```

| Placeholder | Chat | Nametag |
|---|:--:|:--:|
| `@rank` | ✔ | ✔ |
| `@player_name` | ✔ | ✔ |
| `@faction` | ✔ | ✔ |
| `@money` | ✔ | ✔ |
| `@platform` | – | ✔ |
| `@msg` | ✔ | – |

> Fitur faction belum memiliki perintah di build ini. Opsi `faction.color` hanya memengaruhi tampilan `@faction`; opsi `min`, `max`, `maximum_member`, dan `enabled` belum dipakai.

### Scorehud

```js
scorehud: {
  enable_by_default: true,
  template_default: `textures/ui/title|@ln§l§6info§r@ln§arank: @rank|QssentialsV2`
}
```

Format `template_default`: **`TITLE|BODY|WATERMARK`**

- `TITLE`: teks, atau path tekstur (mis. `textures/ui/title`).
- `BODY`: isi papan. Pakai `@ln` untuk baris baru dan kode warna `§`.
- `WATERMARK`: teks kecil di bagian footer.

Placeholder yang tersedia di `BODY`:

`@player_name` `@rank` `@player_gamemode` `@platform` `@faction` `@player_velocity` `@x` `@y` `@z` `@player_dimension` `@money` `@moneyr` (uang singkat, mis. `1.50k`) `@kills` `@deaths` `@land_here` (ID lahan tempat berdiri) `@durability_mainhand_current` `@durability_mainhand_max` `@ln` (baris baru) `@n`

Pengubahan saat server berjalan (tersimpan di database, mengalahkan `template_default`):

```mcfunction
/setscorehud "textures/ui/title" "@ln§6Halo @player_name@lnUang: @moneyr" "MyServer"
```

Jalankan `/setscorehud` tanpa isi untuk kembali ke template config. Pemain dapat mematikan scorehud miliknya dengan `/scorehud`.

### ClearLag dan ClearChat

```js
clear: {
  announcement: [300, 180, 60, 10, 5, 3, 2, 1], // detik sebelum pembersihan
  wait: 20,                                      // jeda antar siklus (detik)
  chat: true,                                    // ikut membersihkan chat
  actived: true                                  // false = matikan sistem
},
```

- Hitung mundur dimulai dari angka pertama (`300` dtk = 5 menit) dan diumumkan pada detik yang tercantum.
- Yang dihapus: entitas **item** dan **arrow** di semua dimensi.
- Jika `chat: true`, siklus bergantian: pembersihan entitas, lalu pembersihan chat.

### RTP

```js
rtp: {
  radius: 2000,   // jarak maksimal dari koordinat 0,0
  countdown: 5,   // detik sebelum teleport
  cooldown: 60    // detik antar pemakaian
},
```

Selama pencarian lokasi, gerakan pemain dikunci dan progres tampil di actionbar. Jika pemain keluar saat proses, ia dikembalikan ke posisi awal ketika masuk lagi.

### Lobby Protect

```js
lobby_protect: {
  enabled: true,
  pos: [{ x: 5, z: 5 }, { x: -5, z: -5 }],  // dua sudut area (sumbu X dan Z)
  allowInteractNpc: true,
  disable_pvp: true
},
```

- Area lobby berlaku di **Overworld**, semua ketinggian Y.
- Pemain biasa tidak bisa membongkar atau berinteraksi di area ini. Admin dan Owner dikecualikan.
- Lahan (land) tidak bisa dibeli di dalam area lobby.
- Opsi `allowInteractNpc` dan `disable_pvp` belum dipakai di build ini. Interaksi dengan NPC diatur lewat `allowedInteractEntities` di [konfigurasi land](#land-1).

### Interaksi NPC / Satu Klik

Beri tag pada NPC/entitas agar klik atau pukulan menjalankan perintah:

```js
interaction: {
  "npc.daily":  { command: "daily",  hit: true,  interact: true },
  "npc.rtp":    { command: "rtp",    hit: false, interact: true },
  // tambahkan sendiri:
  "npc.shop":   { command: "warp shop", hit: false, interact: true }
},
```

| Kunci | Fungsi |
|---|---|
| nama kunci (`npc.daily`) | Tag yang harus dimiliki entitas. |
| `command` | Perintah (tanpa `/`) yang dijalankan oleh pemain. |
| `hit` | Aktif saat entitas dipukul. |
| `interact` | Aktif saat entitas diklik/diinteraksi. |

Memberi tag pada NPC terdekat:

```mcfunction
/tag @e[type=npc,r=3,c=1] add npc.daily
```

Tag bawaan: `npc.daily`, `npc.weekly`, `npc.rtp`, `npc.rules`, `npc.news`, `npc.warp`.

### Broadcast Otomatis

```js
broadcast: {
  title: "§cbroadcast",
  auto_message: {
    enabled: true,
    delay_per_index: 250,   // detik antar pesan
    message: [
      "dont forgot your daily reward, type §e/daily",
      "§aonline now @online/@max_players"
    ]
  }
}
```

- Pesan dikirim bergiliran sesuai urutan array, lalu mengulang dari awal.
- `@online` = jumlah pemain online. `@max_players` = jumlah pemain yang tercatat di database addon.
- Opsi `cooldown`, `price_limit`, `message_max_length`, dan `limit_*` pada `broadcast` belum dipakai di build ini.

### Rules

```js
rules: [
  { text: "Teks pembuka aturan (tampil sebagai body)", texture: "" },
  { text: "GRIEF§c§c§6§t§t\n§r§7No property destruction", texture: "textures/ui/icon_recipe_construction" },
],
```

- Elemen **pertama** menjadi isi utama (body) form.
- Elemen berikutnya tampil sebagai baris aturan dengan ikon dari `texture`.
- `texture` dipotong maksimal 200 karakter, kosongkan jika tanpa ikon.

### Daily dan Weekly Reward

#### Daily

Kunci `0`–`6` mengikuti hari menurut `Date.getDay()` dan **jam server**: `0 = Minggu`, `1 = Senin`, `2 = Selasa`, `3 = Rabu`, `4 = Kamis`, `5 = Jumat`, `6 = Sabtu`. Pemain bisa klaim sekali per hari, dengan cooldown 24 jam.

```js
daily: {
  1: [{ type: "currency", value: 2000 }],
  2: [{ type: "item", item_type: "minecraft:iron_sword", item_name: "§bDaily Sword",
        enchants: [{ id: "sharpness", level: 2 }] }],
  // ...
},
```

#### Weekly

Tiap tier memiliki `prize` dan opsional `permission` (tag yang dibutuhkan). Cooldown 7 hari per tier. Tier baru yang ditambahkan ke `weekly` otomatis muncul sebagai pilihan di `/weekly`.

```js
weekly: {
  free:   { prize: [ /* ... */ ] },
  rare:   { permission: "weekly.rare", prize: [ /* ... */ ] },
  mythical: { permission: "weekly.myth", prize: [ /* ... */ ] }
}
```

#### Format hadiah (`prize`)

| `type` | Properti | Keterangan |
|---|---|---|
| `"item"` | `item_type`, `amount`, `item_name`, `enchants` | Memberi item. |
| `"currency"` | `value` | Menambah uang. |
| `"random"` | `item: [ ...hadiah ]` | Memilih **satu** hadiah acak dari daftar. |

Aturan nilai:

- `amount` / `value`: angka tetap (`5`) atau rentang acak `[min, max]` (`[1, 3]`). `amount` bawaan = `1`.
- `item_type`: satu ID item atau array ID (salah satu dipilih acak).
- `enchants`: array berisi
  - `{ id: "sharpness", level: 2 }`: enchant tertentu,
  - `{ id: "efficiency", level: [1, "max"] }`: level acak sampai level maksimal,
  - `{ id: "random", level: [1, 3] }`: enchant acak dengan level acak,
  - `"random"`: enchant acak sepenuhnya (tidak duplikat dalam satu item).

Contoh lengkap:

```js
{
  type: "random",
  item: [
    { type: "currency", value: [1000, 5000] },
    {
      type: "item",
      item_type: ["minecraft:iron_pickaxe", "minecraft:diamond_pickaxe"],
      item_name: "§bPickaxe Spesial",
      enchants: [{ id: "efficiency", level: [1, 2] }, "random"]
    }
  ]
}
```

---

## Konfigurasi Per-Modul

Beberapa fitur memiliki file `ess.config.js` sendiri di folder `Qssentials [BP]/scripts/commands/<modul>/`.

### Home

`commands/home/ess.config.js`

| Opsi | Default | Keterangan |
|---|---|---|
| `home.min` | `3` | Panjang minimal nama home. |
| `home.max` | `20` | Panjang maksimal nama home. |
| `home.limit_default` | `1` | Batas home pemain biasa (`0`–`99`). |
| `home.limit_admin` | `10` | Batas home Admin. |
| `home.limit_owner` | `'unli'` | Batas home Owner (`'unli'` = tanpa batas). |

Batas per pemain dapat ditimpa dengan `/sethomelimit <limit> <nama>` (Owner).

### Land

`commands/land/ess.config.js`

| Opsi | Default | Keterangan |
|---|---|---|
| `land.pricePerBlock` | `500` | Harga per blok. Dibatasi **100–20000**. |
| `land.percentSell` | `20` | Persen dari harga beli yang diterima saat menjual lahan (`1`–`100`). |
| `land.minimumSize` | `5` | Luas minimum lahan dalam blok (`0`–`2000`). |
| `land.limitLandperPlayer` | `null` | Batas jumlah lahan per pemain. `null` = tanpa batas. |
| `land.cooldown_message_farm_trample` | `1` | Jeda (detik) pesan peringatan menginjak farmland. |
| `land.allowedInteractEntities` | `['minecraft:npc']` | Daftar entitas yang tetap boleh diinteraksi di dalam lahan orang lain/lobby. |

### TPA

`commands/tpa/ess.config.js`

| Opsi | Default | Keterangan |
|---|---|---|
| `countdown` | `5` | Hitung mundur teleport setelah diterima (detik). |
| `expiredTime` | `20` | Masa berlaku permintaan (detik). |

### Warp

`commands/warp/ess.config.js` berisi `cooldown` dan `countdown`. Pada build ini file tersebut belum dibaca oleh `warp.js`; hitung mundur warp memakai nilai bawaan 3 detik.

---

## Panduan Fitur

### Home

1. `/sethome rumah` menyimpan lokasi (atau `/sethome` untuk form).
2. `/home` membuka UI daftar home beserta koordinat dan dimensi, atau `/home rumah` untuk langsung teleport.
3. `/delhome` membuka UI hapus dengan konfirmasi.

Nama home disimpan dalam huruf kecil.

### Warp

1. Admin membuat warp: `/setwarp spawn`, `/setwarp pasar`.
2. Pemain membuka `/warp` → UI daftar dengan label dimensi dan jarak, bisa diurutkan A–Z atau terdekat. Pilih warp → **TELEPORT**.
3. Atau langsung: `/warp spawn`.

### TPA

1. A: `/tpa B`.
2. B: `/tpaaccept` (atau `/tpaaccept A`) untuk menerima, `/tpadeny` untuk menolak.
3. Setelah diterima, A diteleport ke B setelah hitung mundur. Permintaan kedaluwarsa sesuai `expiredTime`.

> Semua teleport yang memakai hitung mundur (home, warp, back, tpa, rtp) **dibatalkan bila pemain bergerak**.

### Land Claim

1. Siapkan uang yang cukup: harga = **luas × `pricePerBlock`**.
2. Jalankan `/land create`, lalu **klik blok** titik pertama dan **klik blok** titik kedua. Lahan berbentuk persegi panjang pada sumbu X dan Z dan mencakup semua ketinggian Y.
3. Konfirmasi pembelian di form. Lahan mendapat ID 4 digit.
4. Gunakan `/land here` untuk memeriksa, `/land list` untuk melihat daftar, dan `/land sell` (sambil berdiri di dalam lahan) untuk menjual kembali dengan nilai `percentSell`%.

Perlindungan di dalam lahan milik pemain lain (dan lobby):

- Tidak bisa membongkar blok.
- Tidak bisa berinteraksi dengan blok/entitas (kecuali entitas pada `allowedInteractEntities`).
- Tidak bisa memukul entitas non-pemain (mis. hewan ternak).
- Ledakan tidak merusak blok di dalam lahan.
- Farmland tidak bisa diinjak hingga rusak.

### Ekonomi

Saldo disimpan pada scoreboard `money`. Selain lewat `/pay`, saldo dapat diubah addon lain melalui [ScriptEvent](#scriptevent-dan-integrasi). Reward `/daily` dan `/weekly` juga dapat memberi uang.

### Kill / Death dan `/back`

Pembunuhan pemain dicatat di scoreboard `kills`, kematian di `deaths`. Saat pemain mati, lokasinya disimpan; ketik `/back` untuk kembali (lokasi dihapus setelah dipakai).

---

## Resource Pack dan Tampilan UI

### Variabel UI

File: `Qssentials [RP]/ui/_global_variables.json`

```json
{
  "$ess_form_disable_default_panel": true,
  "$ess_rulesUI_disable_custom_highlight_label": true,
  "$ess_actionbar_disable_custom_bar": true,
  "$ess_actionbar_disable_custom_dialog": true
}
```

| Variabel | Fungsi |
|---|---|
| `$ess_form_disable_default_panel` | Mengatur panel bawaan pada form Qssentials. |
| `$ess_rulesUI_disable_custom_highlight_label` | Mengatur label highlight custom pada UI Rules. |
| `$ess_actionbar_disable_custom_bar` | Mengatur bar custom pada actionbar. |
| `$ess_actionbar_disable_custom_dialog` | Mengatur dialog custom pada actionbar. |

Nilai `true` berarti fitur custom dinonaktifkan (sesuai nama variabel); `false` mengaktifkannya. Nilai bawaan build ini semuanya `true`.

### Bahasa dan Teks Pesan

- File bahasa: `Qssentials [RP]/texts/en_US.lang` dan `id_ID.lang`.
- Semua pesan addon memakai kunci `ess.*`. Edit teks setelah tanda `=`; `%s` adalah tempat nilai dinamis, jangan dihapus.
- Untuk menambah bahasa, buat file `.lang` baru lalu daftarkan di `texts/languages.json`.

### Pesan Khusus yang Disembunyikan dari Chat

Beberapa awalan teks dipakai Custom UI dan tidak tampil sebagai chat biasa: `nperma-woi` (title custom `/ctitle`), `@message@` (toast `/notif`), `cbar:` (bar info lahan), `clearchat-nperma` (pembersih chat).

### Urutan Pack

Jika memakai resource pack UI lain, letakkan Qssentials RP pada prioritas yang sesuai agar UI custom tidak tertimpa.

---

## ScriptEvent dan Integrasi

Addon lain atau NPC dapat memanggil Qssentials lewat `/scriptevent`.

| ScriptEvent | Pesan | Keterangan |
|---|---|---|
| `set:currency` | `<angka>` | Mengatur saldo pemain pengirim. |
| `add:currency` | `<angka>` | Menambah/mengurangi saldo pemain pengirim. |
| `register:plugin` | `{"namespace":"...","protocol":"...","description":"..."}` | Mendaftarkan plugin agar muncul di `/plugins`. |
| `b4173:notification` | `title\|message\|icon\|background` | Menampilkan toast notifikasi ke entitas pengirim. |

Catatan: `set:currency` dan `add:currency` hanya berfungsi jika pengirimnya adalah **pemain** (misalnya tombol dialog NPC), bukan command block.

Contoh toast:

```mcfunction
/execute as @a run scriptevent b4173:notification §aBerhasil!|Reward diklaim.|textures/ui/confirm|textures/ui/purpleBorder
```

### Database

Data disimpan sebagai dynamic property world: `player-db`, `scorehud-db`, `home-db`, `faction-db`, `land-db`, `warp-db`, `quest-db`. Menghapus world akan menghapus seluruh data.

---

## Struktur Folder

```text
Qssentials [BP]/
├─ manifest.json
├─ Changelog
└─ scripts/
   ├─ config.js              ← konfigurasi utama
   ├─ index.js               ← sistem inti (chat, scorehud, clear, broadcast, K/D)
   ├─ _db.js                 ← definisi database
   ├─ _loader/               ← pemuat inti dan sistem command
   ├─ extension/             ← fungsi bantu, forms, ekonomi, quick-db
   └─ commands/
      ├─ command.register.js ← daftar perintah yang diaktifkan
      ├─ general/  tpa/  warp/  home/  land/  mode/
      ├─ economy/  experimental/ (daily, weekly)
      └─ owner/  admin/

Qssentials [RP]/
├─ manifest.json
├─ texts/                    ← en_US.lang, id_ID.lang, languages.json
├─ ui/                       ← Custom UI (form, hud, bar, chat)
├─ textures/  font/
```

### Mengaktifkan / menonaktifkan perintah

Buka `scripts/commands/command.register.js`. Perintah aktif jika baris `import` tidak diberi komentar:

```js
import "./general/ping";        // aktif
//import "./admin/cdialog";     // nonaktif
```

---

## Troubleshooting

| Masalah | Penyebab / solusi |
|---|---|
| Perintah `/es:...` tidak muncul atau tidak jalan | Pastikan **BP dan RP** aktif, **Beta APIs** aktif, dan **cheats** menyala (atau `command_required_cheats: false`). |
| "You do not have permission to use this feature" | Pemain belum memiliki tag izin atau level akses yang dibutuhkan. Lihat [Rank dan Izin Akses](#rank-dan-izin-akses). |
| Rank tidak tampil di chat/nametag | Pastikan tag berawalan `rank:` dan `nametag.enabled` / template chat memuat `@rank`. |
| Scorehud tidak muncul | Cek `scorehud.enable_by_default` dan apakah pemain mematikannya via `/scorehud`. Pastikan RP aktif. |
| UI form terlihat polos/berantakan | Resource pack Qssentials belum aktif atau tertimpa pack UI lain. |
| `Required plugin(s) not loaded or incompatible` | Perintah membutuhkan plugin lain (mis. `/cdialog` butuh `DynamicDialogV2`). |
| Perubahan config tidak berlaku | Simpan file, lalu muat ulang world/server. |
| Lahan tidak bisa dibuat di dekat spawn | Area tersebut berada di dalam `lobby_protect.pos`. |

---

## Changelog

### Build 241026 (v2.0.0)

**Behavior Pack**

- Perbaikan bug `/daily` dan `/weekly`.
- Home UI: koordinat ditambahkan pada isi tombol.
- Varian teks actionbar baru: prefix `bar:` dan `cbar:` (dipakai Land Event).
- Custom UI baru untuk `/ranklist`; kunci terjemahan `ess.ui.ranklist.header1`, `header2`, dan `label` dihapus.
- Perintah `/cdialog` dinonaktifkan secara default.
- Land: refactor besar (dictionary dimensi, properti `bounds`, deteksi otomatis ID pada `/land sell`, struktur database baru, pengecekan dimensi lebih dulu agar lebih ringan).
- Land event: memakai `bounds`, hapus fitur fall id/damage, perbaikan logika `isInsideLand` dan kalkulasi cooldown pesan farmland, delay 2 tick untuk farmland.
- Core: pengurangan subscriber scriptevent dan penyederhanaan logika cooldown command.

**Resource Pack**

- Variabel baru: `$ess_actionbar_disable_custom_dialog`, `$ess_actionbar_disable_custom_bar`.

**Known bug**

- Farmland kadang tetap rusak jika berada pada `y = -64`.

---

## Kontributor dan Lisensi

**Kontributor**: Nperma, Fary
Terima kasih kepada BAO Community untuk basis sistem toast notifikasi.

> **Dilarang membagikan konten atau memakai resource addon ini tanpa izin.**
