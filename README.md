# ESP-01 Relay Control 🔌

Firmware untuk mengontrol modul relay menggunakan ESP-01 (ESP8266) melalui halaman web, tanpa memerlukan koneksi internet. ESP-01 bekerja dalam mode **Access Point (AP)** — HP atau laptop terhubung langsung ke WiFi yang dipancarkan ESP-01, dan halaman kontrolnya sudah tersimpan di dalam firmware itu sendiri.

![Platform](https://img.shields.io/badge/platform-ESP8266-blue)
![Board](https://img.shields.io/badge/board-ESP--01-orange)
![License](https://img.shields.io/badge/license-MIT-green)

---

## 🛒 Link Pembelian

Beli ESP01 dan USB to TTL programmer di sini:
👉 [Shopee - Link Pembelian Terpercaya](https://s.shopee.co.id/1gIPv5W8xi)

## ✨ Fitur

- 📶 Mode Access Point — tidak butuh router/internet, HP langsung konek ke ESP-01
- 🖥️ Halaman web tersimpan di dalam firmware (tidak perlu SPIFFS/LittleFS eksternal)
- 🔘 Kontrol relay ON/OFF dengan satu tombol bulat besar, mudah dipakai di layar HP
- 🎨 4 pilihan skin warna: **Light, Dark, Green, Orange** — pilihan tema tersimpan otomatis di browser
- 🖼️ Slot logo custom di halaman web (disematkan sebagai Base64, tanpa perlu hosting gambar)
- ⚡ Mendukung modul relay **active LOW**
- 🔄 Status relay otomatis tersinkron saat halaman dibuka ulang (endpoint `/status`)

---

## 🧰 Kebutuhan Hardware

| Komponen              | Keterangan                          |
|------------------------|--------------------------------------|
| ESP-01 (ESP8266)      | Modul WiFi utama                     |
| USB to TTL             | Untuk mem-flash firmware ke ESP-01   |
| Modul relay active LOW | Terhubung ke GPIO2                   |
| Power supply 3.3V      | Sesuai kebutuhan arus ESP-01 & relay |

> ⚠️ ESP-01 butuh suplai arus yang stabil (idealnya sumber 3.3V terpisah, bukan dari pin 3.3V USB-TTL yang biasanya arusnya kecil), terutama saat WiFi aktif.

---

## 🔌 Skema Pengkabelan

### Saat Upload Firmware (via USB to TTL)

| ESP-01     | USB to TTL |
|------------|------------|
| VCC        | 3.3V       |
| GND        | GND        |
| TX         | RX         |
| RX         | TX         |
| CH_PD (EN) | 3.3V       |
| GPIO0      | GND *(wajib untuk mode flashing)* |
| GPIO2      | *(lepas sementara, jangan disambung ke relay dulu)* |

### Saat Operasi Normal (setelah firmware ter-upload)

| ESP-01     | Keterangan                          |
|------------|--------------------------------------|
| VCC        | 3.3V                                 |
| GND        | GND (disatukan dengan GND relay)     |
| CH_PD (EN) | 3.3V                                 |
| GPIO0      | Dibiarkan mengambang / ke 3.3V (opsional, tidak disambung ke GND lagi) |
| GPIO2      | ke pin **IN** modul relay             |

---

## 🚀 Cara Upload Firmware

1. Buka **Arduino IDE**, pastikan sudah menginstal board package **ESP8266** (`Tools > Board > Boards Manager` → cari "esp8266").
2. Pilih board: `Tools > Board > Generic ESP8266 Module`.
3. Sesuaikan `Flash Size` dengan modul ESP-01 Anda (umumnya 1MB atau lebih).
4. Sambungkan ESP-01 sesuai skema **"Saat Upload Firmware"** di atas.
5. Buka file `ESP01_Relay_Control.ino`.
6. (Opsional) Ganti nama WiFi dan password pada bagian konfigurasi:
   ```cpp
   const char* ap_ssid     = "ESP01-Relay";
   const char* ap_password = "12345678";
   ```
7. Klik **Upload**.
8. Setelah upload selesai, lepas GPIO0 dari GND, lalu sambungkan ulang ke skema **"Saat Operasi Normal"**.

---

## 🖼️ Menambahkan Logo Sendiri

Logo disimpan sebagai array byte di flash (PROGMEM) dan disajikan lewat endpoint terpisah `/logo.png`. Cara ini dipilih (dibanding base64 langsung di HTML) karena jauh lebih hemat RAM — ESP-01 hanya punya RAM terbatas (~50KB), dan menempel base64 langsung ke halaman HTML sangat mudah membuatnya kehabisan memori sehingga halaman gagal tampil.

1. Siapkan logo berukuran kecil (disarankan ≤150×150 px, format **PNG**, usahakan di bawah 20-30 KB).
2. Simpan script Python berikut sebagai `convert_logo.py`, **satu folder** dengan file logo Anda:
   ```python
   with open("logo.png", "rb") as f:
       data = f.read()
   with open("logo_array.txt", "w") as out:
       out.write("static const uint8_t logo_png[] PROGMEM = {\n")
       for i in range(0, len(data), 12):
           chunk = data[i:i+12]
           out.write("  " + ", ".join(f"0x{b:02X}" for b in chunk) + ",\n")
       out.write("};\n")
       out.write(f"static const unsigned int logo_png_len = {len(data)};\n")
   print("Selesai, lihat logo_array.txt")
   ```
   > ⚠️ Nama file pada baris `open("logo.png", ...)` harus **persis sama** dengan nama file gambar Anda, termasuk ekstensinya. Kalau file Anda `logo.jpg`, ubah baris itu menjadi `open("logo.jpg", "rb")`. Salah ekstensi adalah penyebab paling umum script "tidak terjadi apa-apa".
3. Jalankan script lewat **Command Prompt / Terminal** (jangan diklik dua kali dari File Explorer — kalau ada error, jendelanya akan langsung tertutup sebelum sempat terbaca):
   - Buka folder berisi `convert_logo.py` di File Explorer
   - Klik di address bar, ketik `cmd`, tekan Enter (membuka Command Prompt langsung di folder itu)
   - Ketik `python convert_logo.py`, tekan Enter
   - Jika berhasil, akan muncul tulisan `Selesai, lihat logo_array.txt` dan file baru `logo_array.txt` di folder yang sama
4. Buka `logo_array.txt`, salin seluruh isinya.
5. Di `ESP01_Relay_Control.ino`, cari blok **"DATA LOGO (PROGMEM)"**, lalu ganti seluruh isi array contoh (logo placeholder kotak abu-abu) dengan hasil salinan tadi.
6. Upload ulang firmware.
7. Tes dengan membuka `http://192.168.4.1/logo.png` langsung di browser — kalau gambar tampil di sana, berarti sudah benar.

Jika tidak diganti, akan tampil logo placeholder kotak abu-abu kecil.

**Kalau logo tetap tidak muncul, cek:**
- Apakah `http://192.168.4.1/logo.png` menampilkan gambar saat dibuka langsung? Jika tidak, berarti array/`logo_png_len` salah salin.
- Apakah nama file di script Python sudah cocok dengan nama & ekstensi file logo Anda (`.png` vs `.jpg`)?
- Apakah file gambarnya valid (coba buka dulu di image viewer sebelum dikonversi)?
- Pastikan tidak ada baris yang terpotong saat menyalin isi `logo_array.txt` ke dalam `.ino`.

---

## 📱 Cara Pemakaian

1. Nyalakan ESP-01.
2. Di HP, buka pengaturan WiFi, hubungkan ke jaringan **`ESP01-Relay`** (password default `12345678`).
3. Buka browser, akses alamat:
   ```
   http://192.168.4.1
   ```
4. Halaman kontrol relay akan muncul. Tekan tombol bulat untuk menyalakan/mematikan relay.
5. Pilih skin warna favorit lewat titik-titik warna di bagian bawah kartu.

---

## 🌐 Endpoint API

Firmware ini juga bisa dikontrol langsung lewat HTTP request, misalnya dari aplikasi lain atau automasi:

| Endpoint             | Method | Deskripsi                              | Contoh Response |
|-----------------------|--------|------------------------------------------|------------------|
| `/`                   | GET    | Menampilkan halaman web kontrol          | HTML             |
| `/relay?state=on`     | GET    | Menyalakan relay                         | `ON`             |
| `/relay?state=off`    | GET    | Mematikan relay                          | `OFF`            |
| `/status`             | GET    | Mengambil status relay saat ini          | `ON` / `OFF`     |

Contoh penggunaan dengan `curl`:
```bash
curl "http://192.168.4.1/relay?state=on"
curl "http://192.168.4.1/status"
```

---

## 🎨 Kustomisasi Tema

Empat tema warna sudah tersedia secara default (Light, Dark, Green, Orange). Untuk menambah tema baru, tambahkan blok CSS baru di dalam `<style>` pada `MAIN_page`, contoh:

```css
body.theme-purple{
  --bg:#faf5ff;
  --card:#ffffff;
  --text:#581c87;
  --accent:#9333ea;
  --accent-dark:#7e22ce;
  --off:#c4b5c9;
}
```

Lalu tambahkan tombol titik warna baru di bagian `theme-row` dan fungsi `setTheme('theme-purple')`.

---

## 🛠️ Troubleshooting

| Masalah                                   | Kemungkinan Penyebab & Solusi |
|--------------------------------------------|-------------------------------|
| ESP-01 tidak masuk mode flashing            | Pastikan GPIO0 ke GND saat upload, lalu tekan reset |
| WiFi ESP-01 tidak muncul                    | Cek suplai daya, ESP-01 butuh arus cukup stabil |
| Relay tidak merespons                       | Cek apakah relay memang tipe active LOW, dan GPIO2 tersambung dengan benar |
| Halaman web tidak bisa dibuka               | Pastikan HP benar-benar terhubung ke WiFi `ESP01-Relay`, lalu akses `192.168.4.1` |
| ESP-01 sering restart sendiri               | Biasanya karena kekurangan daya saat relay switching, gunakan sumber 3.3V yang lebih kuat |
| Script `convert_logo.py` "tidak terjadi apa-apa" | Kemungkinan nama/ekstensi file di script tidak cocok dengan file logo Anda (misal script cari `logo.png` tapi file Anda `logo.jpg`), atau dijalankan dengan klik dua kali sehingga pesan error langsung hilang. Jalankan lewat Command Prompt agar error terlihat |

---

## 🌐 Ikuti Kami

**a13d1 Connected**

| Platform | Akun |
|----------|------|
| YouTube | [a13d1 Connected](https://www.youtube.com/@a13d1-connected) |
| Instagram | [a13d1-connected](https://www.instagram.com/a13d1connected?utm_source=qr&stkn=MWlzNTg0dmc1eWJqcg==) |
| TikTok | [a13d1connected](https://www.tiktok.com/@a13d1connected) |

## 📄 Lisensi

Proyek ini bebas digunakan dan dimodifikasi untuk keperluan pribadi maupun edukasi (MIT License).

---

## 🙌 Kontribusi

Pull request dan saran pengembangan sangat diterima, misalnya:
- Dukungan multi-relay
- Timer / penjadwalan otomatis
- Mode STA (terhubung ke WiFi rumah) sebagai alternatif AP
