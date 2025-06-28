# Penalaran Komputer
Case Base Reasoning

# Deskripsi
Proyek ini adalah pipeline end-to-end untuk mengambil, membersihkan, dan menganalisis putusan fidusia dari situs Mahkamah Agung Republik Indonesia (putusan3.mahkamahagung.go.id). Pipeline ini mencakup lima tahap utama:

- Scraping: Mengunduh putusan fidusia (PDF dan metadata) dari situs MA.
- Pembersihan Data: Membersihkan teks PDF untuk fokus pada amar putusan.
- Case Retrieval: Menggunakan TF-IDF dan IndoBERT untuk mencari putusan relevan berdasarkan kueri.
- Prediksi Amar Putusan: Memprediksi amar putusan menggunakan pendekatan voting berbasis IndoBERT.
- Evaluasi: Mengevaluasi performa retrieval dan prediksi menggunakan metrik seperti precision, recall, dan F1-score.

Proyek ini dirancang untuk mendukung analisis hukum berbasis data, khususnya untuk kasus fidusia, dengan memanfaatkan teknik NLP dan machine learning.

# Prasyarat
Pastikan Anda memiliki alat berikut sebelum memulai:
- Python 3.8 atau lebih tinggi
- pip (Python package manager)
- Google Drive untuk menyimpan data (opsional, jika menggunakan Google Colab)
- Akses internet untuk scraping dan mengunduh model IndoBERT
- File requirements.txt dengan dependensi proyek

# Instalasi
Ikuti langkah-langkah berikut untuk mengatur proyek secara lokal atau di Google Colab:

1. Clone RepositoriClone proyek dari repositori:
git clone git clone https://github.com/Zero4427/PK.git

2. Masuk ke Direktori Proyek  
cd PK

3. Buat dan Aktifkan Virtual Environment (opsional, direkomendasikan):
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

4. Instal DependensiInstal semua dependensi yang tercantum dalam requirements.txt:
pip install -r requirements.txt

Isi requirements.txt (contoh disediakan di bawah) mencakup:
- pandas
- requests
- beautifulsoup4
- pdfminer.six
- nltk
- torch
- transformers
- scikit-learn
- numpy

5. Konfigurasi Google Drive (jika menggunakan Colab) Mount Google Drive untuk menyimpan data:
from google.colab import drive
drive.mount('/content/drive')

6. Unduh Data NLTKUnduh stopwords untuk pembersihan teks:
import nltk
nltk.download('stopwords')

# Struktur Direktori
penalaran-komputer/
├── data/
│   ├── eval/                       # Hasil evaluasi (metrik retrieval dan prediksi)
│   ├── results/                    # Hasil prediksi amar
│   └── queries_fidusia.json        # File kueri untuk evaluasi
├── CSV/                           # Output CSV dari scraping dan pembersihan
├── PDF/                           # File PDF putusan yang diunduh
├── scripts/
│   └── penalaran_komputer.py       # Skrip utama untuk semua tahap
├── requirements.txt                # Daftar dependensi
└── README.md                       # Dokumentasi proyek

# Menjalankan Pipeline End-to-End
Pipeline ini mencakup lima tahap yang dijalankan secara berurutan dalam penalaran_komputer.py. Berikut langkah-langkah untuk menjalankan pipeline secara keseluruhan:

1. Scraping Putusan Fidusia (Tahap 1)Mengunduh metadata dan file PDF dari situs MA (maksimal 50 putusan):
python scripts/penalaran_komputer.py --scrape

- Output: File CSV (putusan_fidusia_<tanggal>.csv) di CSV4/ dan file PDF di PDF4/.
- Catatan: Pastikan koneksi internet stabil dan Google Drive sudah di-mount.

2. Pembersihan Data (Tahap 2)Membersihkan teks PDF untuk fokus pada amar putusan:
python scripts/penalaran_komputer.py --clean

- Input: File CSV dari tahap 1 (misalnya, putusan_fidusia_2025-06-24.csv).
- Output: File CSV yang sudah dibersihkan (putusan_fidusia_cleaned_FINAL_BERSIH_FIX.csv) di CSV/.
- Proses: Menghapus watermark, disclaimer, dan teks administratif; mengekstrak amar putusan.

3. Case Retrieval (Tahap 3)Mencari putusan relevan menggunakan TF-IDF dan IndoBERT berdasarkan kueri:
python scripts/penalaran_komputer.py --retrieve

- Input: File CSV yang sudah dibersihkan dan kueri contoh (misalnya, "Terdakwa menyewakan objek fidusia tanpa persetujuan").
- Output: Top-5 putusan relevan dengan skor similaritas (ditampilkan di terminal).
- Catatan: Membutuhkan model IndoBERT dari indobenchmark/indobert-base-p1.

4. Prediksi Amar Putusan (Tahap 4)Memprediksi amar putusan berdasarkan kueri menggunakan voting berbasis IndoBERT:
python scripts/penalaran_komputer.py --predict

- Input: File queries_fidusia.json di data/ dan file CSV yang sudah dibersihkan.
- Output: File CSV (prediksi_amar_fidusia.csv) di data/results/ dengan prediksi amar dan top-5 case ID.
- Proses: Menggunakan embedding IndoBERT untuk mencocokkan kueri dengan dokumen.

5. Evaluasi (Tahap 5)Mengevaluasi performa retrieval dan prediksi menggunakan metrik precision, recall, dan F1-score:
python scripts/penalaran_komputer.py --evaluate

- Input: File queries_fidusia_baru.json (dengan ground truth) dan file prediksi.
- Output: 
retrieval_metrics.csv: Metrik retrieval untuk setiap kueri.
prediction_metrics.csv: Metrik akurasi, precision, recall, dan F1 untuk prediksi amar.
- Hasil ditampilkan di terminal (misalnya, akurasi, precision, recall, F1).

# Menjalankan Seluruh PipelineUntuk menjalankan semua tahap sekaligus:
python scripts/penalaran_komputer.py --all

- Catatan: Pastikan file queries_fidusia.json dan queries_fidusia_baru.json tersedia di data/. Anda mungkin perlu menyesuaikan path file di skrip sesuai lokasi data Anda.

# Contoh Perintah
Berikut beberapa contoh perintah untuk kasus penggunaan tertentu:

- Scraping 50 putusan fidusia:
python scripts/penalaran_komputer.py --scrape
Output: CSV/putusan_fidusia_<tanggal>.csv dan folder PDF/ dengan file PDF.

- Membersihkan teks PDF:
python scripts/penalaran_komputer.py --clean --input CSV/putusan_fidusia_2025-06-24.csv
Output: CSV/putusan_fidusia_cleaned_FINAL_BERSIH_FIX.csv.

- Mencari putusan relevan dengan kueri:
python scripts/penalaran_komputer.py --retrieve --query "Terdakwa menyewakan objek fidusia tanpa persetujuan"
Output: Daftar top-5 case ID dengan skor similaritas (TF-IDF dan IndoBERT).

- Memprediksi amar untuk kueri tertentu:
python scripts/penalaran_komputer.py --predict --query "Terdakwa menyewakan objek fidusia tanpa persetujuan"
Output: Prediksi amar dan top-5 case ID di terminal.

- Evaluasi hasil prediksi:
python scripts/penalaran_komputer.py --evaluate
Output: File retrieval_metrics.csv dan prediction_metrics.csv di data/eval/.

# Hasil
- Tahap 1: Menghasilkan hingga 50 file PDF putusan fidusia dan file CSV dengan metadata (nomor, tahun, hakim, amar, dll.) di CSV/ dan PDF/.
- Tahap 2: File CSV yang sudah dibersihkan dengan kolom text_pdf_cleaned berisi amar putusan yang telah dihapus watermark dan teks administratif.
- Tahap 3: Daftar top-5 putusan relevan untuk setiap kueri berdasarkan TF-IDF dan IndoBERT, ditampilkan dengan skor similaritas.
- Tahap 4: File CSV (prediksi_amar_fidusia.csv) berisi prediksi amar putusan untuk setiap kueri di queries_fidusia.json.
- Tahap 5: Dua file evaluasi:
retrieval_metrics.csv: Precision, recall, dan F1-score untuk retrieval per kueri.
prediction_metrics.csv: Akurasi, precision, recall, dan F1-score untuk prediksi amar.



Kontribusi
Kami menyambut kontribusi! Ikuti langkah-langkah berikut:
- Fork repositori ini dari https://github.com/Zero4427/PK.git.
- Buat branch baru: git checkout -b fitur-baru
- Commit perubahan:git commit -m "Menambahkan fitur X atau perbaikan Y"
- Push ke branch:git push origin fitur-baru
- Buat Pull Request di GitHub.
