# Analisis & Segmentasi Kesehatan Stok Produk E-Commerce

Notebook ini menganalisis dataset `products.csv` (1.000 produk, 11 kolom) untuk mengidentifikasi
pola perilaku konsumen dan mengelompokkan produk berdasarkan tingkat kesehatan stok
menggunakan **K-Means Clustering**.

## Latar Belakang

Gudang mengalami ketidakseimbangan inventaris yang ekstrem:
- **~70% dari produk terpopuler** (berdasarkan jumlah review) mengalami krisis stok (`stock_quantity < 10`)
- **>50% produk** yang sepi peminat justru menumpuk *deadstock* hingga 5.000 unit

Masalah ini terjadi karena belum ada pemetaan berbasis data yang menghubungkan preferensi
pembeli dengan status perputaran stok. Notebook ini membangun pemodelan clustering untuk
menjawab kebutuhan tersebut.

## Struktur Analisis

| Tahap | Deskripsi |
|---|---|
| 1. Data Loading | Load `products.csv`, cek tipe data, duplikat |
| 2. Data Cleaning | Perbaikan tipe data `date_added`, penanganan missing value pada `rating_avg`, pembuatan flag `is_review` |
| 3. Exploratory Data Analysis | Distribusi status review, analisis top-10 & top-1 produk berdasarkan `review_count`, root-cause analysis |
| 4. Feature Engineering | Heatmap korelasi untuk seleksi fitur, pembuatan fitur turunan (`description_length`, `product_age_days`) |
| 5. Outlier Handling | Deteksi outlier adaptif (IQR untuk data skewed, mean±3σ untuk data mendekati normal), capping |
| 6. Scaling | `RobustScaler` (dipilih karena data mengandung outlier) |
| 7. Clustering | K-Means dengan pemilihan k berbasis Silhouette Score |
| 8. Profiling Cluster | Interpretasi tiap cluster memakai fitur inti + fitur pendukung (`price`, `description_length`, `product_age_days`) |

## Temuan Utama

**Missing value pada `rating_avg` (54,8%)**
Diverifikasi secara statistik (korelasi Pearson r=1.0, p=0.0 terhadap `review_count == 0`) —
produk yang belum pernah direview memang tidak mungkin memiliki rating. Ditangani dengan
imputasi 0 + flag kolom baru `is_review`.

**Fitur yang dipilih untuk clustering**: `rating_avg`, `review_count`, `stock_quantity`
(dipilih berdasarkan korelasi terhadap pola pembelian; `price` dikeluarkan karena korelasinya lemah)

**Jumlah cluster optimal**: k=3, dipilih dari Silhouette Score tertinggi (0.567) di antara k=2–6

### Hasil Segmentasi

| Cluster | Nama | Ukuran | Karakteristik |
|---|---|---|---|
| 0 | **High Engagement / Popular Products** | 79 item | Harga premium, stok terbatas, review & rating tinggi |
| 1 | **High Stock / Bulk Inventory** | 76 item | Harga terjangkau, stok melimpah, review masih rendah-sedang |
| 2 | **Standard / Low Review Products** | 297 item | Mayoritas katalog, stok terbatas, review paling rendah |

## Rekomendasi Bisnis

- **Cluster 0**: prioritaskan restock darurat / cari supplier alternatif untuk mencegah stockout
- **Cluster 1**: dorong penjualan lewat bundling atau promosi untuk mempercepat perputaran stok
- **Cluster 2**: pantau lebih lanjut — kelompok terbesar dengan interaksi paling minim, berpotensi jadi target strategi peningkatan visibilitas produk

## Requirements

```
pandas
numpy
scipy
matplotlib
seaborn
scikit-learn
feature-engine
```

## Cara Menjalankan

```bash
pip install pandas numpy scipy matplotlib seaborn scikit-learn feature-engine
jupyter notebook Notebook__5_.ipynb
```

Pastikan `products.csv` berada di direktori yang sama dengan notebook.

## Catatan & Batasan

- Clustering saat ini hanya menggunakan 3 fitur inti; fitur seperti `price`, `description_length`,
  dan `product_age_days` baru dipakai pada tahap profiling pasca-clustering, belum sebagai bagian
  dari model itu sendiri
- Belum ada langkah ekspor hasil akhir (dataframe berlabel cluster) ke file terpisah