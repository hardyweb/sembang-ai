---
title: "Dari OpenCode ke PiG: Cerita Aku Guna Coding Agent dalam TUI"
author: Hardy
date: 2026-10-10
tags: [coding-agent, tui, wsl, golang, nodejs, pig, pi, opencode, kilocode]
status: draf
---

# Dari OpenCode ke PiG: Cerita Aku Guna Coding Agent dalam TUI

Aku guna coding agent dalam terminal untuk belajar dan buat kerja berkaitan AI. Bukan IDE, bukan extension VS Code — semua dalam TUI, dalam WSL.

Artikel ni bukan review penuh atau benchmark rasmi. Aku cuma nak cerita pengalaman aku menggunakan satu coding agent ke satu coding agent yang lain, sampai akhirnya aku jumpa PiG, atau Pi-in-Go.

## Memula, OpenCode

Agent pertama yang aku cuba ialah OpenCode.

Rasa moden, TUI dia kemas, dan boleh terus jalan tanpa banyak setup. Install, login provider, mula kerja.

Masalahnya bukan pada ciri. Masalahnya pada beban.

Dalam WSL yang aku dah hadkan penggunaan memorinya, OpenCode terasa agak berat. Saiz binary yang aku guna sekitar 153 MB. Bila aku jalankan bersama Incus, database, proxy dan servis lain, penggunaan sumber mula terasa.

Aku bukan tak suka OpenCode. Dia jalan, dan dari segi pengalaman menggunakan coding agent, memang menarik. Cuma untuk mesin aku yang menjalankan banyak benda serentak, aku mula mencari alternatif yang lebih ringan.

## Kilocode

Lepastu aku cuba Kilo Code.

Dia lebih dekat dengan cara aku kerja. Banyak tool, banyak pilihan, dan macam-macam ciri yang dah tersedia. Aku guna agak lama, sampai config dan skill aku pun mula disusun berdasarkan workflow Kilo Code.

Tapi pola sama muncul balik.

Pakej pemasangan Kilo Code yang aku guna agak besar. Folder `node_modules` sahaja lebih 450 MB, manakala keseluruhan pakej melebihi 500 MB.

Dalam sesi interaktif, aku pernah ukur penggunaan memori sampai sekitar 700 MB.

Untuk desktop biasa, mungkin angka ni tak menjadi masalah besar. Tapi untuk persekitaran aku, WSL berkongsi sumber dengan Windows, dan dalam masa yang sama aku nak jalankan Incus, database serta servis homelab lain.

Memori tu bukan percuma. Setiap aplikasi ada kosnya.

Aku mula fikir: semua coding agent kena besar macam ni ke?

## Singgah Pi.dev

Lepastu aku cuba Pi.dev.

Aku tertarik dengan falsafah dia yang minimal. Konsepnya mudah: sesuaikan Pi dengan workflow kita, bukan paksa workflow kita ikut Pi.

Aku sempat install `@earendil-works/pi-coding-agent` versi 0.85.1. Macam kebanyakan coding agent yang aku cuba sebelum ni, ia berjalan menggunakan Node.js.

Masalahnya... dia terlalu minimal untuk cara aku bekerja masa tu.

Bila aku buka Pi, aku duduk termenung. Dok tau nok buat apa.

Bukan sebab dia tak boleh digunakan, tapi aku dah terbiasa dengan beberapa kemudahan dalam Kilo Code. Aku mencari workflow yang dah tersedia, termasuk perancangan kerja, pengurusan tugasan dan kawalan permission.

Dalam Pi, banyak perkara yang aku harapkan tu bukan ciri terbina dalam secara lalai. Falsafah projeknya memang memberi ruang kepada pengguna untuk menyesuaikan dan membina workflow sendiri.

Jujur, aku belum bersedia untuk tu.

Aku nak alat yang boleh terus membantu kerja aku, bukan alat yang memerlukan aku menghabiskan masa membina workflow sebelum boleh mula bekerja.

Jadi aku tinggal Pi buat sementara waktu, dan balik semula ke Kilo Code.

Aku tak salahkan Pi. Aku je yang belum sampai ke tahap tu.

## Jumpa PiG (Pi-in-Go)

Lepastu aku jumpa PiG, singkatan kepada Pi-in-Go.

Bila aku tahu ia merupakan implementasi Pi dalam Go, aku terus berkenan nak cuba.

Sebabnya mudah: aku memang berminat dengan Go, dan aku nak tengok sejauh mana coding agent yang menggunakan Go boleh berjalan dalam mesin yang sumbernya terhad.

Dalam dunia coding agent berasaskan terminal, banyak projek menggunakan JavaScript dan TypeScript dengan runtime seperti Node.js atau Bun. PiG pula menggunakan Go.

Perbezaan ni menarik bagi aku, bukan semata-mata sebab bahasa pengaturcaraan yang digunakan, tetapi sebab cara aplikasi tu dibina, dipasang dan dijalankan.

Aku nak tengok sendiri sama ada perbezaan tersebut memberi kesan yang boleh dirasa dalam penggunaan harian.

Jadi aku buat ujian ringkas pada mesin aku sendiri.

### Angka yang aku peroleh

Ini pemerhatian aku dalam persekitaran WSL2, x86_64.

| Coding agent | Runtime / teknologi | Saiz yang direkodkan | Puncak memori sesi interaktif |
|---|---|---:|---:|
| PiG | Go | ~61 MB | ~60 MB |
| Pi | Node.js | ~157 MB | ~246 MB |
| OpenCode | Bun | ~153 MB | ~539 MB |
| Kilo Code | Node.js | ~518 MB | ~702 MB |

**Nota:** Angka ni ialah hasil pemerhatian aku sendiri, bukan benchmark makmal terkawal. Saiz yang direkodkan bergantung pada artifak atau kaedah pemasangan yang digunakan. Penggunaan memori pula bergantung pada keadaan ujian, versi aplikasi dan cara proses diukur. Jadi, angka ni lebih sesuai dibaca sebagai gambaran penggunaan pada mesin aku, bukan keputusan mutlak untuk semua sistem.

Walaupun begitu, perbezaannya cukup ketara untuk menarik perhatian aku.

Dalam ujian yang aku buat, PiG menggunakan sekitar 60 MB memori, berbanding sekitar 702 MB untuk Kilo Code. Lebih kurang 11.7 kali perbezaan berdasarkan angka tersebut.

Untuk mesin yang aku kongsi dengan Incus, dnsmasq, database dan servis lain, perbezaan ni bukan sekadar angka atas kertas.

Ia menentukan berapa banyak ruang yang masih ada untuk aplikasi lain.

Aku turut perhatikan penggunaan memori ketika menjalankan arahan `--version`. Dalam ujian aku, PiG menggunakan sekitar 24 MB, Kilo Code sekitar 45 MB dan Pi sekitar 107 MB.

Tapi kena ingat, ini pemerhatian pada keadaan tertentu. Penggunaan memori semasa arahan ringkas tak semestinya menggambarkan prestasi atau penggunaan sumber ketika agent sedang menjalankan tugasan sebenar.

## Kenapa Go menarik untuk kes macam ni?

Ni bahagian teknikal yang aku rasa menarik.

Node.js menyediakan runtime JavaScript bersama enjin V8 dan komponen lain yang diperlukan untuk menjalankan aplikasi JavaScript. Bun pula menggunakan pendekatan runtime yang berbeza. Saiz pemasangan dan penggunaan memori setiap aplikasi turut bergantung pada framework, dependency dan cara aplikasi tersebut diedarkan.

Go pula membolehkan kod dikompil menjadi executable native. Untuk aplikasi yang dibina sebagai binary kendiri, kita boleh mengedarkannya tanpa perlu memasang Node.js atau membawa bersama `node_modules` seperti aplikasi JavaScript yang biasa.

Itu salah satu perkara yang menarik tentang PiG.

Cuma jangan salah faham. Program Go tetap mempunyai runtime sendiri, termasuk garbage collector. Binary Go juga tak semestinya statik sepenuhnya kerana ia bergantung pada pilihan build dan library yang digunakan.

Jadi, aku tak boleh kata Go sentiasa lebih ringan daripada Node.js dalam semua keadaan.

Apa yang aku boleh kata ialah, **dalam ujian aku, PiG memberikan footprint yang jauh lebih kecil berbanding agent lain yang aku cuba.**

Dan untuk kegunaan aku, itulah yang penting.

Antara kelebihan praktikal pendekatan ni:

- **Deployment lebih mudah:** binary boleh disalin dan dijalankan tanpa memasang runtime Node.js secara berasingan, jika semua keperluan binary tersebut dipenuhi.
- **Kurang kebergantungan pada pemasangan Node.js:** tak perlu mengurus runtime JavaScript untuk menjalankan aplikasi Go tersebut.
- **Penggunaan sumber yang lebih rendah dalam ujian aku:** PiG menggunakan memori yang jauh lebih sedikit ketika sesi interaktif.
- **Lebih mudah diintegrasikan dengan persekitaran homelab:** aku boleh mencuba agent tanpa menambah satu lagi pemasangan Node.js yang besar.

Dari sudut keselamatan pula, binary yang lebih ringkas boleh memudahkan pengurusan deployment. Tapi ia tak bermaksud aplikasi itu automatik lebih selamat. Kod, dependency, extension, permission dan cara API key diuruskan tetap perlu diberi perhatian.

Bagi aku, semua ni menjadikan PiG menarik untuk dicuba dalam persekitaran yang aku urus.

## Semua dalam TUI, semua dalam WSL

Mungkin konteks aku berbeza daripada pengguna coding agent yang bekerja sepenuhnya dalam IDE.

Aku menggunakan semua agent ni melalui TUI dalam WSL. Pada masa yang sama, WSL aku bukan sekadar tempat untuk menjalankan coding agent.

Ada Incus, database, proxy dan pelbagai servis lain yang aku gunakan untuk kerja serta eksperimen.

Dalam keadaan macam ni, ada beberapa perkara yang aku ambil kira:

- Memori WSL terhad dan dikongsi dengan Windows host.
- Aku nak kekalkan sumber untuk servis lain, bukan habiskan semuanya pada coding agent.
- Aku lebih suka aplikasi yang mudah dipasang, dijalankan dan diuruskan.
- Aku nak boleh biarkan agent terbuka ketika bekerja tanpa terlalu risau tentang penggunaan sumber.

Aku tak semestinya perlukan agent yang mempunyai paling banyak ciri. Aku perlukan agent yang sesuai dengan cara aku bekerja dan keadaan mesin aku.

Dalam konteks ni, PiG memberikan pengalaman yang aku cari.

Ia bukan bermaksud PiG paling pantas untuk setiap tugasan, paling bagus dari segi kualiti kod atau paling lengkap cirinya. Aku belum menjalankan ujian terkawal untuk membuat kesimpulan macam tu.

Apa yang aku nampak setakat ni ialah penggunaan sumbernya lebih rendah dalam ujian aku, dan itu cukup untuk menjadikannya pilihan yang menarik.

## Apa yang aku belajar?

Ada tiga perkara yang aku dapat daripada pengalaman ni.

### 1. Minimal tak semestinya bermaksud mudah

Pi.dev minimal dari segi ciri terbina dalam. Pengguna mempunyai kebebasan untuk membina workflow sendiri, tetapi kebebasan tu juga memerlukan masa dan usaha.

PiG pula menarik perhatian aku kerana footprint yang rendah dalam ujian aku.

Dua perkara ni berbeza.

Satu pendekatan memberikan kebebasan untuk menyesuaikan alat. Satu lagi kelebihan yang aku cari ialah penggunaan sumber yang rendah.

Sebagai pengguna, aku perlu tahu apa yang aku perlukan sebelum memilih sesuatu alat.

### 2. Pilihan runtime memang ada kesannya

Sebelum ni, aku lebih banyak menilai coding agent berdasarkan ciri dan workflow.

Selepas mencuba beberapa pilihan, aku mula memberi perhatian kepada perkara yang lebih asas: teknologi yang digunakan, saiz pemasangan, penggunaan memori dan cara aplikasi dijalankan.

Node.js bukan teknologi yang buruk. Ia memudahkan pembangunan banyak aplikasi dan mempunyai ekosistem yang luas.

Go juga bukan penyelesaian ajaib untuk semua masalah.

Tetapi bagi aplikasi yang aku jalankan sepanjang hari dalam persekitaran dengan sumber terhad, pilihan teknologi boleh memberikan perbezaan yang ketara.

Aku rasa ini antara perkara yang menarik untuk dipelajari oleh developer. Kadang-kadang, pengalaman pengguna bukan ditentukan oleh ciri yang kita nampak sahaja, tetapi juga oleh apa yang berlaku di belakang tabir.

### 3. Alat yang bagus ialah alat yang sesuai dengan kita

Kilo Code mempunyai banyak ciri yang aku suka. Pi pula menawarkan pendekatan minimal yang menarik. OpenCode juga mempunyai kelebihan tersendiri.

Tapi apabila aku menjalankan semuanya dalam mesin yang sama dengan servis lain, keutamaan aku berubah.

Aku mula menghargai alat yang tidak terlalu membebankan sistem.

Bagi aku, kalau sesuatu aplikasi boleh membantu aku bekerja tanpa menggunakan sumber yang berlebihan, itu satu kelebihan yang besar.

Aku tak perlukan alat yang sentiasa mengingatkan aku tentang kewujudannya melalui penggunaan memori atau beban sistem.

Aku nak alat yang membantu aku menyiapkan kerja.

## Penutup

Perjalanan aku bermula dengan OpenCode, kemudian Kilo Code, singgah di Pi.dev, dan akhirnya mencuba PiG.

Buat masa ni, PiG menjadi pilihan yang menarik untuk aku, bukan sebab ia paling hebat dari segi ciri, tetapi sebab ia lebih sesuai dengan mesin dan cara aku bekerja.

Aku masih menghargai apa yang aku dapat daripada OpenCode, Kilo Code dan Pi. Setiap projek ada pendekatan dan kelebihan tersendiri.

Cuma untuk aku yang menggunakan WSL, mengurus homelab dan menjalankan beberapa servis serentak, penggunaan sumber merupakan salah satu faktor penting dalam memilih coding agent.

Aku juga tak anggap pilihan ni sebagai keputusan muktamad. Projek masih berkembang, dan keperluan aku mungkin berubah apabila workflow aku semakin matang.

Kalau deme guna WSL atau mesin dengan sumber terhad, dan rasa coding agent yang digunakan sekarang agak membebankan, PiG mungkin berbaloi untuk dicuba.

Tak semestinya ia akan menjadi pilihan deme juga. Tapi kadang-kadang, kita hanya perlu mencuba sesuatu yang berbeza untuk sedar bahawa pilihan bahasa pengaturcaraan dan cara sesuatu aplikasi dibina boleh memberi kesan kepada pengalaman penggunaan harian.

Aku bermula dengan mencari coding agent yang sesuai untuk kerja aku.

Rupa-rupanya, aku turut belajar sesuatu tentang Go, runtime, penggunaan memori dan cara memilih alat yang betul untuk persekitaran sendiri.

Bagi aku, itu pun sebahagian daripada proses belajar.
