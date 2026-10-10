# Penjelasan Layman: Agent Harness (PiG / KiloCode)


Dokumen ini menerangkan lima komponen utama dalam workflow coding agent yang aku gunakan. Tujuannya mudah: memahami bagaimana agent menerima arahan, menyimpan konteks, mempelajari cara kerja dan mengikuti prinsip pembangunan perisian.

Kita tak perlu bermula dengan jargon teknikal. Kita gunakan analogi pekerja, buku panduan dan buku log supaya konsep lebih mudah difahami.

**Nota:** Dokumen ini menggabungkan konsep umum coding agent dengan konfigurasi peribadi aku. Tidak semua agent mempunyai mekanisme atau struktur fail yang sama. Perincian teknikal perlu dirujuk kepada dokumentasi agent masing-masing.

---

## 1. Skills — Buku Panduan Kerja

### Apa itu Skills?

Bayangkan agent AI sebagai pekerja baru yang sangat pintar, tetapi belum tahu cara kerja di tempat kita.

Dia tahu menulis kod, memahami arahan dan menyelesaikan masalah. Tetapi dia mungkin belum tahu:

- Macam mana kita suka commit message ditulis.
- Format laporan bulanan yang kita gunakan.
- Langkah-langkah deployment ke server.
- Standard keselamatan dalam projek.
- Cara kita mengurus projek Laravel.

**Skills ialah panduan yang mengajar agent cara melakukan sesuatu tugas mengikut keperluan kita.**

### Bezakan Capability dengan Knowledge

| Perkara | Extension / Tool | Skill |
|---|---|---|
| Fungsi | Memberi keupayaan melakukan sesuatu | Memberi panduan bagaimana melakukan sesuatu |
| Contoh | Menjalankan arahan Git | Menulis commit message mengikut format projek |
| Bentuk | Kod, integrasi atau mekanisme alat | Arahan dan bahan rujukan |
| Kegunaan | Agent perlu melakukan sesuatu tindakan | Agent perlu mengikuti kaedah kerja tertentu |

Contoh mudah:

Extension atau tool memberikan agent tangan untuk menggunakan Git. Skill pula mengajarnya cara menulis commit message yang kita kehendaki.

### Bagaimana Skills berfungsi?

Dalam PiG, salah satu bentuk skill ialah fail `SKILL.md` yang disimpan dalam direktori skill.

Contoh:

```text
~/.pig/skills/commit/SKILL.md
```

Kandungan fail boleh menerangkan nama skill, tujuan penggunaannya dan arahan yang perlu diikuti.

Contoh ringkas:

```markdown
---
name: commit
description: Tulis commit message mengikut format projek.
---

Gunakan format:

type(scope): ringkasan

Ringkasan mestilah padat dan jelas.
Terangkan sebab perubahan dalam body jika perlu.
Jangan masukkan perubahan yang tidak berkaitan.
```

Apabila agent menemui skill tersebut, ia boleh menggunakan maklumat dan arahan yang disediakan apabila skill berkenaan diperlukan.

Cara pemuatan dan penggunaan skill bergantung pada implementasi agent. Jangan anggap semua agent membaca keseluruhan kandungan setiap skill pada permulaan sesi.

### Bagaimana PiG menemui Skills?

PiG menyokong pemuatan skill secara eksplisit, termasuk penggunaan pilihan `--skill`, serta mekanisme discovery daripada direktori yang disokong.

Contoh:

```bash
pig --skill commit
```

Direktori yang digunakan perlu mengikut struktur dan konfigurasi yang disokong oleh versi PiG berkenaan.

### Kenapa Skills berguna?

Tanpa skill, kita mungkin perlu menerangkan semula cara kerja yang sama berulang kali.

Dengan skill, kita boleh menyimpan panduan tersebut dan menggunakannya semula.

Ibarat SOP (Standard Operating Procedure) untuk pekerja manusia.

**Ringkasan:** Skills ialah panduan kerja khusus yang boleh digunakan semula supaya agent dapat mengikuti kaedah kerja yang kita kehendaki.

---

## 2. AGENTS.md — Buku Peraturan Tetap

Kalau skill ialah panduan untuk tugas tertentu, `AGENTS.md` pula boleh digunakan untuk menetapkan peraturan umum dan arahan bagi agent yang bekerja dalam sesuatu projek.

Contohnya, sebelum mula bekerja, kita mahu agent memahami peraturan berikut:

- Jangan mengubah fail sensitif tanpa kebenaran.
- Periksa struktur projek sebelum membuat perubahan.
- Gunakan Form Request untuk pengesahan input Laravel.
- Jangan membuat deployment tanpa kelulusan.
- Sahkan hasil perubahan sebelum menyatakan kerja selesai.

Arahan seperti ini sesuai diletakkan dalam fail arahan projek.

### Apa yang patut ada dalam AGENTS.md?

Dalam workflow aku, kandungannya merangkumi beberapa perkara.

| Bahagian | Maksud |
|---|---|
| Global Principles | Prinsip asas seperti keselamatan dan backup |
| Workspace & Project Boundaries | Had kawasan kerja agent |
| Workflow Preferences | Cara kerja yang disukai, termasuk Laravel, Go dan Git |
| State & Memory | Arahan tentang cara membaca dan mengemas kini konteks kerja |
| Approval Gates | Tindakan yang memerlukan kelulusan |
| Verification | Keperluan mengesahkan hasil sebelum melaporkan kejayaan |

Ini ialah susunan yang aku gunakan untuk mengurus workflow sendiri, bukan format wajib bagi semua coding agent.

### Kenapa nama fail penting?

Agent tidak semestinya membaca semua fail Markdown dalam direktori projek.

Ia biasanya mempunyai mekanisme tertentu untuk mencari fail arahan yang dikenali. PiG mempunyai aturan pemuatan konteks tersendiri, manakala KiloCode juga mempunyai mekanisme arahan dan konfigurasi sendiri.

Oleh itu, jangan menganggap fail bernama `Agent.md`, `AGENTS.md` atau `CLAUDE.md` akan diproses dengan cara yang sama oleh semua agent.

Jika kita mahu menggunakan fail arahan tertentu, semak dokumentasi agent yang digunakan dan pastikan fail itu benar-benar dimuatkan.

### Kenapa AGENTS.md perlu ringkas?

Arahan yang sentiasa dimasukkan ke dalam konteks agent menggunakan sebahagian daripada token yang tersedia.

Sebab itu, aku lebih suka meletakkan peraturan umum yang penting dalam fail arahan utama. Panduan terperinci untuk tugas tertentu boleh disimpan dalam skills atau dokumen rujukan berasingan.

**Ringkasan:** `AGENTS.md` ialah tempat untuk meletakkan arahan dan batas kerja agent. Pastikan kandungannya jelas, relevan dan benar-benar dibaca oleh agent yang digunakan.

---

## 3. brain/ — Buku Ingatan Kerja

### Masalah yang cuba diselesaikan

Apabila sesi perbualan tamat, agent mungkin tidak lagi mempunyai keseluruhan konteks kerja yang diperlukan dalam sesi seterusnya.

Bayangkan kita sedang membangunkan sistem Laravel. Kita sudah memeriksa database, mengubah beberapa fail dan mengenal pasti satu masalah. Esok, kita membuka sesi baru.

Kalau konteks tidak disimpan, kita mungkin terpaksa mengulangi pemeriksaan yang sama.

Di sinilah konsep `brain/` berguna.

Dalam workflow aku, `brain/` ialah direktori untuk menyimpan catatan kerja yang boleh dibaca semula oleh agent pada sesi berikutnya.

Ia bukan bermaksud model AI tiba-tiba mempunyai ingatan kekal. Maklumat itu kekal kerana kita menyimpannya dalam fail dan mengarahkan agent supaya merujuknya semula.

### Analogi buku log jurutera

Buku log seorang jurutera biasanya merekodkan:

- Apa yang sudah dilakukan.
- Masalah yang ditemui.
- Keputusan yang dibuat.
- Perkara yang masih belum selesai.
- Langkah seterusnya.

Konsep yang sama digunakan dalam `brain/`.

### Contoh struktur brain/

Berikut ialah contoh struktur yang aku gunakan sebagai konvensyen kerja:

```text
brain/
├── task.md
├── walkthrough.md
├── architecture.md
├── ai_guidelines.md
├── gaya-penulisan.md
└── decisions/
    ├── 001-database.md
    └── 002-authentication.md
```

Fungsi setiap fail:

| Fail | Kegunaan |
|---|---|
| `task.md` | Status tugas, masalah semasa dan langkah seterusnya |
| `walkthrough.md` | Rekod perjalanan sesi kerja |
| `architecture.md` | Gambaran struktur dan hubungan komponen sistem |
| `decisions/` | Rekod keputusan teknikal dan sebab pemilihannya |
| `ai_guidelines.md` | Prinsip kerja dan pendekatan pembangunan |
| `gaya-penulisan.md` | Panduan gaya penulisan |

Struktur ini bukan struktur wajib PiG atau KiloCode. Ia ialah reka bentuk dokumentasi yang boleh disesuaikan mengikut projek.

### Mental Anchor — Penanda untuk sambung kerja

Satu idea yang aku gunakan ialah *Mental Anchor*.

Pada akhir catatan sesi, agent perlu menyatakan lokasi sebenar untuk menyambung kerja.

Contoh:

```markdown
## Status Semasa

- Migration telah diperiksa.
- Form Request telah dikemas kini.
- Feature test masih gagal pada kes authorization.

**Mental Anchor:**
Sambung dengan menyiasat kegagalan authorization
dalam feature test sebelum mengubah kod production.
```

Mental Anchor membantu agent mengetahui titik permulaan yang sesuai untuk sesi seterusnya.

Namun, catatan ini masih perlu disahkan dengan keadaan sebenar repository. Fail boleh berubah selepas catatan ditulis.

### Dua jenis brain dalam workflow aku

Aku membezakan catatan global dengan catatan khusus projek.

| Jenis | Contoh lokasi | Tujuan |
|---|---|---|
| Global | `~/.config/kilo/brain/` | Prinsip dan catatan umum workflow |
| Projek | `<projek>/.agents/brain/` | Konteks dan status projek tertentu |

Lokasi ini ialah konvensyen peribadi aku, bukannya lokasi standard yang dijamin dibaca secara automatik oleh PiG atau KiloCode.

Agent perlu diarahkan untuk membaca lokasi yang betul. Jika ada catatan global dan projek, aturan keutamaan juga perlu ditetapkan supaya konteks tidak bercanggah.

### Peraturan penting

1. Jangan simpan password, API token atau rahsia lain dalam fail brain.
2. Catat perkara yang benar-benar berlaku, bukan perkara yang diandaikan.
3. Nyatakan masalah yang belum selesai dengan jelas.
4. Rekodkan keputusan penting dan sebabnya.
5. Gunakan Git untuk mengesan perubahan pada fail dokumentasi apabila sesuai.

**Ringkasan:** `brain/` ialah sistem catatan kerja yang membantu mengekalkan konteks antara sesi, dengan syarat agent membaca dan mengemas kini catatan tersebut dengan betul.

---

## 4. ai_guidelines.md — Falsafah Kerja dan Cara Berfikir

Kalau `AGENTS.md` menerangkan arahan dan batas kerja, `ai_guidelines.md` pula menerangkan prinsip yang menjadi panduan kepada cara kerja aku.

Ia bukan fail konfigurasi ajaib. Ia ialah dokumen yang mengandungi prinsip yang mahu aku kekalkan dalam pembangunan perisian dengan bantuan AI.

### Analogi mudah

- `AGENTS.md`: Jangan langgar lampu merah.
- `ai_guidelines.md`: Aku percaya keselamatan lebih penting daripada sampai cepat.

Yang pertama menetapkan arahan. Yang kedua menerangkan prinsip di sebalik cara kita bekerja.

### Prinsip utama

Dalam workflow aku, antara prinsip yang penting ialah:

**1. Manusia memahami dan bertanggungjawab**

AI membantu menyediakan penyelesaian, tetapi aku perlu memahami, mengesahkan dan bertanggungjawab terhadap keputusan akhir.

**2. Kod mudah mengatasi kod yang terlalu clever**

Penyelesaian yang mudah dibaca dan diselenggara biasanya lebih sesuai daripada kod yang kelihatan hebat tetapi sukar difahami.

**3. Keselamatan bermula sejak reka bentuk**

Security bukan kerja tambahan selepas sistem siap. Ia perlu dipertimbangkan sejak awal pembangunan.

**4. Periksa sebelum mengubah**

Agent perlu memahami keadaan sebenar projek sebelum membuat perubahan. Jangan mengandaikan struktur, dependency atau konfigurasi tanpa pemeriksaan.

**5. Git dan deployment kekal di bawah kawalan manusia**

Agent boleh menyediakan perubahan, menjalankan ujian dan melaporkan hasil. Dalam workflow aku, keputusan untuk commit, push dan deploy kekal di tangan manusia.

**6. Terus terang apabila tidak pasti**

Jika sesuatu arahan gagal, agent perlu melaporkan kegagalan tersebut dan bukannya mendakwa kerja sudah selesai.

### Prinsip utama aku

> AI assists. Hardy understands, verifies, and owns the final decision.

AI membantu. Aku memahami, mengesahkan dan bertanggungjawab terhadap keputusan akhir.

Dokumen ini boleh dirujuk apabila agent perlu memahami pendekatan pembangunan yang aku utamakan. Untuk memastikan ia benar-benar digunakan, arahan utama perlu memberitahu agent bila dan bagaimana hendak membaca fail tersebut.

**Ringkasan:** `ai_guidelines.md` ialah rujukan kepada prinsip dan pendekatan kerja yang aku mahu kekalkan dalam pembangunan perisian dengan bantuan AI.

---

## 5. soul.md — Tujuan dan Falsafah Mendalam Agent

Kalau `ai_guidelines.md` memberikan ringkasan prinsip kerja, `soul.md` pula digunakan dalam workflow aku untuk menghuraikan prinsip tersebut dengan lebih mendalam.

Nama `soul.md` bukan bermaksud agent mempunyai jiwa atau kesedaran seperti manusia. Ia ialah nama fail yang aku pilih untuk menyimpan falsafah dan tujuan reka bentuk agent.

### Perbezaan antara tiga fail

| Fail | Soalan utama |
|---|---|
| `AGENTS.md` | Apakah arahan dan batas kerja aku? |
| `ai_guidelines.md` | Apakah prinsip kerja yang perlu aku ikuti? |
| `soul.md` | Mengapa prinsip tersebut penting? |

Analogi mudah:

- `AGENTS.md` ialah buku peraturan.
- `ai_guidelines.md` ialah nota ringkas di atas meja.
- `soul.md` ialah dokumen yang menerangkan sebab di sebalik prinsip tersebut.

### Apa yang terkandung dalam soul.md?

Dalam reka bentuk aku, dokumen ini merangkumi prinsip seperti:

- **Security First:** Keselamatan dipertimbangkan sejak awal.
- **Human Judgment First:** Manusia kekal bertanggungjawab terhadap keputusan.
- **Verify, Don't Assume:** Pengesahan lebih penting daripada andaian.
- **Production-Ready:** Perubahan dibuat dengan mengambil kira kebolehselenggaraan dan pemulihan.
- **Self-Hosted Pragmatism:** Mengutamakan penyelesaian open-source dan self-hosted apabila sesuai.
- **Learn Fundamentals:** Memahami asas Linux, rangkaian, SQL dan pembangunan perisian.
- **CLI Transparency:** Menunjukkan arahan dan hasil sebenar supaya kerja boleh difahami.
- **Backup & Rollback:** Merancang pemulihan sebelum perubahan berisiko.
- **Documentation:** Dokumentasi ialah sebahagian daripada kerja.
- **Preserve Context:** Menyimpan maklumat penting untuk sesi seterusnya.
- **Bounded Steps:** Membuat perubahan secara terkawal dan berperingkat.
- **Agent Boundaries:** Tidak mereka hasil, menyembunyikan ralat atau mendakwa kejayaan tanpa bukti.

Prinsip ini membentuk identiti workflow yang aku mahu bina, bukannya keupayaan yang terjamin tersedia secara automatik dalam sesuatu model.

### Mental Anchor — Kitaran kerja utama

Dalam workflow aku, kerja agent mengikuti kitaran berikut:

```text
Understand
    ↓
Inspect
    ↓
Plan
    ↓
Change
    ↓
Verify
    ↓
Document
    ↓
Next Task
```

Maksudnya:

1. **Understand:** Fahami permintaan dan masalah.
2. **Inspect:** Periksa keadaan sebenar sistem.
3. **Plan:** Tentukan perubahan yang diperlukan.
4. **Change:** Laksanakan perubahan secara terkawal.
5. **Verify:** Jalankan ujian atau pemeriksaan yang sesuai.
6. **Document:** Catat perubahan dan hasil pengesahan.
7. **Next Task:** Tentukan langkah seterusnya.

Kitaran ini membantu mengelakkan agent daripada terus mengubah kod sebelum memahami masalah.

### Satu prinsip yang aku pegang

> Do the work, verify the work, and leave enough evidence for the next person—or the next session—to understand the work.

Buat kerja, sahkan hasilnya dan tinggalkan bukti yang mencukupi supaya orang lain atau sesi seterusnya boleh memahami apa yang telah dilakukan.

**Ringkasan:** `soul.md` ialah dokumen falsafah mendalam yang menerangkan tujuan, prinsip dan pendekatan yang aku mahu jadikan panduan kepada workflow coding agent.

---

## 6. Gambaran Besar — Lima Lapisan Workflow

Selepas memahami setiap komponen, kita boleh melihat bagaimana semuanya saling melengkapi.

```text
soul.md
   ↓
Falsafah dan tujuan kerja

ai_guidelines.md
   ↓
Prinsip dan pendekatan pembangunan

AGENTS.md
   ↓
Arahan dan batas kerja

brain/
   ↓
Konteks dan catatan antara sesi

skills/
   ↓
Panduan bagi tugas khusus
```

Setiap komponen menjawab soalan yang berbeza.

| Komponen | Soalan yang dijawab | Peranan |
|---|---|---|
| `soul.md` | Mengapa prinsip ini penting? | Falsafah mendalam |
| `ai_guidelines.md` | Bagaimana aku mahu bekerja? | Prinsip kerja |
| `AGENTS.md` | Apakah arahan dan batas kerja? | Arahan agent |
| `brain/` | Apa yang telah berlaku dan apa seterusnya? | Catatan kerja |
| `skills/` | Bagaimana tugas tertentu patut dilakukan? | Panduan khusus |

Walaupun kelima-lima komponen ini berbeza, semuanya boleh membantu membina workflow yang lebih konsisten.

Namun, keberkesanannya bergantung pada cara agent dikonfigurasikan, fail yang benar-benar dibaca dan ketepatan maklumat yang disimpan.

## 7. Kesimpulan

Bagi aku, penggunaan coding agent bukan sekadar memberikan prompt dan menunggu kod siap.

Aku mahu agent memahami batas kerjanya, mengikuti panduan projek, mengekalkan konteks dan mengesahkan hasil perubahan. Pada masa yang sama, aku mahu proses kerja kekal telus supaya aku boleh belajar daripada setiap perubahan yang dibuat.

Lima komponen ini membantu memisahkan tanggungjawab tersebut:

- **Skills:** Panduan untuk tugas tertentu.
- **AGENTS.md:** Arahan dan batas kerja.
- **brain/:** Catatan dan konteks kerja.
- **ai_guidelines.md:** Prinsip pembangunan.
- **soul.md:** Falsafah yang mendasari workflow.

Tidak semua projek memerlukan kelima-lima komponen. Projek kecil mungkin memadai dengan arahan ringkas dan beberapa skills. Projek yang lebih kompleks mungkin memerlukan rekod keputusan, dokumentasi seni bina dan catatan sesi yang lebih tersusun.

Yang penting, jangan membina terlalu banyak lapisan semata-mata kerana ia kelihatan menarik. Gunakan apa yang benar-benar membantu kerja.

Bagi aku, prinsip akhirnya mudah:

**Handle the routine.** — Urus kerja rutin.

**Surface the overlooked.** — Bangkitkan perkara yang mungkin terlepas pandang.

**Explain the complicated.** — Terangkan perkara yang rumit.

**Verify the important.** — Sahkan perkara yang penting.

**Preserve what was learned.** — Simpan apa yang telah dipelajari.

**Leave the human in control.** — Manusia kekal mengawal keputusan akhir.

Itulah asas workflow coding agent yang aku mahu bina: AI membantu melakukan kerja, tetapi manusia tetap memahami sistem, mengesahkan hasil dan bertanggungjawab terhadap keputusan.
