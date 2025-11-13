# **README – Basic Web Handling With Views (Step 07)**

## 1. Overview

Pada langkah ini, kita mulai mengorganisir kode aplikasi Pyramid dengan lebih baik:

* Memindahkan view ke file terpisah (`views.py`)
* Menggunakan **decorator `@view_config`** untuk deklaratif konfigurasi view
* Menjalankan scanning module (`config.scan`) untuk mendaftarkan view otomatis
* Menambahkan lebih dari satu view dan menyesuaikan routing
* Memperbarui unit dan functional tests

Tujuannya adalah memperkenalkan cara standar Pyramid dalam mengorganisir bagian aplikasi web.

---

## 2. Struktur Proyek

```
views/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    └── tests.py
```

---

## 3. Kode Startup Baru di **init**.py

```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

Perubahan penting:

* View tidak lagi ada di sini
* Menggunakan `config.scan('.views')` untuk mencari decorator `@view_config` pada module tersebut
* Menambahkan dua route: `/` dan `/howdy`

---

## 4. File views.py Dengan Decorators

```python
from pyramid.response import Response
from pyramid.view import view_config

@view_config(route_name='home')
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')

@view_config(route_name='hello')
def hello(request):
    return Response('<body>Go back <a href="/">home</a></body>')
```

Keuntungan:

* View terorganisir rapi
* Tidak perlu menambahkan `config.add_view` secara manual
* Deklaratif, lebih mudah di-maintain

---

## 5. Update Unit Test

```python
def test_home(self):
    from .views import home
    request = testing.DummyRequest()
    response = home(request)
    self.assertEqual(response.status_code, 200)
    self.assertIn(b'Visit', response.body)
```

```python
def test_hello(self):
    from .views import hello
    request = testing.DummyRequest()
    response = hello(request)
    self.assertEqual(response.status_code, 200)
    self.assertIn(b'Go back', response.body)
```

---

## 6. Functional Tests

```python
def test_home(self):
    res = self.testapp.get('/', status=200)
    self.assertIn(b'<body>Visit', res.body)

def test_hello(self):
    res = self.testapp.get('/howdy', status=200)
    self.assertIn(b'<body>Go back', res.body)
```

---

## 7. Menjalankan Tes

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
....
4 passed in 0.28 seconds
```

---

## 8. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Lalu buka:

* [http://localhost:6543/](http://localhost:6543/)
* [http://localhost:6543/howdy](http://localhost:6543/howdy)

---

# **9. Analisis**

Pada langkah ini, kita melakukan beberapa penyempurnaan arsitektur:

### 1. Memindahkan view ke file terpisah

Memisahkan view dari file startup membuat struktur project:

* lebih bersih
* lebih mudah diperluas
* mengikuti pola Pyramid yang idiomatis

---

### 2. Menggunakan `@view_config`

Deklaratif konfigurasi dengan decorator membuat kode:

* lebih mudah dibaca
* lebih dekat dengan view itu sendiri
* tidak perlu konfigurasi manual di `__init__.py`

Kedua pendekatan (`config.add_view` dan decorator) sah, namun decorator lebih umum digunakan.

---

### 3. Menggunakan `config.scan()`

Pyramid akan:

* mencari decorator `@view_config`
* mendaftarkan view otomatis
* menyederhanakan startup code

---

### 4. Routing Lebih Fleksibel

URL, nama route, dan nama fungsi view dapat berbeda.
Routing diatur oleh `config.add_route`, bukan nama fungsi.

---

# **10. Extra Credit – Jawaban**

---

## 1. Apa arti tanda titik (.) dalam `.views` pada `config.scan('.views')`?

Tanda titik **mengacu pada package relatif**, bukan absolute import.

* `'.views'` berarti:
  **scan module views.py yang berada di dalam package yang sama dengan **init**.py**

Kalau ditulis tanpa titik (`views`), Pyramid akan mencari module global bernama `views`, yang mungkin tidak ada.

`'.views'` = relative module import
`'views'` = absolute module import

---

## 2. Mengapa `assertIn` lebih baik daripada `assertEqual` untuk menguji response body?

Karena:

### 1. Response HTML biasanya lebih panjang

Kita tidak perlu mencocokkan seluruh halaman HTML — cukup memastikan **bagian penting** ada.

### 2. HTML bisa berubah format

Misalnya:

* spasi berubah
* atribut ditambah
* line break berubah

`assertEqual` akan gagal meskipun hasilnya benar secara fungsional.

### 3. Test jadi lebih fleksibel dan robust

Dengan:

```python
self.assertIn(b'Visit', res.body)
```

Test tetap lulus walaupun HTML lengkap berubah, selama bagian penting tetap ada.

### 4. Sesuai untuk functional testing

Yang penting: response berisi teks yang ingin kita uji, bukan keseluruhan dokumen.

---

# **11. Kesimpulan**

Pada langkah ini kamu belajar:

* memindahkan view ke module khusus (`views.py`)
* menggunakan `@view_config` untuk deklaratif konfigurasi
* melakukan scanning module dengan `config.scan()`
* menambah dua view baru dengan route berbeda
* memperbarui unit dan functional tests
* memahami perbedaan assertIn vs assertEqual dalam pengujian HTML

Struktur aplikasi kini jauh lebih terorganisir dan siap untuk fitur yang lebih kompleks seperti templating, form handling, dan route mapping lanjutan.

---
<img width="957" height="1077" alt="praktikumpaw7" src="https://github.com/user-attachments/assets/0a0953d1-2f46-4bde-b336-46c02d38ea75" />
<img width="950" height="1078" alt="praktikumpaw7s" src="https://github.com/user-attachments/assets/010f0764-f41f-4682-bd7b-4fae56ffc456" />


