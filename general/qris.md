# 🔍 Memahami Cara Kerja QRIS di Indonesia & Implementasi QRIS Dinamis untuk GoPay

QRIS (Quick Response Code Indonesian Standard) adalah standar nasional kode QR untuk pembayaran digital di Indonesia. Dikelola oleh **Bank Indonesia (BI)** dan **ASPI**, QRIS bertujuan menyatukan berbagai sistem pembayaran dalam satu QR code yang **interoperable**. QRIS bisa dipakai dengan berbagai dompet digital seperti GoPay, OVO, DANA, ShopeePay, hingga aplikasi mobile banking seperti BCA Mobile.

Artikel ini membahas:
- Cara kerja QRIS secara teknis
- Isi data dalam QRIS
- Kemungkinan memodifikasi QRIS statis
- Implementasi QRIS dinamis via API (misalnya Midtrans, Xendit)
- Integrasi QRIS untuk pembayaran menggunakan GoPay

---

## ⚙️ Bagaimana Cara Kerja QRIS?

1. **Merchant (penjual)** mendaftar ke penyedia QRIS resmi.
2. Merchant mendapat QR code — bisa bersifat **statis** atau **dinamis**:
   - **Statis:** QR tetap, pembeli memasukkan nominal sendiri.
   - **Dinamis:** QR berubah tiap transaksi, berisi nominal dan info tambahan (misal invoice).
3. Pembeli scan QR dengan aplikasi (GoPay, OVO, DANA, dll).
4. Transaksi diproses via sistem switching & acquirer → merchant menerima dana.

---

## 🧾 Struktur Data di Dalam QRIS

QRIS menggunakan format berbasis standar **EMVCo**. Isinya berupa string teks dengan tag dan value berurutan.

Contoh isi QRIS (disederhanakan):

000201
010211
26050010ID.CO.QRIS.WWW
520483
5303360
540610000.00
5802ID
5907WARUNGKU
6007JAKARTA
6304A13B


Penjelasan beberapa tag:

| Tag | Deskripsi                          |
|-----|------------------------------------|
| 00  | Versi format QR                    |
| 01  | Tipe transaksi (11 = statis, 12 = dinamis) |
| 26  | Merchant ID (misal: id.co.qris.www) |
| 52  | Kategori merchant (MCC)            |
| 53  | Mata uang (360 = IDR)              |
| 54  | Nominal pembayaran (jika dinamis)  |
| 58  | Negara                             |
| 59  | Nama merchant                      |
| 60  | Kota merchant                      |
| 63  | CRC (checksum, wajib valid)        |

---

## 🔄 Apakah QRIS Statis Bisa Diubah Menjadi Dinamis?

### 🔧 Secara Teknis: **BISA**

- Kamu bisa memodifikasi QRIS statis:
  1. Tambahkan tag nominal (`54`)
  2. Tambahkan info tambahan (`62`)
  3. Hitung ulang CRC (`63`)
  4. Generate ulang QR code

- Hasilnya akan **terlihat seperti QRIS dinamis**, dan di beberapa aplikasi **bisa terbaca dan digunakan** untuk membayar.

### ⚠️ Namun Secara Regulasi & Praktik: **BERISIKO**

- Uang tetap akan masuk ke **merchant asli** yang terdaftar pada tag `26`.
- Jika kamu bukan pemilik merchant ID tersebut, kamu **tidak akan menerima uangnya**.
- Penggunaan QRIS hasil modifikasi untuk transaksi nyata **melanggar peraturan BI** dan bisa berakibat pemblokiran, pelaporan, hingga sanksi hukum.

### 💡 Untuk Pembelajaran:
Modifikasi QRIS statis hanya aman dilakukan untuk simulasi lokal atau edukasi pribadi, **tidak untuk produksi**.

---

## 🚀 Solusi Legal & Aman: Gunakan API QRIS Dinamis dari PJSP Resmi

Jika kamu butuh QRIS dinamis untuk menerima pembayaran (termasuk GoPay), gunakan layanan dari **Penyedia Jasa Sistem Pembayaran (PJSP)** resmi seperti:

### ✅ Midtrans (anak perusahaan GoTo/Gojek)

Contoh penggunaan:

```http
POST https://api.midtrans.com/v2/charge
Authorization: Basic <base64-server-key>
Content-Type: application/json

{
  "payment_type": "qris",
  "transaction_details": {
    "order_id": "INV-001",
    "gross_amount": 15000
  }
}
```

Respons:
```json
{
  "payment_type": "qris",
  "transaction_id": "...",
  "actions": [
    {
      "name": "generate-qr-code",
      "url": "https://api.qr-code"
    }
  ]
}
```

→ Tampilkan URL QR tersebut sebagai QR code. Bisa dibayar pakai GoPay, OVO, DANA, dll.

✅ Xendit
Contoh curl request:

```bash
curl https://api.xendit.co/qr_codes \
  -u xnd_development_...: \
  -d external_id=qr123 \
  -d type=DYNAMIC \
  -d amount=50000 \
  -d callback_url=https://example.com/qris-callback \
  -d currency=IDR
```

Respons:
```json
{
  "qr_string": "000201010212...6304XXXX",
  "status": "ACTIVE"
}
```

→ Gunakan qr_string untuk generate QR code.

🔒 Mengapa Tidak Ada API Resmi QRIS Langsung dari GoPay?
GoPay adalah issuer, bukan acquirer.

QRIS diterbitkan dan dikelola oleh acquirer atau aggregator resmi seperti Midtrans, Xendit, OY!, dll.

Jadi, GoPay tidak menyediakan API QRIS langsung.

Namun QRIS yang dihasilkan lewat acquirer bisa dibayar pakai GoPay karena sistemnya terhubung lewat switching resmi (Artajasa, Rintis, dll).

📌 **Kesimpulan**

| Topik                                      | Jawaban Singkat                                                      |
|--------------------------------------------|----------------------------------------------------------------------|
| Bisa modifikasi QRIS statis jadi dinamis?  | Secara teknis: ya. Secara legal: tidak boleh                         |
| QR hasil modifikasi bisa dibayar?          | Tergantung aplikasi, tapi uang masuk ke merchant asli                |
| Cara resmi buat QRIS dinamis GoPay?        | Gunakan Midtrans, Xendit, atau PJSP resmi lainnya                    |
| Apakah GoPay punya API QRIS langsung?      | Tidak. Gunakan aggregator                                            |
| Aman & legal untuk produksi?               | Hanya lewat jalur resmi PJSP yang terdaftar di BI                    |

---

Jika kamu ingin:
- Belajar struktur QRIS
- Membuat generator QRIS dinamis lokal
- Memahami CRC (checksum) EMVCo

Saya sarankan:
- Gunakan bahasa **Python** / **Node.js**
- Pelajari **CRC-16/CCITT-FALSE** untuk menghitung tag `63`

**Referensi:**
- [Spesifikasi QRIS – ASPI](https://www.aspi-indonesia.or.id)
- [Dokumentasi Midtrans QRIS](https://docs.midtrans.com)
- [Dokumentasi Xendit QRIS](https://developers.xendit.co)

---
