
**1. Pengenalan**

llama.cpp ialah runtime inference LLM berasaskan C/C++ yang popular kerana ringan, pantas, dan sesuai untuk menjalankan model GGUF secara local. Ia boleh digunakan pada:

Server x86_64
Workstation Linux
Mini PC
Raspberry Pi
Orange Pi
SBC ARM64
Container Linux

Dalam deployment sebenar, terdapat dua pendekatan utama:

Native build
Compile terus pada mesin yang akan menjalankan llama.cpp.
Cross compile
Compile pada mesin lebih laju (contohnya PC x86_64), tetapi menghasilkan binary untuk platform lain (contohnya ARM64 Orange Pi).

**Bahagian 1 — Persediaan Debian 12/13**

1.1 Install dependency asas
```bash
sudo apt update

sudo apt install -y \
    git \
    build-essential \
    cmake \
    ninja-build \
    pkg-config
```
Komponen utama:

Package	Fungsi
git	Ambil source code
build-essential	GCC, G++, make
cmake	Build configuration
ninja-build	Build engine lebih pantas
pkg-config	Cari library dependency

**Bahagian 2 — Clone llama.cpp**

```shell
git clone https://github.com/ggml-org/llama.cpp.git

cd llama.cpp
```
Semak versi:

```shell
git log -1 --oneline
```
**Bahagian 3 — Compile Native (Mesin Sama)**

Contoh:

Debian 12/13 x86_64
Debian ARM64
Orange Pi
Raspberry Pi

**3.1 Configure CMake**

```shell
Build menggunakan Ninja:

cmake -B build \
    -G Ninja \
    -DCMAKE_BUILD_TYPE=Release
3.2 Compile
ninja -C build -j$(nproc)
```
atau:
```console
cmake --build build
```plaintext
**3.3 Hasil build**

Semak:
```
ls build/bin
```plaintext
Contoh:
```
llama-cli
llama-server
llama-bench
llama-perplexity
```shell

**Bahagian 4 — Enable OpenBLAS (Pilihan)**

OpenBLAS boleh membantu operasi matrix CPU.

Install:
```
sudo apt install libopenblas-dev
```cmake
Build:
```
cmake -B build \
    -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DGGML_BLAS=ON \
    -DGGML_BLAS_VENDOR=OpenBLAS
```shell
Kemudian:
```shell
ninja -C build
```
Nota Penting: CMake Cache

Jika pernah configure dengan:
```shell
-DGGML_BLAS=ON
```
kemudian buang option tersebut, CMake masih menyimpan konfigurasi lama.

Contoh masalah:

BLAS not found
missing: BLAS_LIBRARIES

Penyelesaian:
```shell
rm -rf build
```

Kemudian configure semula.

Sentiasa ingat:

CMakeCache.txt menyimpan konfigurasi lama.
**Bahagian 5 — Cross Compile x86_64 → ARM64**

Contoh:

PC Debian 12 x86_64
        |
        |
        v
Orange Pi ARM64

Kelebihan:

Compile lebih cepat
Tidak membebankan SBC
Sesuai untuk production image

**5.1 Install ARM64 cross compiler**
```shell
sudo apt install -y \
    gcc-12-aarch64-linux--gnu\
    g++-12-aarch64-linux-gnu
```
```shell
sudo apt install -y \
    gcc-13-aarch64-linux--gnu\
    g++-13-aarch64-linux-gnu
```

Semak:
```shell
aarch64-linux-gnu-gcc --version
```

**5.2 Configure cross build**

Bersihkan dahulu:
```shell
rm -rf build-arm
```
Kemudian:
```cmake
cmake -B build-arm \
    -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_SYSTEM_NAME=Linux \
    -DCMAKE_SYSTEM_PROCESSOR=aarch64 \
    -DCMAKE_C_COMPILER=aarch64-linux-gnu-gcc \
    -DCMAKE_CXX_COMPILER=aarch64-linux-gnu-g++
5.3 Compile
ninja -C build-arm -j$(nproc)
```
Hasil:
```shell
ls build-arm/bin
```

**Bahagian 6 — Semak Architecture Binary**

Gunakan:
```shell
file build-arm/bin/llama-server
```
Contoh output berjaya:
```plaintext
ELF 64-bit LSB pie executable,
ARM aarch64,
dynamically linked
```
Maksud:

Output	Maksud
ELF 64-bit	Binary 64-bit
ARM aarch64	Untuk ARM64
dynamically linked	Perlukan shared library
PIE executable	Linux security hardening
Bahagian 7 — Semak Dependency .so
Jangan guna ldd untuk cross binary

Jika compile ARM64 tetapi check pada PC x86:
```shell
ldd llama-server
```
boleh gagal:
```plaintext
not a dynamic executable
```
Sebab:

PC:

x86_64 loader

Binary:

ARM64 loader
Gunakan readelf
aarch64-linux-gnu-readelf \
-d build-arm/bin/llama-server | grep NEEDED

Contoh:
```plaintext
Shared library: [libllama.so]
Shared library: [libggml.so]
Shared library: [libstdc++.so.6]
```
Cari semua .so
```shell
find build-arm -name "*.so"
```
Contoh:
```plaintext
libllama.so
libggml.so
libggml-base.so
libggml-cpu.so
```
Semak architecture:
```shell
file build-arm/bin/*.so
```
Output:
```plaintext
ARM aarch64
Bahagian 8 — Dynamic vs Static Binary
```
Semak:
```shell
file llama-server
```
Contoh dynamic:
```plaintext
dynamically linked
```
Perlu:

lib*.so

Contoh static:

statically linked

Tidak perlu .so.

**Bahagian 9 — Installation ke Linux**
Pilihan standard

Binary:
```plaintext
/usr/local/bin
```
Library:
```plaintext
/usr/local/lib
```
Contoh:
```shell
sudo cp llama-server /usr/local/bin/
sudo cp llama-cli /usr/local/bin/

sudo cp *.so /usr/local/lib/

sudo ldconfig
```

Pilihan appliance / embedded

Untuk SBC:
```plaintext
/opt/llama.cpp/

    llama-server
    llama-cli
    libllama.so
    libggml.so
```
Kemudian:
```shell
export LD_LIBRARY_PATH=/opt/llama.cpp
```
Sesuai untuk:

Orange Pi
kiosk AI
edge inference node

**Bahagian 10 — Deploy ke Orange Pi**

Copy:
```shell
scp build-arm/bin/llama-server \
orangepi:/usr/local/bin/
```
```shell
scp build-arm/bin/llama-cli \
orangepi:/usr/local/bin/
```
Jika perlu:
```shell
scp build-arm/bin/*.so \
orangepi:/usr/local/lib/
```
Pada Orange Pi:
```shell
sudo ldconfig
```
Semak:
```shell
uname -m
```
Expected:
```plaintext
aarch64
```

**Bahagian 11 — Cadangan Production Architecture**

Untuk sistem AI agent:

+----------------+
| Go Agent       |
| Tool Router    |
+-------+--------+
        |
        |
 HTTP API
        |
        v
+----------------+
| llama-server   |
| llama.cpp      |
+----------------+
        |
        |
      GGUF
      Model

Kelebihan:

Go agent tidak perlu embed model
Model boleh tukar tanpa rebuild
llama.cpp boleh upgrade sendiri
Mudah scale ke banyak node
Kesimpulan

Workflow yang stabil:

Native
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release

ninja -C build
Cross Compile ARM64
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu

rm -rf build-arm

cmake -B build-arm \
-G Ninja \
-DCMAKE_SYSTEM_NAME=Linux \
-DCMAKE_SYSTEM_PROCESSOR=aarch64 \
-DCMAKE_C_COMPILER=aarch64-linux-gnu-gcc \
-DCMAKE_CXX_COMPILER=aarch64-linux-gnu-g++

ninja -C build-arm
Verification
file llama-server

aarch64-linux-gnu-readelf -d llama-server | grep NEEDED

find . -name "*.so"

Dengan proses ini, satu mesin Debian 12/13 boleh menjadi build server untuk menghasilkan node AI ARM64 seperti Orange Pi, Raspberry Pi, atau edge inference appliance.
