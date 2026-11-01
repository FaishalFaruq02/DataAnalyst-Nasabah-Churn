# Bank Customer Churn Analysis Dashboard

Analisis churn nasabah bank dari data mentah sampai dashboard: pembersihan dan transformasi data dengan **Python (pandas)** dan **Excel**, lalu visualisasi dengan **Power BI**.

**Hasil singkat:** dari 10.000 nasabah, **20,4%** berhenti menjadi nasabah (churn). Risikonya terkonsentrasi pada nasabah di **Jerman**, usia **50–59 tahun**, dan yang **tidak aktif**.

**Dataset:** `bank_churn_dirty.csv`, 10.150 baris dan 18 kolom (negara, usia, credit score, saldo, jumlah produk, status member, dan `churn`).
[Lihat file mentah](https://github.com/USERNAME/bank-customer-churn-analysis/blob/main/data/raw/bank_churn_dirty.csv) · Sumber dan lisensi: _[isi sumber dataset di sini]_

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

**Python** ([`src/bank_churn_pipeline.py`](src/bank_churn_pipeline.py))

```python
# Kunci duplikat tidak memakai customer_id (sebagian salinan kosong ID-nya)
df = (df.assign(_id_kosong=df["customer_id"].isna())
        .sort_values("_id_kosong", kind="stable")
        .drop_duplicates(subset=kunci, keep="first"))

# Dua format tanggal digabung
iso = pd.to_datetime(df["date_of_birth"], format="%Y-%m-%d", errors="coerce")
dmy = pd.to_datetime(df["date_of_birth"], format="%d/%m/%Y", errors="coerce")
df["dob"] = iso.fillna(dmy)

# Usia dihitung ulang dari tanggal lahir (tanggal acuan tetap: 2026-10-04)
```

**Excel**

| Kebutuhan | Cara |
|---|---|
| Hapus duplikat | `Data` → `Remove Duplicates` |
| Rapikan teks kota | `=PROPER(TRIM(B2))` |
| Tandai nilai tidak valid | Rumus di bawah |
| Warnai baris bermasalah | `Conditional Formatting` dengan rumus `=$S2<>"OK"` |

```excel
=IF(AND(J2>=0,J2<>99999999,G2<>999,P2<>0),"OK",
 TEXTJOIN("; ",TRUE,IF(J2<0,"saldo negatif",""),IF(J2=99999999,"saldo placeholder 99999999",""),
 IF(G2=999,"credit_score 999",""),IF(P2=0,"gaji 0","")))
```

_Huruf kolom pada rumus hanyalah contoh; sesuaikan dengan sheet Anda._

### 2. Transform

Kolom turunan yang ditambahkan agar data siap dianalisis:

| Kolom | Isi |
|---|---|
| `age_group` | `<30`, `30-39`, `40-49`, `50-59`, `60+` |
| `score_band` | Poor, Fair, Good, Very Good, Excellent, Tidak valid |
| `balance_segment` | Nol, 0-8 jt, 8-12 jt, 12-16 jt, >16 jt, Tidak valid |
| `churn_label` | `Churn` atau `Bertahan` |
| `data_quality_flag` | `OK` atau jenis masalah pada baris itu |

Kolom `name`, `email`, `count_number`, dan `date_of_birth` dibuang karena berisi data pribadi dan tidak dibutuhkan untuk analisis churn.

**Python**

```python
out["age_group"] = pd.cut(out["age"], [0, 30, 40, 50, 60, 200], right=False,
                          labels=["<30", "30-39", "40-49", "50-59", "60+"])
out["churn_label"] = np.where(df["churn"] == 1, "Churn", "Bertahan")
```

**Excel**

```excel
age_group:       =IFS(E2<30,"<30",E2<40,"30-39",E2<50,"40-49",E2<60,"50-59",TRUE,"60+")
score_band:      =IF(G2="","Tidak valid",IFS(G2<580,"Poor (<580)",G2<670,"Fair (580-669)",G2<740,"Good (670-739)",G2<800,"Very Good (740-799)",TRUE,"Excellent (800+)"))
balance_segment: =IF(J2="","Tidak valid",IFS(J2=0,"Nol",J2<=8000000,"0-8 jt",J2<=12000000,"8-12 jt",J2<=16000000,"12-16 jt",TRUE,">16 jt"))
churn_label:     =IF(Q2=1,"Churn","Bertahan")
```

Ringkasan churn per segmen dibuat dengan `COUNTIFS`:

```excel
=COUNTIFS(Data_Bersih!$C$2:$C$10001,"="&A8,Data_Bersih!$Q$2:$Q$10001,1)
```

Kriteria memakai `"="&A8` supaya label seperti `<30` dan `>16 jt` dibaca sebagai teks, bukan operator pembanding.

**Hasil akhir:** [`data/clean/Bank_Churn_Clean.xlsx`](data/clean/Bank_Churn_Clean.xlsx)

| Sheet | Isi |
|---|---|
| `Data_Bersih` | 10.000 baris dan 19 kolom; baris dengan flag selain `OK` berwarna merah |
| `Log_Kualitas_Data` | Jumlah dan penanganan setiap masalah data |
| `Ringkasan_Churn` | Churn rate per segmen dengan rumus |

| | Sebelum | Sesudah |
|---|---|---|
| Baris | 10.150 | 10.000 |
| Kolom | 18 | 19 |
| Baris bernilai tidak valid | tidak ditandai | 1.350 (13,5%), ditandai |

**Cara menjalankan**

```bash
pip install pandas numpy openpyxl
python src/bank_churn_pipeline.py data/raw/bank_churn_dirty.csv data/clean/Bank_Churn_Clean.xlsx
```

Nilai uang di CSV berbentuk desimal (`128449.33`) dan dikalikan 100 agar satuannya sama dengan label segmen saldo ("jt"). Satuan mata uang tidak disebutkan di data sumber.

### 3. Dashboard Power BI

Sumber data: sheet `Data_Bersih` dari `Bank_Churn_Clean.xlsx`.

```DAX
Churn Rate = DIVIDE(SUM(Data_Bersih[churn]), COUNTROWS(Data_Bersih))

Total Nasabah = COUNTROWS(Data_Bersih)

Nasabah Churn = SUM(Data_Bersih[churn])

Member Aktif Churn =
CALCULATE(COUNTROWS(Data_Bersih), Data_Bersih[active_member] = 1, Data_Bersih[churn] = 1)

Total Tidak Valid =
CALCULATE(COUNTROWS(Data_Bersih), Data_Bersih[data_quality_flag] <> "OK")
```

- Urutan kategori (`age_group`, `score_band`, `balance_segment`) diatur dengan kolom urutan dan fitur *Sort by column*, agar tidak terurut abjad.
- Slicer: `country`, `gender`, `active_member`.
- Tema **Churn Watch**: biru tua `#14213D` untuk teks, biru kehijauan `#3B7F95` untuk total nasabah, dan merah `#C2334D` khusus untuk churn.

---

## Dashboard

![Dashboard Churn Watch](dashboard/screenshot.png)

_Dashboard Power BI: KPI churn rate, churn per negara, kelompok usia, jumlah produk, status member, band credit score, dan segmen saldo, dengan slicer negara, gender, dan status member._

- **File Power BI:** [`dashboard/churn-watch.pbix`](dashboard/churn-watch.pbix)
- **Versi online (opsional):** _[tempel link Power BI Service di sini bila sudah dipublikasikan]_

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

### Keterbatasan
- Semua temuan adalah **korelasi**, bukan sebab-akibat; data tidak menjelaskan alasan nasabah pergi.
- Kelompok produk 3–4 kecil (326 orang), jadi angkanya ekstrem dan perlu dibaca hati-hati.
- Periode churn dan satuan mata uang tidak tercantum di data sumber.

---

## Struktur repo

```
bank-customer-churn-analysis/
├── data/
│   ├── raw/          bank_churn_dirty.csv
│   └── clean/        Bank_Churn_Clean.xlsx
├── src/
│   └── bank_churn_pipeline.py
├── dashboard/
│   ├── churn-watch.pbix
│   └── screenshot.png
└── README.md
```

**Tools:** Python (pandas, numpy, openpyxl) · Microsoft Excel · Power BI (DAX)
