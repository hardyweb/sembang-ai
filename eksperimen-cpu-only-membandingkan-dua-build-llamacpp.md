Aku sangat berminat dengan local LLM, terutamanya bila model bahasa boleh digunakan pada komputer biasa tanpa GPU khusus.

Bagi aku, dok semua orang mempunyai workstation dengan GPU mahal. Ada yang hanya menggunakan laptop atau komputer lama. Kalau model masih boleh digunakan untuk menulis dokumen, menyediakan surat rasmi, membantu kerja pengaturcaraan mudah dan menjawab soalan teknikal, itu pun sudah cukup menarik.

Kali ini aku cuba membandingkan dua build `llama.cpp` menggunakan model Gemma 4 E2B dengan quantization Q4_K_XL.

Tujuan eksperimen ini bukan untuk menghasilkan benchmark yang sempurna. Aku cuma mahu melihat sama ada build yang lebih baharu memberikan perbezaan prestasi yang boleh diperhatikan pada perkakasan CPU-only.

## Setup eksperimen

Untuk perbandingan ini, aku menggunakan dua build:

| Perkara | Build lama | Build baharu |
|---|---|---|
| Build number | 9976 | 11571 |
| Commit | `e3546c794` | `097cf33d9` |
| Backend | CPU-only | CPU-only |
| Model | Gemma 4 E2B Q4_K_XL | Gemma 4 E2B Q4_K_XL |
| Model execution | CPU | CPU |

Build baharu menggunakan `llama.cpp` versi `0.6.0-dev`, dibina daripada source dengan GNU 12.2.0 untuk Linux x86_64.

Model yang digunakan ialah:

`gemma-4-E2B-it-qat-UD-Q4_K_XL.gguf`

Aku sengaja mengekalkan model yang sama supaya perbandingan lebih tertumpu kepada perubahan antara kedua-dua build, bukannya perbezaan model atau quantization.

## Parameter benchmark

Untuk ujian ini, aku menggunakan konfigurasi berikut:

```bash
-t 3 -tb 4 -c 4128 -b 128 -ub 32 -np 1 -ngl 0 \
--temp 0.7 --top-p 0.8 --top-k 20 \
--metrics --jinja \
--cache-type-k q8_0 --cache-type-v q8_0 \
--reasoning-format none \
--host 127.0.0.1 --port 5002
```

Beberapa perkara yang aku cuba kekalkan:

- CPU-only, tanpa offloading ke GPU.
- Tiga thread untuk inference dan empat thread untuk batch processing.
- Context size 4128.
- Satu parallel slot.
- Tetapan sampling yang sama.
- Restart server sebelum setiap run.
- `cache_n: 0` bagi setiap sampel yang direkodkan.
- Prompt dan had output yang sama, iaitu 256 token.

Prompt yang digunakan:

> Explain three practical ways to improve Linux server security. Give a concise explanation for each.

Setiap run menghasilkan 256 token. Ini membolehkan aku membandingkan masa generation untuk jumlah output yang sama.

## Keputusan benchmark

Aku menjalankan tiga sampel bagi setiap build.

### Build lama: 9976

| Run | Prompt tok/s | Generation tok/s | Generation time |
|---|---:|---:|---:|
| 1 | 10.78 | 6.45 | 39.71 s |
| 2 | 12.67 | 5.41 | 47.30 s |
| 3 | 12.30 | 6.29 | 40.69 s |
| **Purata** | **11.91** | **6.05** | **42.57 s** |

### Build baharu: 11571

| Run | Prompt tok/s | Generation tok/s | Generation time |
|---|---:|---:|---:|
| 1 | 13.45 | 6.82 | 37.42 s |
| 2 | 10.62 | 6.39 | 39.93 s |
| 3 | 9.36 | 6.61 | 38.58 s |
| **Purata** | **11.14** | **6.60** | **38.64 s** |

Angka purata dikira daripada tiga run bagi setiap build. Masa generation ialah masa yang dilaporkan oleh `llama.cpp` untuk menghasilkan 256 token.

## Apa yang aku perhatikan?

### 1. Generation bertambah baik

Purata kelajuan generation berubah daripada sekitar 6.05 tok/s kepada 6.60 tok/s.

Ini bersamaan peningkatan kira-kira 9.1%.

Dari sudut penggunaan harian, build baharu mengambil masa sekitar 38.64 saat untuk menghasilkan 256 token, berbanding 42.57 saat bagi build lama.

Perbezaan ini bukanlah perubahan yang akan mengubah segala-galanya. Namun, pada komputer tanpa GPU khusus, beberapa saat yang berjaya dijimatkan tetap menarik untuk diperhatikan.

### 2. Prompt processing bukan lebih laju dalam ujian ini

Purata prompt processing turun daripada 11.91 tok/s kepada 11.14 tok/s, iaitu kira-kira 6.5% lebih perlahan.

Ini mengingatkan aku bahawa prestasi LLM bukan sekadar satu angka.

Prompt processing dan text generation merupakan dua bahagian kerja yang berbeza. Satu build boleh menunjukkan peningkatan pada generation tetapi tidak semestinya pada prompt processing.

Oleh itu, aku tidak mahu membuat kesimpulan bahawa build baharu lebih pantas untuk semua workload hanya berdasarkan satu benchmark ringkas ini.

### 3. Prestasi berubah antara run

Walaupun konfigurasi dikekalkan, keputusan setiap run tidak sama.

Sebagai contoh, generation build lama berada antara 5.41 hingga 6.45 tok/s. Build baharu pula berada antara 6.39 hingga 6.82 tok/s.

Sebab itulah aku menjalankan tiga sampel dan membandingkan purata, bukannya memilih satu run yang paling laju.

Walau bagaimanapun, tiga sampel masih belum mencukupi untuk membuat kesimpulan statistik yang kukuh. Ujian ini lebih sesuai dianggap sebagai pemerhatian praktikal pada mesin dan konfigurasi yang aku gunakan.

## Bukan prestasi sahaja: feature baharu

Satu lagi perkara yang menarik perhatian aku ketika menggunakan build baharu ialah sokongan untuk *load model on demand*.

Feature seperti ini menarik untuk diterokai kerana pengurusan model juga penting apabila menjalankan local LLM pada komputer yang mempunyai sumber terhad.

Namun, aku belum mengukur masa pemuatan model, penggunaan RAM ketika idle atau tingkah laku pelepasan memori dalam eksperimen ini.

Jadi, aku tidak akan mendakwa feature tersebut sudah terbukti menjimatkan RAM berdasarkan keputusan benchmark tadi.

Untuk mengujinya dengan betul, aku perlu membuat eksperimen berasingan yang mengukur:

- Masa server bermula.
- Masa model dimuatkan apabila diperlukan.
- Penggunaan RAM sebelum dan selepas model dimuatkan.
- Tingkah laku server apabila model tidak digunakan.
- Masa yang diperlukan apabila model diminta semula.

Ini semua perkara yang berbeza daripada kelajuan inference.

## Keputusan: aku kekalkan build baharu

Berdasarkan eksperimen ringkas ini, aku memilih untuk meneruskan penggunaan build `llama.cpp` yang lebih baharu.

Sebabnya bukan semata-mata peningkatan generation sekitar 9.1%. Aku juga mahu meneroka perkembangan projek ini, termasuk feature baharu yang boleh membantu penggunaan local LLM pada komputer biasa.

Aku tidak menjalankan semua kemungkinan ujian. Aku juga tidak membandingkan semua backend atau mencuba setiap pilihan konfigurasi.

Buat masa ini, aku rasa lebih baik menggunakan build tersebut untuk kerja sebenar dan melihat bagaimana ia berkelakuan ketika digunakan untuk menyediakan dokumen, menulis surat rasmi dan menjalankan tugasan harian.

Benchmark ada nilainya, tetapi pengalaman penggunaan sebenar juga penting.

## Kesimpulan

Eksperimen ini memberikan aku tiga pemerhatian:

1. Build `llama.cpp` yang lebih baharu menunjukkan peningkatan purata generation dalam ujian ini.
2. Peningkatan itu tidak berlaku pada semua metrik; prompt processing sedikit perlahan.
3. Feature baharu seperti load model on demand wajar diterokai secara berasingan, terutama bagi pengguna yang mempunyai sumber perkakasan terhad.

Aku masih mahu menggunakan local LLM secara praktikal, bukan sekadar mengejar angka benchmark.

Kalau komputer biasa boleh membantu kita menulis dokumen, belajar teknologi dan menjalankan tugasan harian tanpa bergantung sepenuhnya pada perkhidmatan cloud, itu sudah satu kelebihan.

Dok semua orang ada GPU mahal untuk mula bereksperimen dengan AI secara local.


penulisan ini dibantu oleh AI
