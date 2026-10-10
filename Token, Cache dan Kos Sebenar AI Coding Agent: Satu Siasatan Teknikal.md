# Token, Cache dan Kos Sebenar AI Coding Agent: Satu Siasatan Teknikal

*Analisis penggunaan PiG, 75 skills, DeepSeek V4.1 Flash dan sistem kredit prabayar.*

**Tarikh:** 10 Oktober 2026  
**Persekitaran:** PiG coding agent  
**Provider:** asAi (`serveras.click`)  
**Model:** `asai/deepseek-v4.1-flash`

## Pengenalan

Berapakah kos sebenar untuk menjalankan AI coding agent yang mempunyai pelbagai lapisan arahan, konteks projek, memori berterusan dan puluhan skills?

Itulah persoalan yang timbul selepas aku mengkonfigurasi PiG coding agent dengan beberapa komponen:

- `soul` — identiti dan prinsip operasi agent.
- `AGENTS.md` — arahan umum untuk agent.
- `AI_GUIDELINES.md` — garis panduan pembangunan.
- `brain/` — lapisan state dan maklumat kontekstual yang disimpan.
- 75 skills yang tersedia untuk digunakan oleh agent.

Andaian awal aku ialah konfigurasi ini mungkin menghasilkan sekitar 20,000 input token bagi setiap request kerana semua lapisan tersebut akan menyumbang kepada prompt.

Aku mula mempersoalkan sama ada konfigurasi ini terlalu mahal. Perlukah sebahagian skills dibuang? Perlukah arahan dipendekkan?

Pada masa yang sama, aku mempunyai satu lagi andaian:

> Kalau sebahagian besar konteks boleh dicache, kosnya mungkin tidak menjadi masalah besar.

Daripada membuat andaian berdasarkan saiz fail, aku memilih untuk menyiasat rekod penggunaan sebenar daripada sesi PiG dan membandingkannya dengan formula pengebilan kredit yang digunakan oleh asAi.

Hasil siasatan ini memberikan tiga penemuan utama:

1. Anggaran awal 20,000 input token baharu bagi setiap request terlalu tinggi.
2. Prompt caching mengurangkan penggunaan kredit yang dikira sebanyak kira-kira 60.5% berbanding senario hipotesis tanpa diskaun cache.
3. Nilai kos yang dipaparkan oleh PiG bukan bil USD sebenar yang disahkan oleh provider.

Artikel ini menerangkan apa yang aku ukur, bagaimana pengiraan dibuat dan apa yang aku pelajari daripada eksperimen tersebut.

## 1. Memahami sistem pengebilan asAi

Sebelum menganalisis penggunaan token, aku perlu memahami bagaimana asAi mengenakan caj untuk request model.

Perkara penting yang perlu difahami ialah **asAi menggunakan sistem kredit prabayar, bukannya pengebilan terus dalam USD bagi setiap token untuk model yang diuji ini.**

PiG memaparkan nilai kos dengan simbol `$`. Walau bagaimanapun, nilai tersebut datang daripada konfigurasi `cost` dalam `models.json`. Ia merupakan anggaran yang dikira oleh PiG, bukannya bil USD sebenar yang disahkan oleh asAi.

Untuk siasatan ini, aku menggunakan multiplier kredit yang direkodkan daripada dokumentasi pricing provider.

Rujukan rasmi:

- [Harga asAi](https://serverasai.com/#pricing)
- [Dokumentasi pengebilan asAi](https://serverasai.com/docs/billing)
- [Harga model asAi](https://serverasai.com/docs/models)

Bagi model `asai/deepseek-v4.1-flash`, kadar yang digunakan dalam pengiraan ini ialah:

| Komponen | Multiplier kredit |
|---|---:|
| Input baharu | 2.5 |
| Input daripada cache (*cache read*) | 0.625 |
| Output token | 12.5 |

Formula pengiraan:

```text
kredit = ceil(
    input_baharu * 2.5
    + cache_read * 0.625
    + output * 12.5
)
```

Provider turut menetapkan caj minimum sebanyak 1,000 kredit bagi setiap request yang berjaya.

Terdapat dua perkara penting di sini.

Pertama, token daripada cache hanya dikenakan satu perempat daripada kadar input baharu.

Kedua, multiplier output token adalah lima kali lebih tinggi berbanding input baharu.

Ini bermakna penggunaan cache dan panjang output boleh memberikan kesan yang ketara terhadap jumlah kos.

Kadar tersebut khusus untuk model dan konfigurasi harga yang digunakan dalam eksperimen ini. Semak dokumentasi semasa sebelum menggunakannya untuk model lain.

## 2. Kaedah siasatan

Aku tidak menganggarkan penggunaan token berdasarkan saiz fail konfigurasi semata-mata. Sebaliknya, aku memeriksa rekod sesi sebenar PiG.

PiG menyimpan data sesi dalam format JSONL di direktori:

```text
~/.pig/agent/sessions/
```

Rekod mesej assistant boleh mengandungi maklumat penggunaan seperti berikut:

```json
{
  "input": 2436,
  "cacheRead": 29184,
  "cacheWrite": 0,
  "output": 366,
  "reasoning": 120,
  "cost": {
    "total": 0.02976
  }
}
```

Contoh ini menunjukkan struktur rekod yang diperiksa, bukannya penyata bil lengkap.

Aku menganalisis lima perkara:

1. **Penggunaan token:** Input baharu, cache reads dan output yang dijana.
2. **Cache hit rate:** Peratus input yang diwakili oleh cache reads.
3. **Komposisi system prompt:** Saiz setiap bahagian prompt yang direkodkan.
4. **Peristiwa dalam sesi:** Pertukaran model dan proses context compaction.
5. **Tool results yang besar:** Kandungan yang berpotensi menambah saiz konteks dan mempengaruhi penggunaan cache.

Snapshot pertama mengandungi 122 request. Oleh sebab sesi masih berjalan, aku membekukan snapshot tersebut untuk memastikan pengiraan menggunakan set data yang tetap.

Aku turut membandingkan hasilnya dengan satu snapshot berasingan daripada dashboard provider bagi tempoh 24 jam.

Kedua-dua snapshot ini merupakan set data yang berbeza dan tidak boleh dianggap sebagai rekod request yang sama.

## 3. Penemuan pertama: Andaian 20,000 token tidak tepat

Hasil pertama agak jelas.

| Metrik | Hasil pengukuran |
|---|---:|
| Request dianalisis | 122 |
| Purata input baharu setiap request | 6,234 token |
| Purata cache reads setiap request | 48,069 token |
| Saiz system prompt | 6,550 token |

Anggaran awal aku sebanyak 20,000 input token baharu bagi setiap request adalah kira-kira tiga kali lebih tinggi daripada purata yang diukur.

Namun, perbezaan antara input baharu dengan cache reads sangat penting.

Sesuatu request boleh menggunakan sejumlah besar konteks terdahulu tanpa semua token tersebut dikenakan kadar input baharu. Mekanisme prompt caching membolehkan provider mengurangkan kos bagi prefix prompt yang boleh digunakan semula.

Dalam sesi aku, purata cache reads jauh lebih tinggi berbanding purata input baharu.

Ini mengubah cara aku melihat konfigurasi tersebut.

Agent memang mempunyai konteks yang besar, tetapi itu tidak bermaksud setiap request membayar kadar input baharu untuk keseluruhan konteks.

**Konteks yang besar tidak semestinya bersamaan dengan jumlah input baharu yang besar untuk tujuan pengebilan.**

## 4. Penemuan kedua: Cache mengurangkan penggunaan kredit yang dikira sebanyak 60.5%

Berikut ialah jumlah token daripada snapshot 122 request.

| Metrik | Nilai |
|---|---:|
| Cache reads | 5,864,448 token |
| Input baharu | 760,574 token |
| Output | 128,006 token |
| Cache hit rate | Kira-kira 88.5% |

Untuk pengiraan ini, aku menganggap medan input baharu dan cache reads sebagai dua komponen yang berasingan.

### Pengiraan kredit

Input baharu:

```text
760,574 * 2.5 = 1,901,435 kredit
```

Cache reads:

```text
5,864,448 * 0.625 = 3,665,280 kredit
```

Output:

```text
128,006 * 12.5 = 1,600,075 kredit
```

Jumlah:

```text
1,901,435
+ 3,665,280
+ 1,600,075
--------------
7,166,790 kredit
```

Walau bagaimanapun, jumlah yang direkodkan pada peringkat sesi ialah 7,166,821 kredit.

Perbezaannya ialah 31 kredit. Salah satu kemungkinan ialah pembundaran `ceil()` dilakukan bagi setiap request sebelum semua caj dijumlahkan, bukannya sekali sahaja pada jumlah agregat. Namun, angka agregat yang tersedia tidak mencukupi untuk mengesahkan punca sebenar.

Untuk perbandingan seterusnya, aku mengekalkan jumlah yang direkodkan oleh sesi.

### Bagaimana jika tiada diskaun cache?

Untuk menganggarkan kesan diskaun cache, aku mengira semula beban token yang sama seolah-olah token daripada cache dikenakan kadar input baharu.

Jumlah input:

```text
760,574 + 5,864,448
= 6,625,022 token
```

Kos input tanpa diskaun cache:

```text
6,625,022 * 2.5
= 16,562,555 kredit
```

Kos output:

```text
128,006 * 12.5
= 1,600,075 kredit
```

Jumlah:

```text
16,562,555 + 1,600,075
= 18,162,630 kredit
```

Dengan mengambil kira perbezaan 31 kredit yang ditemui dalam rekod sesi, perbandingannya adalah seperti berikut:

| Senario | Kredit |
|---|---:|
| Penggunaan sebenar yang direkodkan dengan cache | 7,166,821 |
| Anggaran tanpa diskaun cache | 18,162,661 |
| Perbezaan | 10,995,840 |
| Pengurangan kredit yang dikira | Kira-kira 60.5% |

Perbandingan ini menggunakan jumlah token yang sama dengan multiplier harga yang berbeza. Ia bukan eksperimen terkawal yang menjalankan semula workload dengan caching dimatikan.

Walau bagaimanapun, pengiraan tersebut menunjukkan kesan kewangan diskaun cache berdasarkan struktur harga yang digunakan.

### Menukar kredit kepada Ringgit

Pek prabayar RM5 menyediakan 80 juta kredit berdasarkan kadar yang direkodkan dalam eksperimen ini.

Dengan menggunakan kadar tersebut:

```text
Anggaran kos dengan cache:

7,166,821 * RM5 / 80,000,000
≈ RM0.4479
```

Bagi senario hipotesis tanpa diskaun cache:

```text
18,162,661 * RM5 / 80,000,000
≈ RM1.1352
```

Nilai ini ialah anggaran kos berdasarkan harga pembelian kredit. Ia bukan jumlah yang dibilkan secara langsung dalam Ringgit oleh provider.

Penemuan utama ialah penggunaan cache mengurangkan kos pengiraan bagi workload ini dengan ketara.

## 5. Penemuan ketiga: Pengiraan sepadan dengan dashboard provider

Aku mahu memastikan sama ada pengiraan berdasarkan rekod sesi konsisten dengan data yang dipaparkan oleh provider.

Untuk snapshot berasingan bagi tempoh 24 jam, dashboard menunjukkan angka berikut:

| Metrik dashboard | Nilai |
|---|---:|
| Kredit digunakan | 15,500,000 |
| Baki kredit | 70,700,000 |
| Jumlah request | 259 |
| Cache hit rate | 89.8% |
| Purata kredit setiap request | 60.0K |
| Input tokens | 15.5M |
| Cache reads | 13.9M |
| Output tokens | 224.1K |
| Purata output setiap request | 865 |

Angka ini merupakan nilai yang telah dibundarkan oleh dashboard.

### Mengira semula jumlah kredit

Jumlah input yang dipaparkan merangkumi token daripada cache. Oleh itu, anggaran input baharu ialah:

```text
Input baharu:
15.5M - 13.9M
= 1.6M token
```

Menggunakan formula harga yang sama:

```text
Input baharu:
1,600,000 * 2.5
= 4,000,000 kredit

Cache reads:
13,900,000 * 0.625
= 8,687,500 kredit

Output:
224,100 * 12.5
= 2,801,250 kredit
```

Jumlah:

```text
4,000,000 + 8,687,500 + 2,801,250
= 15,488,750 kredit
```

Perbandingan:

| Pengukuran | Kredit |
|---|---:|
| Pengiraan daripada angka dashboard | 15,488,750 |
| Nilai pada dashboard | 15,500,000 |
| Perbezaan | 11,250 atau kira-kira 0.07% |

Perbezaan yang kecil ini konsisten dengan pembundaran angka yang dipaparkan.

Purata input baharu yang dianggarkan juga hampir sama dengan ukuran bebas daripada snapshot 122 request:

```text
1,600,000 / 259
≈ 6,178 token setiap request
```

Angka ini hampir dengan purata 6,234 token yang diperoleh daripada snapshot sesi pertama.

Ini merupakan semakan konsistensi yang berguna, walaupun bukan audit bebas terhadap sistem pengebilan dalaman provider.

### Perangkap istilah: `Input tokens`

Metrik ini mudah disalah tafsir.

Jika kesemua 15.5 juta input token dianggap sebagai input baharu, anggaran kos akan menjadi jauh lebih tinggi daripada yang sepatutnya berdasarkan pecahan yang dipaparkan.

Dalam snapshot ini:

```text
Jumlah input pada dashboard: 15.5M
Cache reads:                 13.9M
Anggaran input baharu:        1.6M
```

Sentiasa semak definisi metrik yang digunakan oleh provider sebelum membuat pengiraan kos.

Nama medan yang hampir sama antara API yang berbeza tidak semestinya mempunyai maksud pengebilan yang sama.

## 6. Penemuan keempat: Skills menggunakan 72% daripada system prompt

System prompt yang direkodkan mempunyai saiz 24,891 bait, bersamaan kira-kira 6,550 token dalam sesi ini.

Pecahannya adalah seperti berikut:

| Bahagian | Saiz | Peratus |
|---|---:|---:|
| `skills` | 18,060 bait | 72% |
| `project_context` | 4,220 bait | 17% |
| `docs` | 1,236 bait | 5% |
| `rules` | 839 bait | 3% |
| `tools` | 339 bait | 1% |
| `preamble` | 170 bait | 1% |
| `cwd` | 27 bait | Kurang daripada 1% |

Peratusan telah dibundarkan.

Bahagian `skills` merupakan komponen terbesar dalam system prompt.

Aku turut memeriksa struktur metadata bagi 75 skills tersebut.

| Metadata skills | Saiz |
|---|---:|
| Medan `<description>` | 6,313 bait |
| Medan `<location>` | 5,713 bait |
| Medan `<name>` | 1,971 bait |
| Tag `<skill>` | 1,575 bait |

Kira-kira 7,288 bait, atau sekitar 1,918 token berdasarkan anggaran yang digunakan dalam siasatan ini, datang daripada metadata dan struktur yang berulang.

Sebahagian kandungan tersebut tetap berguna. Nama dan penerangan skill membantu agent memilih keupayaan yang sesuai. Lokasi fail pula memberitahu agent di mana skill tersebut disimpan.

Namun, laluan fail dan markup yang berulang mungkin menyediakan ruang untuk pengoptimuman.

Persoalan sebenar bukan sekadar sama ada 75 ialah jumlah skills yang besar. Persoalannya ialah sama ada semua skills tersebut perlu dimasukkan dalam konteks awal.

### Perlukah aku membuang sebahagian skills?

Tidak semestinya.

Prompt yang lebih kecil boleh mengurangkan kos pemprosesan awal, tetapi membuang skills yang berguna juga boleh menjejaskan kualiti tugasan atau memaksa agent membuat request tambahan untuk mencari maklumat yang hilang.

Eksperimen yang lebih baik ialah membandingkan dua konfigurasi:

- Konfigurasi sedia ada dengan 75 skills.
- Konfigurasi yang dikurangkan kepada skills yang relevan untuk workload tertentu.

Kemudian, aku boleh mengukur kualiti penyelesaian tugasan, input baharu, cache reads, output token dan jumlah kredit.

Dengan cara itu, kita boleh menentukan sama ada pengurangan skills benar-benar meningkatkan keberkesanan kos agent.

## 7. Penemuan kelima: Cache miss boleh menjadi mahal

Caching memang membantu, tetapi kesannya tidak sama bagi setiap request.

Aku mengenal pasti lima request dengan cache miss penuh dalam snapshot 122 request.

| Turn | Input token | Kredit | Pemerhatian |
|---|---:|---:|---|
| 1 | 9,499 | 29,748 | Request pertama |
| 31 | 55,987 | 145,955 | Prefix berubah |
| 41 | 23,836 | 64,178 | Context compaction |
| 73 | 53,193 | 153,820 | Prefix berubah |
| 111 | 89,977 | 227,580 | Prefix berubah |

Kesemua request tersebut menggunakan 621,281 kredit, iaitu kira-kira 8.7% daripada jumlah kredit yang direkodkan.

Timeline sesi menunjukkan peristiwa berikut:

```text
22:52:45  MODEL_CHANGE
          openrouter/free -> serveras-click

22:53:01  MODEL_CHANGE

23:07:20  COMPACTION
```

Pertukaran model atau proses context compaction boleh mengganggu penggunaan semula prefix prompt yang dicache. Bergantung pada pelaksanaan caching provider, request berikutnya mungkin perlu memproses semula sejumlah besar input.

Walau bagaimanapun, tidak semua cache miss berlaku sejurus selepas pertukaran model atau compaction.

Turn 31, 73 dan 111 tidak mempunyai peristiwa berkaitan yang jelas dalam timeline berdekatan. Perubahan pada prefix konteks ialah satu kemungkinan, tetapi rekod yang tersedia tidak membuktikan punca sebenar.

Tool result yang besar atau perubahan struktur konteks wajar disiasat, tetapi aku tidak boleh menyimpulkan bahawa faktor tersebut menyebabkan setiap cache miss.

### Apa yang patut dipantau selepas ini?

Untuk coding agent yang berjalan dalam sesi panjang, aku akan memantau:

- Pertukaran model dan kesannya terhadap cache reuse.
- Peristiwa context compaction.
- Tool result besar yang ditambah kepada perbualan.
- Perubahan system prompt dan metadata skills.
- Lonjakan input baharu selepas beberapa request yang sebelum itu cekap menggunakan cache.

Matlamatnya bukan untuk menghapuskan semua cache miss. Ada keadaan yang tidak dapat dielakkan. Matlamatnya ialah mengenal pasti corak yang mahal dan menentukan sama ada corak tersebut boleh dikurangkan tanpa menjejaskan keberkesanan agent.

## 8. Kredit bukan Ringgit

Dashboard provider memaparkan penggunaan dan baki kredit, tetapi tidak menyediakan kos Ringgit secara terus.

Kos efektif bergantung pada pek prabayar yang digunakan.

Kadar yang direkodkan dalam eksperimen ini ialah:

| Pek | Jumlah kredit | Kredit bagi setiap RM |
|---|---:|---:|
| RM5 | 80 juta | 16 juta |
| RM8 | 140 juta | 17.5 juta |
| RM10 | 200 juta | 20 juta |

Formula penukaran:

```text
Anggaran kos RM =
    kredit digunakan * harga pek / jumlah kredit pek
```

Bagi penggunaan 15.5 juta kredit pada dashboard:

| Pek | Anggaran kos |
|---|---:|
| RM5 | RM0.9688 |
| RM8 | RM0.8857 |
| RM10 | RM0.7750 |

Jumlah kredit yang sama boleh mempunyai kos pemerolehan berbeza bergantung pada pek yang dibeli.

Sebab itu, aku tidak akan melaporkan kos Ringgit tanpa menyatakan kadar pek yang digunakan dalam pengiraan.

Angka ini juga tidak mengambil kira kemungkinan kredit luput, kredit promosi atau pelarasan akaun.

### Anomali baki kredit yang belum dijelaskan

Snapshot dashboard menunjukkan satu lagi perkara yang perlu disiasat.

| Snapshot | Kredit digunakan | Baki kredit | Jumlah |
|---|---:|---:|---:|
| Terdahulu | 11.3M | 76.7M | 88.0M |
| Terkini | 15.5M | 70.7M | 86.2M |

Jumlah tersebut berkurangan sebanyak 1.8 juta kredit.

Jika jumlah awal sebanyak 88 juta kredit tidak berubah, baki yang sepatutnya ialah:

```text
88.0M - 15.5M = 72.5M kredit
```

Namun, dashboard memaparkan baki sebanyak 70.7 juta kredit.

Aku tidak dapat menentukan puncanya berdasarkan data yang tersedia.

Antara kemungkinan ialah pembundaran, kredit luput atau perbezaan cara kredit promosi dan pelarasan akaun direkodkan. Semua ini masih hipotesis, bukannya penjelasan yang telah disahkan.

Provider menyatakan tempoh sah kredit selama 30 hari. Oleh itu, isu kredit luput wajar diperiksa, tetapi maklumat tersebut sahaja tidak membuktikan bahawa kredit luput menjadi punca perbezaan baki.

Untuk tujuan belanjawan yang tepat, aku akan menyemak sejarah transaksi kredit atau mendapatkan penjelasan daripada pihak provider.

## 9. Pengajaran praktikal untuk pembangun coding agent

Eksperimen ini mengubah cara aku melihat pengoptimuman coding agent.

### Ukur sebelum membuang konteks

Jangan terus menganggap fail `AGENTS.md`, garis panduan atau koleksi skills yang besar menjadikan setiap request mahal.

Periksa rekod penggunaan sebenar dan bezakan input baharu daripada cache reads.

### Kekalkan konteks yang boleh digunakan semula

Prefix prompt yang stabil boleh memberi manfaat apabila provider menyokong prompt caching.

Elakkan perubahan yang tidak perlu pada arahan yang boleh digunakan semula. Namun, jangan korbankan ketepatan semata-mata untuk mengekalkan cache.

### Pantau output token

Dalam konfigurasi harga ini, output token mempunyai multiplier 12.5.

Output perantaraan yang terlalu panjang boleh meningkatkan kos tanpa manfaat yang setimpal. Walau bagaimanapun, kualiti kod, ketepatan analisis dan pengesahan hasil tetap perlu diutamakan.

### Siasat cache miss

Pantau pertukaran model, compaction dan pertumbuhan konteks. Gunakan rekod yang tersedia untuk menyiasat lonjakan penggunaan dan jangan menganggap semua cache miss mempunyai punca yang sama.

### Fahami maksud metrik pengebilan

Pastikan kita memahami bagaimana provider mentakrifkan input token, cache reads, output token dan caj minimum.

Jumlah input pada dashboard tidak semestinya mewakili input baharu yang dikenakan kadar penuh.

### Anggap kos paparan sebagai anggaran sehingga disahkan

Jika coding agent memaparkan nilai dalam USD, semak sama ada nilai itu datang daripada bil provider, pengiraan tempatan atau konfigurasi harga.

Bagi perkhidmatan prabayar, nyatakan kadar pek kredit apabila menukar penggunaan kredit kepada Ringgit.

### Nilai kos bersama kualiti

Konfigurasi yang menggunakan token lebih sedikit tidak semestinya lebih baik.

Persoalan yang lebih berguna ialah sama ada konfigurasi tersebut dapat menghasilkan keputusan pembangunan yang sama dengan kos lebih rendah, latency yang boleh diterima dan hasil yang boleh dipercayai.

## 10. Kesimpulan

Andaian awal aku ialah coding agent dengan 75 skills dan beberapa lapisan konteks mungkin menggunakan sekitar 20,000 input token baharu bagi setiap request.

Purata yang diukur ialah kira-kira 6,234 input token baharu, bersama-sama 48,069 cache-read token bagi setiap request.

Dalam snapshot 122 request, diskaun cache mengurangkan penggunaan kredit yang dikira sebanyak kira-kira 60.5% berbanding pengiraan hipotesis yang mengenakan kadar input baharu terhadap token cache yang sama.

System prompt pula berukuran kira-kira 6,550 token, dengan bahagian `skills` menjadi komponen terbesar.

Siasatan ini turut menunjukkan beberapa perkara penting: output token mempunyai multiplier yang tinggi, cache miss boleh menjadi mahal dan nilai dolar yang dipaparkan oleh PiG bukan bil USD sebenar yang disahkan oleh asAi.

Kesimpulan aku bukanlah konfigurasi dengan 75 skills sentiasa cekap atau prompt caching menjadikan pengoptimuman tidak diperlukan.

Sebaliknya, **kos AI coding agent perlu diukur menggunakan data penggunaan sebenar, ditafsir mengikut peraturan pengebilan provider dan dinilai bersama kualiti tugasan yang dihasilkan.**

Langkah seterusnya ialah membandingkan konfigurasi skills yang berbeza menggunakan workload yang boleh diulang. Aku mahu mengetahui sama ada pengurangan konteks awal benar-benar mengurangkan jumlah kredit tanpa menjejaskan keupayaan agent menyelesaikan tugasan pembangunan.

Itulah sasaran pengoptimuman yang lebih bermakna daripada sekadar cuba menjadikan prompt lebih pendek.

---

## Nota eksperimen

- Snapshot sesi 122 request dan snapshot dashboard 24 jam merupakan dua set data berasingan.
- Pengurangan 60.5% dikira daripada jumlah token dan multiplier harga yang digunakan. Ia bukan hasil eksperimen terkawal dengan caching dimatikan.
- Perbezaan kecil antara pengiraan agregat dengan jumlah kredit yang direkodkan belum disahkan melalui rekod pengebilan setiap request.
- Multiplier model dan kadar pek prabayar ialah kadar yang direkodkan bagi eksperimen pada 10 Oktober 2026. Semak dokumentasi provider sebelum mengulangi pengiraan.

## Rujukan

- [Harga asAi](https://serverasai.com/#pricing)
- [Dokumentasi pengebilan asAi](https://serverasai.com/docs/billing)
- [Harga model asAi](https://serverasai.com/docs/models)

**Tag:** `#ai` `#llm` `#devtools` `#programming`
