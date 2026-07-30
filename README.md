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
| 5. Outlier Handling | Deteksi outlier adaptif (IQR untuk data skewed, persentil ke-96 khusus `review_count` & `stock_quantity`, business rule `[0, 5]` untuk `rating_avg`), capping |
| 6. Scaling | `StandardScaler` |
| 7. Clustering | K-Means dengan pemilihan k dievaluasi lewat Elbow Method (Inertia), Silhouette Score, dan Silhouette Diagram per-k |
| 8. Profiling Cluster | Interpretasi tiap cluster memakai fitur inti + fitur pendukung (`price`, `description_length`, `product_age_days`) |

## Temuan Utama

**Missing value pada `rating_avg` (54,8%)**
Diverifikasi secara statistik (korelasi Pearson r=1.0, p=0.0 terhadap `review_count == 0`) —
produk yang belum pernah direview memang tidak mungkin memiliki rating. Ditangani dengan
imputasi 0 + flag kolom baru `is_review`.

**Fitur yang dipilih untuk clustering**: `rating_avg`, `review_count`, `stock_quantity`
(dipilih berdasarkan korelasi terhadap pola pembelian; `price` dikeluarkan karena korelasinya lemah)

**Jumlah cluster optimal**: k=4, dipilih berdasarkan kombinasi tiga evaluasi — Elbow Method
(penurunan inertia mulai melandai signifikan setelah k=4), Silhouette Score (k=4 masih kompetitif
di 0.540, selisih tipis dari k=3 di 0.573), dan validasi profil bisnis (setiap cluster di k=4
menunjukkan karakteristik yang berbeda dan actionable, termasuk munculnya segmen "produk
bermasalah kualitasnya" yang tidak terlihat di k=3)

### Hasil Segmentasi

| Cluster | Nama | Ukuran | Karakteristik |
|---|---|---|---|
| 0 | **Premium & High Rating** | 273 item | Harga termahal, rating tertinggi (4,58), stok sangat menipis |
| 1 | **Low Rating / Underperforming** | 104 item | Rating terendah (2,57), review paling minim — kandidat evaluasi kualitas |
| 2 | **High Demand / Most Popular** | 46 item | Review terbanyak (12,98), rating baik, stok sangat kritis |
| 3 | **Bulk Stock / Mass Storage** | 29 item | Stok melimpah (309 unit), harga paling terjangkau, produk paling senior |

## Rekomendasi Bisnis

- **Cluster 0 & Cluster 2**: prioritas restock — keduanya produk favorit pasar (margin tinggi / demand tinggi) dengan stok kritis (<5 unit)
- **Cluster 1**: audit kualitas produk, deskripsi, atau vendor — rating rendah berisiko merusak citra toko
- **Cluster 3**: kandidat kampanye diskon/bundling untuk mempercepat perputaran stok yang menumpuk

## Requirements

```
pandas
numpy
scipy
matplotlib
seaborn
scikit-learn
```

## Cara Menjalankan

```bash
pip install pandas numpy scipy matplotlib seaborn scikit-learn
jupyter notebook Notebook.ipynb
```

Pastikan `products.csv` berada di direktori yang sama dengan notebook.

## Catatan & Rencana Pengembangan

- Analisis saat ini hanya mencakup produk dengan `is_review == 1` (pernah direview). Segmentasi
  untuk produk `is_review == 0` (belum pernah direview) direncanakan sebagai pengembangan lanjutan,
  kemungkinan menggunakan pendekatan rule-based (bukan clustering) karena minimnya sinyal interaksi
  pada kelompok ini
- Kategori (`category`, `subcategory`, `brand`) belum dilibatkan sebagai fitur clustering, masih
  berpotensi digunakan sebagai lapisan analisis/pelaporan tambahan di iterasi berikutnya