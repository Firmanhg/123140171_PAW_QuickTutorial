# **README – Functional Testing with WebTest (Step 06)**

## 1. Overview

Pada langkah ini, kita memperluas mekanisme testing dengan menggunakan **WebTest**, yaitu library Python untuk melakukan **functional testing** — sebuah jenis pengujian end-to-end pada seluruh stack web.

Berbeda dengan unit test yang hanya menguji fungsi tunggal seperti view, functional test:

* mensimulasikan permintaan HTTP nyata
* menjalankan seluruh WSGI app
* memproses routing, view, response, templating, middleware, dll
* tanpa memerlukan server HTTP sebenarnya

Ini menjadikan functional test cepat, lengkap, dan cocok untuk TDD.

---

## 2. Struktur Proyek

```
functional_testing/
├── setup.py
└── tutorial/
    ├── __init__.py
    └── tests.py
```

---

## 3. Menambahkan WebTest ke setup.py

Tambahkan ke extras `[dev]`:

```python
dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
    'webtest',
]
```

Kemudian install:

```
$VENV/bin/pip install -e ".[dev]"
```

---

## 4. Functional Test di tests.py

Tambahkan class baru:

```python
class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp

        self.testapp = TestApp(app)

    def test_hello_world(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<h1>Hello World!</h1>', res.body)
```

**Penjelasan:**

* `TestApp(app)` → membungkus WSGI app Pyramid menjadi objek yang bisa menerima request HTTP
* `self.testapp.get('/')` → mensimulasikan GET request ke `/`
* `status=200` → memastikan kode status sesuai
* `res.body` → body response dalam bentuk *bytes*
* `assertIn(...)` → mengecek apakah HTML tertentu ada di response

---

## 5. Menjalankan tes

```
$VENV/bin/pytest tutorial/tests.py -q
```

Hasil:

```
..
2 passed in 0.25 seconds
```

Tes pertama = unit test
Tes kedua = functional test dengan WebTest

---

# **6. Analisis**

### WebTest = full-stack testing tanpa server

Dengan WebTest:

* tidak perlu menjalankan waitress atau pserve
* tidak perlu open browser
* seluruh layer web Pyramid berjalan dari view hingga response final
* testing lebih cepat dan lebih realistis

WebTest adalah solusi ideal untuk menguji:

* routing
* response HTML
* error view
* templating
* middleware
* integrasi kecil dalam stack Pyramid

---

### Functional test hidup berdampingan dengan unit test

Kedua jenis tes dijalankan oleh pytest dalam satu eksekusi:

* cepat
* mudah dikelola
* menyatu di pipeline CI/CD

Tidak perlu konfigurasi tambahan.

---

### Mengapa app = main({})?

`main({})` memanggil application factory dan menghasilkan WSGI app murni.

Karena tidak ada environment, kita cukup beri dictionary kosong `{}` sebagai konfigurasi.

---

# **7. Extra Credit – Jawaban**

## **Mengapa functional test menggunakan `b''`?**

Karena:

### 1. `res.body` selalu berupa **bytes**, bukan string

WSGI specification mengharuskan body response dikembalikan sebagai **bytes**, bukan teks Python (`str`).

Oleh sebab itu:

```python
res.body == b"..."
```

bukan:

```python
res.body == "..."
```

### 2. WebTest mengikuti standar WSGI

WebTest tidak mengubah body menjadi string, sehingga kita perlu membandingkan byte-by-byte.

### 3. Jika ingin string, gunakan `res.text`

WebTest juga menyediakan:

```python
res.text
```

yang otomatis decode ke unicode string berdasarkan response headers.

Tetapi untuk testing HTML, menggunakan bytes lebih aman dan netral.

Contoh alternatif:

```python
self.assertIn("<h1>Hello World!</h1>", res.text)
```

Namun tutorial menggunakan `b''` karena mengikuti spesifikasi WSGI.

---

# **8. Kesimpulan**

Pada langkah ini kamu telah belajar:

* menambahkan functional testing dengan WebTest
* membuat test end-to-end yang menguji seluruh WSGI stack
* membedakan unit test dan functional test
* memahami cara mengecek response HTML dalam bentuk bytes

Ini adalah fondasi penting sebelum kamu masuk ke bab berikutnya yang berhubungan dengan **templating**, **routing kompleks**, atau **form submission**.

---

<img width="613" height="312" alt="praktikumpaw6" src="https://github.com/user-attachments/assets/24f49ff3-16d3-426f-9e6b-3dc730a41092" />

