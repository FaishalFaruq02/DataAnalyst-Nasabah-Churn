# Bank Customer Churn Analysis Dashboard

Analisis churn nasabah bank end to end dari data mentah sampai dashboard

**Dataset:** `bank_churn_dirty.csv`
[Lihat file mentah](https://github.com/USERNAME/bank-customer-churn-analysis/blob/main/data/raw/bank_churn_dirty.csv) ·

**Isi:** [Pertanyaan](#pertanyaan) · [Proses](#proses) · [Dashboard](#dashboard) · [Project Insight](#project-insight)

---

## Pertanyaan

1. Berapa **churn rate** keseluruhan?
2. **Negara** mana yang churn-nya paling tinggi?
3. **Kelompok usia** mana yang paling berisiko?
4. Apakah **jumlah produk** yang dipakai nasabah memengaruhi churn?
5. Apakah **member aktif** berbeda dari yang tidak aktif?
6. Apakah ada perbedaan churn berdasarkan **gender**?
7. Apakah **saldo, credit score, lama menjadi nasabah, dan kartu kredit** berpengaruh?
8. **Segmen mana** yang paling layak diprioritaskan untuk program retensi?

> **Definisi churn:** kolom `churn` bernilai 1 untuk nasabah yang sudah berhenti menjadi nasabah bank, dan 0 untuk yang bertahan. Data sumber tidak mencantumkan periode waktunya.

---

## Proses

```
Data mentah (CSV)
   │
   ├─ 1. Cleaning   ── Python & Excel
   ├─ 2. Transform  ── Python & Excel
   └─ 3. Dashboard  ── Power BI
```

---

### 1. Cleaning

Masalah yang ditemukan pada data mentah dan cara penanganannya:

| Masalah | Jumlah | Penanganan |
|---|---|---|
| Baris duplikat | 150 | Dihapus |
| `customer_id` kosong | 202 | Diberi ID pengganti yang unik |
| Usia tidak wajar (di bawah 18 tahun, termasuk -1) | 300 | Dihitung ulang dari tanggal lahir |
| Tanggal lahir dengan dua format (`1982-10-04` dan `04/10/1982`) | 6.096 dan 4.054 | Diseragamkan |
| Nama kota huruf kecil (`nantes`, `paris`, ...) | ±800 | Diubah ke format judul (`Nantes`) |
| Saldo negatif | 308 | Dikosongkan dan ditandai |
| Saldo placeholder `999999.99` | 200 | Dikosongkan dan ditandai |
| `credit_score` = 999 (di luar skala 300–850) | 300 | Dikosongkan dan ditandai |
| `estimated_salary` kosong | 600 | Ditandai |

Baris bernilai tidak valid **tidak dihapus**. Nilainya dikosongkan dan barisnya ditandai di kolom `data_quality_flag`, supaya data churn pada baris itu tetap bisa dipakai.

---

## Dashboard
<img width="1212" height="691" alt="Screenshot" src="https://github.com/user-attachments/assets/3cadeee7-64ac-4717-931d-95fe2687017a" />

---

## Project Insight

Churn rate keseluruhan: **20,4%** (2.037 dari 10.000 nasabah).

1. **Jerman adalah pusat masalah.** Churn 32,4% dibanding 16,2% (Prancis) dan 16,7% (Spanyol). Jerman hanya 25% nasabah tetapi 40% dari seluruh churn. Keenam kotanya berada di kisaran 29–36%, jadi penyebabnya kemungkinan di level negara.
2. **Usia 50–59 tahun paling rawan.** Churn 55,7%, sedangkan di bawah 40 tahun di bawah 10%. Usia 45–64 tahun menyumbang sekitar 54% dari seluruh churn.
3. **Jumlah produk berpola tidak linear.** Dua produk paling loyal (7,6%), satu produk 27,7%, dan tiga produk ke atas 85,9% (326 nasabah).
4. **Nasabah tidak aktif lebih mudah pergi.** Churn 26,9% vs 14,3%. Pada usia 50–59 yang tidak aktif churn mencapai 74%, dan usia 60+ yang tidak aktif 87%.
5. **Perempuan churn lebih tinggi:** 25,1% vs 16,5% pada laki-laki.
6. **Saldo bukan perisai.** Nasabah bersaldo nol churn 13,8%, sedangkan yang bersaldo positif sekitar 22–24%.
7. **Faktor yang hampir tidak berpengaruh:** credit score (19–22%), lama menjadi nasabah (17–23%), dan kepemilikan kartu kredit (20,8% vs 20,2%).
8. **Segmen prioritas:** nasabah Jerman berusia 40+ (1.310 orang) churn 47,8%.

### Rekomendasi
- Arahkan program retensi ke segmen berisiko (Jerman, usia 40+, tidak aktif), bukan ke semua nasabah.
- Jadikan status tidak aktif sebagai sinyal peringatan dini.
- Selidiki paket produk 3–4, misalnya apakah ada masalah pada paketnya atau nasabah dijual terlalu banyak produk.
- Cari tahu apa yang berbeda di pasar Jerman (kompetitor, layanan, biaya).
