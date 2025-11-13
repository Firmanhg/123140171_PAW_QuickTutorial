---

# **README – Python Packages for Pyramid Applications**

##  1. Overview

Pada langkah ini, kita memindahkan aplikasi Pyramid dari satu file sederhana menjadi **Python package** yang lebih terstruktur.
Tujuannya agar aplikasi bisa diorganisir, di-*install*, dan dikembangkan menggunakan praktik standar Python modern.

Dengan membuat folder `tutorial` sebagai package (ditandai oleh `__init__.py`) dan menambahkan `setup.py`, kita membangun pondasi struktur proyek Pyramid yang lebih profesional.

---

## 2. Struktur Proyek

```
package/
│
├── setup.py
│
└── tutorial/
    ├── __init__.py
    └── app.py
```

---

## 3. setup.py (File Proyek)

File ini menjadikan project kita dapat di-*install* menggunakan pip dan berjalan dalam “development mode”.

Isi file:

```python
from setuptools import setup

requires = [
    'pyramid',
    'waitress',
]

setup(
    name='tutorial',
    install_requires=requires,
)
```

---

## 4. Cara Menjalankan

1. Pindah direktori:

   ```bash
   cd ..; mkdir package; cd package
   ```
2. Buat file `setup.py`.
3. Install project dalam mode editable:

   ```bash
   $VENV/bin/pip install -e .
   ```
4. Buat package folder:

   ```bash
   mkdir tutorial
   ```
5. Isi file `tutorial/__init__.py`:

   ```python
   # package
   ```
6. Isi file `tutorial/app.py` (kode Hello World sama seperti Step 01).
7. Jalankan aplikasi:

   ```bash
   $VENV/bin/python tutorial/app.py
   ```
8. Buka di browser:

   ```
   http://localhost:6543/
   ```

---

# **5. Analisis**

### Python Package = Unit Organisasi

Dengan menambahkan `__init__.py`, folder `tutorial/` berubah menjadi **package** Python.
Ini membuat kode lebih mudah:

* di-*import*
* didistribusikan
* dipasang secara modular

---

### Python Project via setup.py

`setup.py` memungkinkan project untuk:

* memiliki dependency resmi (`install_requires`)
* di-*install* memakai pip (`pip install -e .`)
* digunakan dalam mode development (*editable mode*)

Mode development berarti setiap perubahan di source code langsung berlaku tanpa re-install.

---

### Struktur Ini = Best Practice Python Modern

Framework modern seperti Pyramid, Django, FastAPI, Flask (dengan blueprints) sangat mengandalkan package.
Dengan struktur ini kamu siap membangun aplikasi yang lebih besar dan terorganisir.

---

### Menjalankan modul langsung adalah “opsi buruk”

Menjalankan:

```
python tutorial/app.py
```

…sebenarnya *bukan praktik ideal* karena:

* modul dalam package biasanya tidak dijalankan langsung
* seharusnya menggunakan entry point atau perintah seperti `pserve`

Namun untuk tujuan edukasi, tutorial ini menunjukkan proses dasar cara Python memuat package.

---

# **6. Kesimpulan**

Pada langkah ini kamu berhasil:

* Membuat Python package (`tutorial`)
* Membuat project Pyramid dengan `setup.py`
* Menginstall project dalam mode editable (`pip install -e .`)
* Menjalankan aplikasi Pyramid dari package

Langkah ini adalah fondasi penting sebelum aplikasi diperluas dengan konfigurasi, templating, dan komponen Pyramid lainnya.

---
<img width="957" height="340" alt="Screenshot 2025-11-13 190710" src="https://github.com/user-attachments/assets/8fb6ec69-31f6-4b1c-a438-24f1e691a084" />

