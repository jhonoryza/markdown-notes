# SQLITE

Kapan Pilih SQLite vs Client/Server?

| Kondisi                                     | Pilihan       |
| ------------------------------------------- | ------------- |
| Akses data lewat jaringan                   | Client/Server |
| Banyak penulis bersamaan                    | Client/Server |
| Data sangat besar                           | Client/Server |
| Aplikasi lokal, penulis sedikit, data kecil | SQLite        |

SQLite = simpel dan cukup untuk banyak kasus lokal. Client/Server = lebih cocok
untuk kasus kompleks dan skala besar.

---

## Kapan SQLite Kurang Cocok?

| Situasi                                  | Kenapa Kurang Cocok                                     | Solusi Lebih Baik |
| ---------------------------------------- | ------------------------------------------------------- | ----------------- |
| Banyak klien akses database via jaringan | File locking bisa gagal → risiko korupsi data           | PostgreSQL, MySQL |
| Website sibuk atau banyak penulisan      | Butuh skala dan performa tinggi                         | PostgreSQL, MySQL |
| Data sangat besar (>1 TB atau >1 file)   | SQLite simpan semua dalam 1 file → bisa mentok batas OS | PostgreSQL, MySQL |
| Banyak penulis bersamaan                 | SQLite hanya izinkan 1 penulis di satu waktu            | PostgreSQL, MySQL |

---

## Kapan SQLite Cocok?

| Kebutuhan / Situasi                     | Cocok? | Alasan                                              |
| --------------------------------------- | ------ | --------------------------------------------------- |
| Perangkat kecil / IoT                   | ✅     | Ringan, offline, tanpa admin                        |
| Aplikasi desktop pakai file penyimpanan | ✅     | Mudah diakses, hemat tempat, auto update            |
| Website kecil-menengah                  | ✅     | Cukup cepat dan andal untuk traffic sedang          |
| Analisis data lokal                     | ✅     | Mudah digunakan, hasil bisa dibagikan dengan 1 file |
| Cache lokal untuk server                | ✅     | Percepat akses, tetap jalan saat offline            |
| Format kirim data (seperti ZIP)         | ✅     | Format stabil, query langsung, cross-platform       |
| Testing/demo aplikasi                   | ✅     | Gampang setup, tak butuh server                     |
| File sementara / internal               | ✅     | Query fleksibel, efisien                            |
