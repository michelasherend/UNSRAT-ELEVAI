# UNSRAT-ELEVAI

Aplikasi Asisten Akademik Digital Berbasis Large Language Model (LLM) Universitas Sam Ratulangi.
UNSRAT-ELEVAI adalah chatbot asisten akademik yang berjalan secara on-premise, dibangun menggunakan model dari ollama dengan arsitektur Retrieval-Augmented Generation (RAG). Sistem ini dirancang untuk menjawab pertanyaan seputar informasi akademik UNSRAT secara akurat sekaligus meminimalkan halusinasi jawaban.
## Arsitektur & Teknologi
Model LLM: Menggunakan model Ollama
Antarmuka: Open-WebUI
Kontainerisasi: Docker & Docker Compose
Arsitektur: Retrieval-Augmented Generation (RAG)
Autentikasi: Google OAuth (dibatasi domain UNSRAT)

## Prasyarat
1. Pastikan perangkat berikut sudah terinstall di komputer:
2. Docker dan Docker Compose
Ollama (dijalankan langsung di komputer host, bukan di dalam Docker)
3. Git
   
## Cara Menjalankan Proyek
1. Clone repository ini:
git clone https://github.com/michelasherend/UNSRAT-ELEVAI.git
cd UNSRAT-ELEVAI

2. Siapkan Ollama dan unduh model
Pastikan Ollama sudah berjalan, lalu unduh model yang sesuai
ollama pull llama3.1:8b (contoh model)

3. Siapkan file konfigurasi (.env)
Salin file contoh environment, lalu isi dengan nilai kamu sendiri:
cp .env.example .env
Buka file .env dan lengkapi kredensial Google OAuth sesuai konfigurasi di Google Cloud Console:

OAUTH_CLIENT_ID — Client ID dari Google Cloud Console
OAUTH_CLIENT_SECRET — Client Secret dari Google Cloud Console
4. Buat Docker volume
Volume pada proyek ini dikonfigurasi sebagai external, sehingga harus dibuat terlebih dahulu sebelum menjalankan container:
bash
docker volume create open-webui

5. Jalankan aplikasi
bash
docker compose up -d

6. Akses aplikasi
Buka browser dan kunjungi:
http://localhost:3000
Login menggunakan akun Google dengan domain @student.unsrat.ac.id atau @unsrat.ac.id.

## Konfigurasi RAG

Pengaturan RAG — seperti Top K retrieval, reranking, dan sumber dokumen — dikonfigurasi melalui antarmuka Open-WebUI pada menu Settings → Documents. Dokumen basis pengetahuan diunggah melalui menu Workspace → Knowledge.

## Perintah Berguna

Menghentikan aplikasi:

bash
docker compose down

Melihat log:

bash
docker compose logs -f
## Penulis

## Michella Sheeren Talumewo Teknik Informatika, Fakultas Teknik — Universitas Sam Ratulangi.
