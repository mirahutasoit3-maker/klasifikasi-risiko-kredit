# Klasifikasi Risiko Gagal Bayar Nasabah Kartu Kredit

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mirahutasoit3-maker/klasifikasi-risiko-kredit/blob/main/Milestone1_UTS_Klasifikasi_RisikoKreditI.ipynb)

Milestone 1 UTS Machine Learning, **Kelompok A07**.

Anggota:
- Mirawati Hutasoit (231110639)
- Saskia Anggi Pratiwi (231110909)
- Ade Kurnia Setialu (231110333)

## Deskripsi
Proyek ini membangun model klasifikasi untuk memprediksi nasabah kartu kredit yang berisiko **gagal bayar** pada bulan berikutnya. Notebook mencakup pemahaman data, EDA, preprocessing, feature engineering, perbandingan baseline dengan beberapa model kandidat, dan analisis hasil awal.

## Dataset
- Sumber: [Default of Credit Card Clients, UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/350/default+of+credit+card+clients) (Yeh & Lien, 2009)
- Isi: 30.000 nasabah, 23 fitur, dan 1 target
- Target: `default` (1 = gagal bayar / risiko tinggi, 0 = risiko rendah)
- Setelah 35 baris duplikat dihapus: 29.965 baris. Proporsi gagal bayar 22,13% (rasio kelas 3,52 : 1).
- Dataset **diunduh otomatis** oleh notebook dari UCI, jadi tidak disertakan di repository ini.

## Metodologi
1. Data understanding: tipe data, missing value, duplikat, outlier, kode kategori yang tidak terdokumentasi
2. Pembersihan: kode `EDUCATION` 0/5/6 digabung ke 4, kode `MARRIAGE` 0 digabung ke 3, duplikat dihapus
3. EDA: distribusi target, tingkat gagal bayar per kategori, status pembayaran, korelasi
4. Split data 80:20 stratified (23.972 latih, 5.993 uji), `random_state=42`
5. Feature engineering: `n_late_months`, `max_delay`, `utilization`, `pay_ratio`, `avg_bill`, `avg_pay_amt`
6. Preprocessing (pipeline, hanya di-fit pada data latih): transformasi log bertanda dan scaling untuk fitur uang, scaling untuk fitur lain, one-hot encoding untuk kategori (32 fitur akhir)
7. Model: Dummy (mayoritas), **Logistic Regression (baseline)**, Logistic Regression + `class_weight`, Decision Tree, KNN (k=25), Random Forest, HistGradientBoosting
8. Evaluasi: 5-fold stratified CV pada data latih, lalu evaluasi satu kali pada data uji (ambang 0,5). Metrik: Accuracy, Precision, Recall, F1, ROC-AUC, PR-AUC.
9. Ablation: model dengan dan tanpa fitur turunan

## Hasil
Kolom CV berisi rata-rata 5-fold, kolom Test berisi hasil hold-out 20%.

| Model | Peran | CV Recall | CV F1 | CV PR-AUC | Test Recall | Test F1 | Test PR-AUC |
|---|---|---|---|---|---|---|---|
| Dummy (mayoritas) | pembanding trivial | 0,000 | 0,000 | 0,221 | 0,0000 | 0,0000 | 0,2213 |
| Logistic Regression | **baseline** | 0,319 | 0,419 | 0,510 | 0,3160 | 0,4198 | 0,5178 |
| LogReg + class_weight | kandidat | 0,625 | 0,530 | 0,505 | 0,6048 | 0,5166 | 0,5124 |
| Decision Tree | kandidat | 0,645 | 0,524 | 0,524 | 0,6146 | 0,5176 | 0,5150 |
| KNN (k=25) | kandidat | 0,354 | 0,451 | 0,505 | 0,3522 | 0,4475 | 0,5012 |
| Random Forest | kandidat | 0,612 | 0,544 | 0,560 | 0,5980 | 0,5364 | 0,5551 |
| HistGradientBoosting | kandidat | 0,628 | 0,537 | 0,564 | 0,6222 | 0,5265 | 0,5559 |

## Temuan Utama
- Model yang selalu menebak "tidak gagal bayar" sudah mencapai akurasi 0,779 tanpa mendeteksi satu pun nasabah berisiko, sehingga Accuracy tidak cukup sebagai metrik utama.
- Status pembayaran terakhir (`PAY_1`) adalah sinyal terkuat: gagal bayar 50,4% pada nasabah yang terlambat dibanding 13,8% pada yang tidak terlambat.
- Baseline Logistic Regression berakurasi 0,807 pada data uji, tetapi Recall hanya 0,316.
- `class_weight="balanced"` menaikkan Recall (0,319 menjadi 0,625 pada CV) terutama dengan menggeser titik operasi. PR-AUC hampir tidak berubah.
- Random Forest dan HistGradientBoosting memberi PR-AUC tertinggi (0,560 dan 0,564 pada CV) dan belum dapat dibedakan satu sama lain.
- Hasil CV dan hold-out sejalan. Pemilihan model final ditunda ke tahap lanjutan.

## Cara Menjalankan
**Google Colab (disarankan):** klik badge "Open In Colab" di atas, lalu pilih **Runtime > Run all**.

**Lokal:**
1. Clone repository:
   ```
   git clone https://github.com/mirahutasoit3-maker/klasifikasi-risiko-kredit.git
   ```
2. Pasang library:
   ```
   pip install pandas numpy matplotlib seaborn scikit-learn openpyxl xlrd jupyter
   ```
   `openpyxl` dan `xlrd` dipakai untuk membaca file Excel dari UCI. Paket `ucimlrepo` bersifat opsional, karena notebook otomatis memakai unduhan `.zip` UCI jika paket itu tidak ada.
3. Buka `Milestone1_UTS_Klasifikasi_RisikoKreditI.ipynb` dan jalankan semua sel.

Butuh koneksi internet untuk mengunduh dataset. Tidak perlu mengunggah file apa pun. Seluruh hasil memakai `random_state=42` sehingga dapat direproduksi. Diuji dengan scikit-learn 1.6.1 dan pandas 2.2.3.

Saat dijalankan, notebook membuat folder `results/` (tabel CSV dan `summary.json`), folder `figures/` (grafik), dan arsip `keluaran_milestone1.zip`. Semuanya dihasilkan ulang dan tidak disertakan di repository.

## Keterbatasan
- Makna nilai `PAY_*` = -2 dan 0 tidak dijelaskan dokumentasi dataset, sehingga dipertahankan sebagai skala ordinal.
- Korelasi Pearson hanya mengukur hubungan linear dan tidak membuktikan sebab-akibat.
- Semua evaluasi memakai ambang keputusan 0,5.

## Sumber
Yeh, I. C., & Lien, C. H. (2009). The comparisons of data mining techniques for the predictive accuracy of probability of default of credit card clients. *Expert Systems with Applications*.
