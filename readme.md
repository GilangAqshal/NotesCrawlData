# 🐦 Crawl Data Twitter — Tweet Harvest

> Scraping tweet menggunakan **tweet-harvest** di Google Colab, lengkap dengan analisis data menggunakan Pandas dan ekspor ke JSON.

---

## 📋 Daftar Isi

- [Tentang Proyek](#-tentang-proyek)
- [Tech Stack](#-tech-stack)
- [Prasyarat](#-prasyarat)
- [Struktur Folder](#-struktur-folder)
- [Cara Penggunaan](#-cara-penggunaan)
- [Parameter Scraping](#-parameter-scraping)
- [Output Data](#-output-data)
- [Catatan Penting](#-catatan-penting)

---

## 📌 Tentang Proyek

Proyek ini merupakan pipeline sederhana untuk mengumpulkan data tweet dari Twitter/X menggunakan library **tweet-harvest** yang berjalan di atas **Node.js** dan **Playwright Chromium**. Data yang terkumpul kemudian diolah menggunakan **Pandas** dan diekspor ke format CSV maupun JSON untuk keperluan analisis lebih lanjut (misalnya: analisis sentimen, NLP, atau visualisasi data).

---

## 🛠 Tech Stack

| Tool          | Versi  | Fungsi                      |
| ------------- | ------ | --------------------------- |
| Python        | 3.x    | Bahasa utama & Pandas       |
| Node.js       | 20.x   | Runtime tweet-harvest       |
| tweet-harvest | 2.6.1  | Scraping Twitter/X          |
| Playwright    | latest | Headless browser (Chromium) |
| Pandas        | latest | Manipulasi & ekspor data    |
| Google Colab  | —      | Lingkungan eksekusi         |

---

## ✅ Prasyarat

Sebelum menjalankan notebook, pastikan kamu memiliki:

- Akun Google untuk mengakses **Google Colab**
- **Twitter Auth Token** yang masih aktif
- Koneksi internet yang stabil

### Cara Mendapatkan Twitter Auth Token

1. Login ke [twitter.com](https://twitter.com) di browser
2. Buka **DevTools** → tab **Application** → **Cookies**
3. Cari cookie bernama `auth_token`
4. Salin nilainya dan tempel ke variabel `twitter_auth_token`

---

## 📁 Struktur Folder

```
/content/
└── tweets-data/
    ├── piala_dunia_2026.csv      # Hasil scraping (semua kolom)
    ├── piala_dunia_2026.json     # Ekspor JSON (semua kolom)
    └── just_text_piala_dunia.json  # Ekspor JSON (full_text only)
```

---

## 🚀 Cara Penggunaan

Jalankan setiap cell secara berurutan di Google Colab.

### Bagian 1 — Setup Lingkungan

**Langkah 1: Install Pandas**

```python
!pip install pandas
```

**Langkah 2: Konfigurasi Auth Token**

```python
twitter_auth_token = 'GANTI_DENGAN_TOKEN_ANDA'
```

**Langkah 3: Install Node.js v20**

```bash
!sudo apt-get update
!sudo apt-get install -y ca-certificates curl gnupg
!sudo mkdir -p /etc/apt/keyrings
!curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
!NODE_MAJOR=20 && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] \
  https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" \
  | sudo tee /etc/apt/sources.list.d/nodesource.list
!sudo apt-get update && sudo apt-get install nodejs -y
!node -v
```

**Langkah 4: Install Dependensi Browser (Linux GUI)**

```bash
!apt-get install -y \
  libatk1.0-0 libatk-bridge2.0-0 libgtk-3-0 \
  libgbm1 libasound2 libnss3 libxss1 libxtst6 \
  libxrandr2 libxcomposite1 libxdamage1 libxfixes3 \
  libdrm2 libxkbcommon0 libpango-1.0-0 \
  libpangocairo-1.0-0 libcairo2 libatspi2.0-0
```

**Langkah 5: Install Playwright Chromium**

```bash
!npx playwright install chromium
```

---

### Bagian 2 — Proses Scraping

**Langkah 6: Eksekusi Scraping**

```python
filename       = "piala_dunia_2026.csv"
search_keyword = '"Piala Dunia" lang:id'
limit          = 1000

!npx -y tweet-harvest@2.6.1 \
  -o "{filename}" \
  -s "{search_keyword}" \
  --tab "LATEST" \
  -l {limit} \
  --token {twitter_auth_token}
```

---

### Bagian 3 — Analisis Data & Ekspor

**Langkah 7: Baca Dataset CSV**

```python
import pandas as pd

df = pd.read_csv('/content/tweets-data/piala_dunia_2026.csv')
print("Ukuran Data:", df.shape)
df.head()
```

**Langkah 8: Cek Nama Kolom**

```python
print(df.columns)
```

**Langkah 9: Filter Kolom `full_text`**

```python
tweet_df = df[['full_text']]
tweet_df.head()
```

**Langkah 10: Ekspor ke JSON**

```python
# Opsi 1: Semua kolom
df.to_json('/content/tweets-data/piala_dunia_2026.json', orient='records', indent=4)

# Opsi 2: Hanya full_text (hapus '#' untuk aktifkan)
# tweet_df.to_json('/content/tweets-data/just_text_piala_dunia.json', orient='records', indent=4)

print("File JSON berhasil disimpan!")
```

---

## ⚙️ Parameter Scraping

| Parameter        | Nilai Default           | Keterangan                       |
| ---------------- | ----------------------- | -------------------------------- |
| `filename`       | `piala_dunia_2026.csv`  | Nama file output CSV             |
| `search_keyword` | `"Piala Dunia" lang:id` | Kata kunci pencarian Twitter     |
| `limit`          | `1000`                  | Jumlah maksimum tweet            |
| `--tab`          | `LATEST`                | Tab pencarian (`LATEST` / `TOP`) |
| `--token`        | _(dari variabel)_       | Twitter auth token               |

---

## 📦 Output Data

Kolom-kolom yang tersedia pada hasil scraping tweet-harvest antara lain:

| Kolom                  | Tipe     | Deskripsi              |
| ---------------------- | -------- | ---------------------- |
| `full_text`            | string   | Isi teks lengkap tweet |
| `created_at`           | datetime | Waktu tweet dibuat     |
| `user_name`            | string   | Username akun          |
| `user_followers_count` | int      | Jumlah followers       |
| `reply_count`          | int      | Jumlah balasan         |
| `retweet_count`        | int      | Jumlah retweet         |
| `like_count`           | int      | Jumlah like            |
| `lang`                 | string   | Bahasa tweet           |

> Kolom aktual dapat bervariasi tergantung versi tweet-harvest. Gunakan `df.columns` untuk melihat daftar lengkap.

---

## ⚠️ Catatan Penting

- **Auth token bersifat rahasia** — jangan share atau commit ke repositori publik.
- Token dapat kedaluwarsa; perbarui jika scraping gagal dengan error autentikasi.
- Setiap sesi Google Colab baru memerlukan instalasi ulang dari Langkah 1.
- Gunakan secara bertanggung jawab dan patuhi [Ketentuan Layanan Twitter/X](https://twitter.com/en/tos).
- Scraping dalam jumlah besar berpotensi memicu rate limit dari Twitter.

---

## 📄 Lisensi

Proyek ini dibuat untuk keperluan edukasi dan riset. Gunakan dengan bijak.
