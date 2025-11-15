# Analisis Harga Rumah Bekasi 

**Ringkasan singkat**

Kamu ingin membeli rumah di Bekasi untuk tempat tinggal. Untuk membantu pengambilan keputusan, proyek ini mengumpulkan data listing rumah dari **Rumah123** (lokasi: Bekasi), membersihkan data, melakukan analisis deskriptif, menghitung *confidence interval* 95% untuk "harga wajar", menguji hipotesis apakah rumah dengan **Carport** lebih mahal dibandingkan tanpa Carport, dan menguji korelasi apakah **luas tanah** atau **luas bangunan** lebih kuat berhubungan dengan harga.

> File hasil scraping disimpan sebagai `rumah_bekasi.csv` (.csv)

---

## Struktur README
1. Problem Statement
2. Objective
3. Data
4. Metode / Alur Pengerjaan
5. Environment 
6. Langkah Teknis (kode contoh)
7. Analisis & Statistik
8. Hasil yang Diharapkan 

---

## 1. Problem Statement
Kamu ingin membeli rumah di Bekasi. Tersedia banyak pilihan, sehingga diperlukan analisis data listing Rumah123 (lokasi Bekasi) untuk: menemukan harga wajar, menguji apakah fitur seperti *Carport* memengaruhi harga, dan menentukan apakah membeli rumah berdasarkan *luas tanah* atau *luas bangunan* lebih menguntungkan.

---

## 2. Objective
- Mengumpulkan listing rumah Bekasi dari Rumah123, dan menyimpan data (`rumah_bekasi.csv`)
- Membersihkan data (`rumah_bekasi.csv`)
- Menyajikan statistik deskriptif (mean, median, std, quantiles) untuk harga dan variabel penting.
- Menghitung **95% confidence interval** untuk harga wajar .
- Uji hipotesis: apakah rumah dengan *Carport* memiliki harga yang berbeda (lebih tinggi) dibanding yang tanpa *Carport*.
- Menguji korelasi antara harga vs luas tanah dan harga vs luas bangunan; beri rekomendasi beli berdasarkan hasil korelasi.

---

## 3. Data
**Sumber:** listing Rumah123 (filter lokasi: Bekasi)

**Kolom yang direkomendasikan untuk diambil / yang diharapkan di `rumah_bekasi.csv`:**
- `harga` (harga listing dalam IDR)
- `lokasi` (lokasi listing)
- `lt` (luas tanah dalam m²)
- `lb` (luas bangunan dalam m²)
- `kamar` (jumlah kamar)
- `kamar_mandi` (jumlah kamar mandi)
- `carport` (jumlah carport)

> Catatan: Nama kolom bisa berbeda sesuai hasil scraping — pastikan mengubah `column names` saat preprocessing.

---

## 4. Metode / Alur Pengerjaan
1. **Scraping**: Scrape listing Rumah123 untuk lokasi Bekasi. Simpan menjadi `rumah_bekasi.csv`.
2. **Load data**: Baca csv ke pandas DataFrame.
3. **Data processing**:
   - Tangani missing values .
   - Hapus duplikat .
   - Normalisasi value unik .
4. **Analisis deskriptif**
5. **Confidence interval 95%**
6. **Uji hipotesis (Carport)**
7. **Korelasi**
8. **Kesimpulan & Rekomendasi**.

---

## 5. Environment 
Contoh paket Python:
- **BeutifulSoup**
- **Selenium**
- **Pandas**
- **Numpy**
- **Scipy**

---

## 6. Langkah Teknis — Contoh Kode (ringkas)
> Contoh ini ditulis sebagai *guide*; sesuaikan selector scraping dengan struktur HTML Rumah123.

### a) Scraping (contoh sederhana menggunakan requests + BeautifulSoup)
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

# buat instance webdriver
driver = webdriver.Chrome()

for i in range(1, 11):
    url = f'https://www.rumah123.com/jual/bekasi/rumah/?page={i}'
    driver.get(url)

    # Scroll untuk load 
    for _ in range(24):

        # Scroll awah
        driver.execute_script("window.scrollBy(0, 1000)")
        time.sleep(1)

    # inisialisasi html
    html = driver.page_source

    # inisialisasi BeutifulSoup
    soup = BeautifulSoup(html, "html.parser")

pd.DataFrame(results).to_csv('rumah_bekasi.csv', index=False)
```
---

### b) Load & Cleaning (pandas)
```python
# Load Data
df = pd.read_csv('rumah_bekasi.csv')

# Handling Duplicated
df = df.drop_duplicates()
df = df.reset_index()
df.duplicated().sum()

# Cleaning harga
df['harga'] = df['harga'].str.split( )
df['jumlah'] = df['harga'].str[2]
df['harga'] = df['harga'].str[1].str.replace(',','.').astype(float)
df['jumlah'] = df['jumlah'].str.replace('Miliar','1_000_000_000').str.replace('Juta','1_000_000').astype(int)
df['harga'] = df['jumlah']*df['harga'].astype('int64')
```

---

## 7. Analisis & Statistik — Contoh Kode

### Statistik deskriptif
```python
df.describe()
```

### Confidence Interval 95% untuk harga
```python
std = df['harga'].std()
N = len(df)
low, up = stats.norm.interval(0.95,loc=df['harga'].mean(),scale=std/np.sqrt(N))
low = round(low)
up = round(up)
print('Lower Limit:',low)
print('Upper Limit:',up)
pd.DataFrame({
    'Minimal': [low],
    'Maximal':[up]
})
```

### Uji Hipotesis: Carport vs No Carport
**Hipotesis:** H0: harga ada carport = harga tidak carport; H1: harga ada carport > harga tidak carport


```python
wajar = df[df['harga'].between(low,up)]
tanpa_carport = wajar[wajar['carport']==0]
Ada_carport = wajar[wajar['carport']>0]
t_stat, p_val = stats.ttest_ind(tanpa_carport['harga'],Ada_carport['harga'])
print('T-Statistic:',t_stat)
print('P-value:',p_val) 
pd.DataFrame({
    'T-Statistic':[t_stat],
    'P-value':[p_val]
}).round(3)
```

### Korelasi: Harga vs Luas Tanah & Harga vs Luas Bangunan
```python
corr_r_lt, pval_p_lt = stats.pearsonr(wajar['lt'], wajar['harga'])
corr_r_lb, pval_p_lb = stats.pearsonr(wajar['lb'], wajar['harga'])

print(f"r-correlation: {corr_r_lt:.3f}, p-value: {pval_p_lt:.3f}")
print(f"r-correlation: {corr_r_lb:.3f}, p-value: {pval_p_lb:.3f}")

pd.DataFrame({
    'Index':['r-correlation','p-value'],
    'Luas Bangunan':[corr_r_lb,corr_r_lt],
    'Luas Tanah':[pval_p_lb,pval_p_lt] 
})
```

---

## 8. Hasil yang Diharapkan 
- `rumah_bekasi.csv` — file raw hasil scraping.
- Notebook `pengerjaan.ipynb` Precessing, statistic deskriptif, confidence interval, uji hipotesis, korelasi.
- Untuk Detail `pengerjaan.ipynb` Notebook.

---



