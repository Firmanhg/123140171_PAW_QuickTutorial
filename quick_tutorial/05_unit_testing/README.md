# **README – Unit Tests and pytest (Step 05)**

## 1. Overview

Pada langkah ini, kita menambahkan **unit test** untuk memastikan kualitas kode Pyramid kita.
Testing adalah bagian penting dalam pengembangan perangkat lunak karena:

* membantu mencegah bug
* memastikan perubahan kode tidak merusak fitur lain
* mempercepat development (tidak perlu cek manual lewat browser)

Pyramid memiliki dukungan kuat terhadap testing, dan tutorial ini menggunakan **pytest**, test runner populer yang banyak dipakai komunitas Python.

---

## 2. Struktur Proyek

```
unit_testing/
├── setup.py
├── tutorial/
│   ├── __init__.py
│   └── tests.py
└── development.ini
```

---

## 3. Menambahkan pytest ke setup.py

Kita menambahkan pytest sebagai dependency dalam extras `[dev]`:

```python
dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
]
```

Extras `[dev]` digunakan agar dependency untuk development/testing **tidak ikut terinstall** pada production.

---

## 4. Instalasi

```
$VENV/bin/pip install -e ".[dev]"
```

Ini menginstall project + seluruh dependency development (debugtoolbar + pytest).

---

## 5. Membuat Unit Test

File: `tutorial/tests.py`

```python
import unittest

from pyramid import testing


class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_hello_world(self):
        from tutorial import hello_world

        request = testing.DummyRequest()
        response = hello_world(request)
        self.assertEqual(response.status_code, 200)
```

Test di atas:

* membuat config Pyramid palsu
* membuat request palsu
* memanggil view langsung
* memastikan hasilnya sesuai ekspektasi

---

## 6. Menjalankan Tes

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
.
1 passed in 0.14 seconds
```

---

# **7. Analisis**

### Pyramid mendukung testing secara mendalam

Pyramid menyediakan modul `pyramid.testing` untuk:

* membuat config palsu
* membuat DummyRequest (menyerupai request asli)
* meniru environment Pyramid tanpa menjalankan server

Ini membuat testing cepat dan mudah.

---

### Mengapa import view dilakukan di dalam metode test?

Karena:

* import modul bisa mengeksekusi kode (side effects)
* jika import dilakukan di awal file test, bagian lain dari test bisa “tercemar”
* unit test harus benar-benar *terisolasi*
* Pyramid application state (config) harus fresh di tiap test

Inilah prinsip “unit” dalam unit testing.

---

### Mengapa kita hanya menguji status code?

Karena:

* itu adalah kontrak dasar yang dijanjikan oleh view
* mudah diuji
* view pada tahap ini belum memiliki logika kompleks

Di tahap selanjutnya, kamu bisa menambahkan test konten HTML, header, dan lain-lain.

---

# **8. Extra Credit – Jawaban Lengkap**

---

## 1. Ubah test untuk assert status 404, lalu jalankan pytest

Ubah:

```python
self.assertEqual(response.status_code, 404)
```

Hasil pytest:

```
>       assert 200 == 404
E       assert 200 == 404
```

Artinya:

* view mengembalikan status 200
* tetapi test mengharapkan 404
* pytest menampilkan expected vs actual
* kamu bisa lihat dengan jelas nilai yang salah

Ini contoh bagaimana test membantu mengetahui error lebih cepat daripada mengecek di browser.

---

## 2. Tambahkan bug ke view, lalu jalankan test

Ubah:

```python
return Response(...)
```

Menjadi:

```python
return xResponse(...)
```

Jalankan pytest:

```
NameError: name 'xResponse' is not defined
```

Kelebihan test dibanding cek manual:

* error muncul **langsung**
* stack trace jelas
* tidak perlu restart server
* tidak perlu buka browser dan mengklik ulang

Testing = lebih efisien.

---

## 3. Cara mengubah response status code & mengujinya

Contoh:

```python
return Response("Not Found", status=404)
```

Test:

```python
self.assertEqual(response.status_code, 404)
```

Testing memastikan kontrak fungsi terpenuhi.

---

## 4. Bagaimana menambahkan assert untuk mengecek body HTML?

Gunakan:

```python
self.assertIn(b"<h1>Hello World</h1>", response.body)
```

Catatan penting:

* `response.body` selalu berupa **bytes**, bukan string
* maka cari dengan literal `b"..."`

Jika mau testing string, gunakan:

```python
self.assertIn("<h1>Hello World</h1>", response.text)
```

---

## 5. Mengapa import hello_world dilakukan dalam metode test, bukan di top-level?

Karena:

1. **Isolasi test**
   Import dapat menjalankan kode. Memasukkannya per test memastikan environment bersih.

2. **Menghindari state contamination**
   State Pyramid (configurator) berubah tiap test. Import di luar dapat menggunakan state lama.

3. **Menghindari error early**
   Jika import gagal, seluruh file test gagal sebelum menjalankan test mana pun.

4. **Unit Testing Philosophy**
   Tiap test harus berdiri sendiri.

Itulah alasan test Pyramid selalu mengimpor view **di dalam metode test**, bukan di atas modul.

---

# **9. Kesimpulan**

Pada langkah ini kamu belajar:

* menggunakan pytest untuk menjalankan unit test
* membuat DummyRequest untuk mengetes view
* memahami setup/teardown Configurator
* menulis test untuk response status code
* membuat test yang aman dan terisolasi
* debugging lebih cepat dibanding lewat browser

---
<img width="671" height="307" alt="praktikumpaw5" src="https://github.com/user-attachments/assets/d2dac48b-f25d-41e8-8ffc-29defcb60f49" />

