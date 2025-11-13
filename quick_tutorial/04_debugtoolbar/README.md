---

# **README – Easier Development with pyramid_debugtoolbar (Step 04)**

## 1. Overview

Pada langkah ini, kita menambahkan **pyramid_debugtoolbar**, sebuah add-on populer yang mempermudah proses debugging dan development aplikasi Pyramid. Add-on ini menyediakan:

* Error page interaktif
* Panel informasi request, response, routing, SQL, konfigurasi, dll
* Console interaktif saat terjadi error
* Debugging langsung dari browser

Langkah ini bertujuan memperkenalkan cara menggunakan **Setuptools extras**, serta bagaimana Pyramid dapat mengonfigurasi add-on melalui file `.ini` tanpa mengubah kode aplikasi.

---

##2. Struktur Proyek

```
debugtoolbar/
├── setup.py
├── development.ini
└── tutorial/
    └── __init__.py
```

---

## 3. Menambahkan Add-on ke setup.py

Kita menambahkan dependency opsional (extras):

```python
dev_requires = [
    'pyramid_debugtoolbar',
]

setup(
    name='tutorial',
    install_requires=requires,
    extras_require={
        'dev': dev_requires,
    },
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
```

Add-on dimasukkan sebagai **extras_require** → bukan dependency utama aplikasi, melainkan kebutuhan developer.

---

## 4. Instalasi dengan extras `[dev]`

Kita install dengan:

```
$VENV/bin/pip install -e ".[dev]"
```

Bagian `".[dev]"` artinya:

* install project ini (`.`)
* plus dependency ekstra kategori **dev**

---

## 5. Konfigurasi di development.ini

Tambahkan:

```ini
pyramid.includes =
    pyramid_debugtoolbar
```

Setiap kali aplikasi dijalankan dengan `.ini` ini, Pyramid akan mengaktifkan debugtoolbar.

---

## 6. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Toolbar akan muncul di sisi kanan browser, berupa tombol berbentuk panel.

---

# **7. Analisis**

### 🔹 Apa itu Pyramid Add-on?

Add-on adalah package Python yang menyediakan:

* konfigurasi tambahan
* view, route, panel debugger
* integrasi ke Pyramid melalui `config.include()`

`pyramid_debugtoolbar` adalah contoh add-on yang:

* terinstall melalui pip
* dikonfigurasi melalui `.ini` (atau melalui `config.include`)

---

### 🔹 Mengapa konfigurasi dilakukan lewat `.ini`?

Karena:

* memudahkan mengaktifkan/mematikan fitur tanpa mengubah kode
* memisahkan concerns: **kode** untuk aplikasi, **konfigurasi** untuk environment
* fleksibel untuk development / testing / production

Menghapus ini:

```ini
pyramid_debugtoolbar
```

langsung menonaktifkan toolbar.

---

### 🔹 Kenapa debugtoolbar membantu?

Karena:

* memberi stack trace interaktif
* menyediakan console interaktif Python saat error
* menunjukkan route, request, SQL query (jika ada)
* memberi introspeksi mendalam aplikasi

Sangat membantu developer ketika mencari error.

---

### 🔹 Tentang Setuptools extras

`extras_require` adalah daftar fitur opsional proyek.
Misalnya:

* `[dev]` → dependency untuk development
* `[tests]` → dependency untuk testing
* `[docs]` → dependency untuk build dokumentasi

Kita hanya menginstall extras ketika dibutuhkan.

---

# **8. Extra Credit – Jawaban Lengkap**

---

## 1. Mengapa pyramid_debugtoolbar dimasukkan ke `dev_requires` dan bukan ke `requires`?

Jawaban:

* Karena debugtoolbar **hanya untuk development**, bukan untuk production.

* Production environment **tidak memerlukan**, dan **tidak boleh** mengaktifkan debugtoolbar.

* Debugtoolbar membuat:

  * tampilan error lebih detail
  * akses ke console Python langsung
  * potensi security issue jika diaktifkan pada server publik

* Dengan memisahkan ke `[dev]`, developer dapat memilih:

```
pip install -e ".[dev]"
```

sementara deployment cukup:

```
pip install -e .
```

→ Dependency development dan production bisa berbeda.

---

## 2. Buat bug dan lihat hasilnya

Jika kamu mengganti:

```python
return Response(...)
```

dengan:

```python
return xResponse(...)
```

Maka:

* terjadi `NameError: name 'xResponse' is not defined`
* debugtoolbar akan menampilkan halaman error interaktif
* kamu dapat melihat stack trace lengkap

### Console Interaktif

Pada bagian terbawah error page, klik ikon "▶" atau "screen" untuk membuka interactive console.

Cobalah:

```
request
```

Hasil:

* object request Pyramid
* bisa melihat atribut seperti:

  * request.method
  * request.url
  * request.params
  * request.headers

Cobalah:

```
Response
```

Hasil:

* class Response dari Pyramid
* kamu bisa membuat response di console:

  ```
  Response("test")
  ```

Kamu juga bisa:

* mengeksekusi kode Python lain
* memeriksa variable context
* memanggil fungsi di modul

Interactive console adalah salah satu fitur debugging paling powerful.

---

# **9. Kesimpulan**

Pada langkah ini kamu belajar:

* bagaimana menambah Pyramid add-on
* cara mengonfigurasi add-on melalui `.ini`
* konsep Setuptools extras dan instalasi `[dev]`
* penggunaan debugtoolbar untuk debugging interaktif
* memanfaatkan error page yang kaya informasi

Ini menjadikan development Pyramid jauh lebih mudah dan efisien.

---
<img width="952" height="1078" alt="praktikumpaw4" src="https://github.com/user-attachments/assets/6cdc620c-5098-44c2-be94-cfd3d2714353" />
