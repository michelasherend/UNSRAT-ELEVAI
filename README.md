# UNSRAT-ELEVAI

Aplikasi Asisten Akademik Digital Berbasis Large Language Model (LLM) Universitas Sam Ratulangi.

UNSRAT-ELEVAI adalah chatbot asisten akademik yang berjalan secara on-premise, dibangun menggunakan model dari ollama dengan arsitektur Retrieval-Augmented Generation (RAG). Sistem ini dirancang untuk menjawab pertanyaan seputar informasi akademik UNSRAT secara akurat sekaligus meminimalkan halusinasi jawaban.
## Arsitektur & Teknologi
Model LLM: Menggunakan model Ollama
Antarmuka: Open-WebUI
Kontainerisasi: Docker & Docker Compose
Arsitektur: Retrieval-Augmented Generation (RAG)
Autentikasi: Google OAuth (dibatasi domain UNSRAT)

## Arsitektur Sistem
 
| Komponen | Teknologi | Peran |
|----------|-----------|-------|
| Sistem Operasi | Linux (Ubuntu) | Lingkungan utama, mengelola resource & menjalankan Docker/Ollama |
| LLM Runtime | Ollama | Mengunduh, menjalankan & menyediakan API untuk model lokal |
| Model Bahasa | Llama 3.1 8B | Model baseline penghasil jawaban |
| Model Embedding | bge-m3 | Konversi teks dokumen menjadi vektor untuk RAG |
| Kontainerisasi | Docker | Isolasi & deployment Open-WebUI |
| Antarmuka | Open-WebUI | UI web + backend interaksi pengguna |
| Version Control | GitHub | Manajemen versi & dokumentasi |
 
**Alur singkat:** Pengguna → Open-WebUI (container Docker, port `3000`) → endpoint Ollama di host (`172.17.0.1:11434`) → Llama 3.1 8B + retrieval vektor (bge-m3) → jawaban ter-*grounding*.
 
---
 
## Prasyarat
 
**Perangkat keras (minimum yang diuji):**
- RAM 16 GB DDR5 (model Llama 3.1 8B ≈ 4,9 GB memori)
- Penyimpanan cukup untuk model + container
- GPU opsional (mempercepat inferensi; tanpa GPU model dimuat ke RAM/CPU)
**Perangkat lunak:**
- Linux / Ubuntu
- [Ollama](https://ollama.com)
- [Docker](https://docs.docker.com/get-docker/)
---
 
## Instalasi & Menjalankan dari Awal
 
Ikuti tahapan berikut secara berurutan.
 
### 1. Persiapan Lingkungan Linux
 
Perbarui daftar paket dan versi perangkat lunak agar lingkungan stabil sebelum instalasi komponen utama.
 
```bash
sudo apt update && sudo apt upgrade
```
 
### 2. Instalasi & Konfigurasi Ollama
 
Pasang Ollama (lihat [petunjuk resmi](https://ollama.com/download)), lalu unduh model bahasa dan model embedding.
 
```bash
# Unduh model bahasa utama (sesuaikan model ollama dengan perangkat/server yang digunakan)
ollama pull llama3.1:8b
 
# Unduh model embedding untuk RAG (mendukung Bahasa Indonesia)
ollama pull bge-m3
```
 
Uji model dan pastikan sudah tersedia secara lokal:
 
```bash
# Menjalankan model (menggabungkan pulling + serving)
ollama run llama3.1:8b
 
# Verifikasi daftar model lokal — pastikan llama3.1:8b dan bge-m3 muncul
ollama list
```
 
Jika `llama3.1:8b` dan `bge-m3` tampil pada daftar, model siap digunakan Open-WebUI melalui endpoint Ollama.
 
### 3. Instalasi & Konfigurasi Docker
 
Pastikan Docker sudah terpasang dan aktif.
 
```bash
# Cek versi Docker
docker --version
 
# Cek container yang sedang berjalan
sudo docker ps
```
 
### 4. Deployment Open-WebUI melalui Docker
 
Jalankan Open-WebUI sebagai container yang terhubung ke Ollama di host.
 
```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```
 
Penjelasan parameter:
- `-d` — jalankan container di latar belakang (*detached*).
- `-p 3000:8080` — petakan port `3000` host ke port `8080` di dalam container.
- `--add-host=host.docker.internal:host-gateway` — membangun jembatan jaringan agar container dapat berkomunikasi dengan Ollama di host.
- `-v open-webui:/app/backend/data` — volume persisten untuk data akun & riwayat obrolan.
- `--restart always` — container otomatis menyala kembali setelah reboot/gagal.
Verifikasi container berjalan:
 
```bash
docker ps
```
 
### 5. Integrasi Open-WebUI dengan Ollama
 
Buka Open-WebUI di browser (`http://localhost:3000`), lalu arahkan endpoint Ollama ke:
 
```
http://172.17.0.1:11434
```
 
> **Penting:** alamat `172.17.0.1` adalah *bridge gateway* Docker. Dari dalam container, `localhost` menunjuk ke container itu sendiri — bukan ke host. Karena Ollama berjalan di **host**, endpoint harus memakai IP gateway Docker ini, bukan `127.0.0.1`.
 
### 6. Konfigurasi Akun Pengguna & Administrator
 
- Akun pertama yang mendaftar otomatis menjadi **Administrator**.
- Administrator mengelola pengguna via **Admin Panel > Users > Overview**, termasuk pengaturan peran (RBAC: User/Admin), profil, dan reset kata sandi.
### 7. Konfigurasi System Prompt
 
Atur *system prompt* pada pengaturan model agar UNSRAT-ELEVAI bertindak sebagai asisten akademik (Bahasa Indonesia formal, anti-halusinasi referensi, format Markdown terstruktur).
 
<details>
<summary>Lihat System Prompt lengkap</summary>
```
PERAN DAN IDENTITAS
- Nama Asisten: UNSRAT-ELEVAI
- Deskripsi: Asisten AI Akademik Resmi untuk Universitas Sam Ratulangi (UNSRAT).
- Karakter: Profesional, cerdas, berwawasan luas, ramah, suportif, dan berdedikasi tinggi.
 
TUGAS UTAMA DAN TANGGUNG JAWAB
- Dukungan Akademik: Menjelaskan konsep rumit dari berbagai disiplin ilmu secara sistematis menggunakan analogi yang mudah dipahami tanpa mengurangi bobot ilmiah.
- Asistensi Penelitian dan Penulisan: Membimbing penyusunan karya ilmiah (tugas, makalah, laporan praktikum, proposal, skripsi, tesis, materi presentasi) dengan memberikan kerangka struktur, saran perbaikan tata bahasa, dan tinjauan literatur.
- Panduan Terstruktur: Memberikan jawaban yang logis, metodologis, dan sesuai standar penulisan ilmiah.
 
PEDOMAN ETIKA DAN INTEGRITAS AKADEMIK
- Anti-Halusinasi Referensi: JANGAN PERNAH mengarang jurnal, buku, atau nama penulis (kutipan palsu). Jika referensi tidak diketahui pasti, berikan rekomendasi kata kunci pencarian yang efektif untuk Google Scholar, Scopus, atau perpustakaan kampus.
- Verifikasi Fakta: Utamakan keakuratan data daripada kecepatan menjawab. Jika informasi terkait kebijakan kampus atau data spesifik belum pasti, nyatakan secara jujur bahwa pengguna perlu memverifikasinya kembali ke pihak fakultas atau universitas.
 
ATURAN BAHASA DAN GAYA KOMUNIKASI
- Bahasa Utama: Gunakan Bahasa Indonesia yang baku, formal, akademis, dan profesional.
- Adaptasi Bahasa: Menyesuaikan diri secara otomatis dengan bahasa yang digunakan pengguna (contoh: jika pengguna bertanya dalam Bahasa Inggris, jawab penuh dalam Bahasa Inggris).
- Larangan Campur Bahasa: Jangan mencampur istilah Bahasa Indonesia dengan Bahasa Inggris di dalam tanda kurung, kecuali untuk istilah teknis yang belum memiliki padanan kata.
- Bebas Tipografi: Pastikan tidak ada kesalahan ketik (typo) dalam setiap hasil generasi teks.
- Nada Bicara: Tetap ramah, suportif, santun, dan memotivasi.
 
ATURAN FORMAT OUTPUT (MARKDOWN)
- Dilarang Dinding Teks: Jangan pernah memberikan jawaban dalam paragraf tunggal yang terlalu panjang dan raksasa.
- Struktur: Gunakan penomoran bab atau judul yang jelas untuk memisahkan topik pembahasan.
- Penekanan: Gunakan format teks tebal (bold) untuk kata kunci, definisi, atau poin krusial.
- Daftar: Gunakan poin-poin (bullet points) atau daftar bernomor untuk penjelasan prosedural.
- Tabel: Gunakan format tabel jika sedang membandingkan data, teori, atau konsep.
```
 
</details>
### 8. Pengaturan Hyperparameter Inferensi
 
Pada pengaturan model, sesuaikan hyperparameter agar respons stabil dan sesuai konteks akademik:
 
| Parameter | Nilai | Keterangan |
|-----------|-------|------------|
| `temperature` | `0.3` | Rendah agar jawaban konsisten & tidak acak |
| `context length` | `4096` | Disesuaikan dengan RAM 16 GB agar memori stabil |
| `top-p` | *default* | Mengontrol distribusi pilihan token |
 
> Sesuaikan `context length` dengan kapasitas RAM/VRAM perangkat Anda. Nilai terlalu besar dapat membuat sistem tidak stabil.
 
### 9. Konfigurasi RAG & Unggah Basis Pengetahuan
 
**a. Atur mesin embedding** — buka **Settings > Documents**:
- *Embedding Model Engine*: **Ollama**
- *Embedding Model*: **bge-m3** (`bge-m3:latest`)
**b. Buat basis pengetahuan** — buka **Workspace > Knowledge**:
- Buat koleksi baru (mis. `UNSRAT-ELEVAI`).
- Unggah dokumen resmi (mis. `peraturan akademik.pdf` — Peraturan Rektor UNSRAT No. 01 Tahun 2025).
Setelah diunggah, sistem otomatis menjalankan pipeline *ingestion*: **chunking** (pemotongan teks) → **vektorisasi** (via bge-m3) → **penyimpanan indeks** ke vector store. Dokumen kini siap dipanggil (*retrieved*) saat pengguna bertanya.
 
### 10. (Opsional) Google OAuth Domain UNSRAT
 
Agar registrasi dibatasi pada akun email resmi UNSRAT ("Daftar dengan Google Unsrat"), aktifkan Google OAuth pada Open-WebUI dengan menyediakan kredensial melalui *environment variable* (mis. `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, serta pembatasan domain email).
 
> **Keamanan:** JANGAN pernah commit kredensial OAuth asli ke Git. Simpan di file `.env` yang di-*ignore*, dan gunakan `.env.example` hanya berisi placeholder.
 
---
 
## Mengakses Aplikasi
 
Buka browser dan akses:
 
```
http://localhost:3000
```
 
Registrasi/login → mulai bertanya. Jawaban berbasis dokumen akan menampilkan *badge* sumber rujukan di bawah respons.
 
---
 
## Catatan Penting (Non-Obvious)
 
- **Ollama berjalan di host, bukan di dalam container.** Endpoint yang dipakai Open-WebUI adalah `http://172.17.0.1:11434` (bridge gateway Docker).
- **Volume `open-webui`** menyimpan data akun & riwayat secara persisten — jangan dihapus bila ingin mempertahankan data.
- **`bge-m3` wajib di-pull** sebelum RAG bisa berjalan; tanpa model embedding, proses vektorisasi dokumen gagal.
- **Model embedding harus diset ke Ollama + bge-m3** di Settings > Documents, bukan model bawaan yang berorientasi Bahasa Inggris.

 
## Penulis
 
**Michella Sheeren Talumewo**
Teknik Informatika, Fakultas Teknik — Universitas Sam Ratulangi.
