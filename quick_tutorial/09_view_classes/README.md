# **README – Organizing Views With View Classes (Step 09)**

##  1. Overview

Pada langkah ini, kita meningkatkan struktur view dengan mengelompokkan beberapa view ke dalam **view class**.

Sebelumnya, setiap view adalah *function* bebas. Namun ketika:

* beberapa view saling terkait,
* menggunakan template yang sama,
* membutuhkan state bersama (misalnya request object),
* atau merupakan bagian dari API REST,

maka mengelompokkannya dalam satu **class** adalah pilihan terbaik.

Langkah ini adalah pengenalan sederhana — di langkah berikutnya kita akan membahas view class secara lebih mendalam.

---

## 2. Struktur Proyek

```
view_classes/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    ├── home.pt
    └── tests.py
```

---

## 3. View Class dalam views.py

```python
from pyramid.view import (
    view_config,
    view_defaults
)

@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    def hello(self):
        return {'name': 'Hello View'}
```

Perubahan penting:

### 1. View menjadi *methods*

`home()` dan `hello()` kini adalah method pada class, bukan function bebas.

##  2. Konstruktor menyimpan request

Pyramid otomatis memanggil `__init__()` dan memasukkan request.

### 3. `@view_defaults` pada level class

Ini memberikan default deklaratif untuk SEMUA view dalam class:

* renderer
* permission
* request methods
* dan lain-lain

Dalam kasus ini, keduanya menggunakan template `home.pt`.

---

## 4. Unit Test Menggunakan Instans View Class

```python
request = testing.DummyRequest()
inst = TutorialViews(request)
response = inst.home()
self.assertEqual('Home View', response['name'])
```

Karena view kini adalah method, unit test:

* mengimpor class
* membuat instance dengan DummyRequest
* memanggil method (bukan function)
* memeriksa data yang dikembalikan

---

## 5. Functional Test Tidak Berubah

```python
res = self.testapp.get('/', status=200)
self.assertIn(b'<h1>Hi Home View', res.body)
```

Functional test tetap berjalan seperti sebelumnya karena view class tetap bekerja dengan WSGI app normal.

---

## 6. Menjalankan Tests

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
....
4 passed in 0.34 seconds
```

---

## 7. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Akses:

* [http://localhost:6543/](http://localhost:6543/)
* [http://localhost:6543/howdy](http://localhost:6543/howdy)

---

# **8. Analisis**

## 1. View Class Mengelompokkan View Terkait

`TutorialViews` kini menjadi rumah bagi dua view yang:

* menggunakan template yang sama,
* memiliki struktur atau tujuan mirip,
* bisa berbagi logic melalui method internal,
* dan bisa menyimpan state di `self`.

Hal ini mempermudah scaling aplikasi.

---

### 2. `@view_defaults` sebagai Konfigurasi Global

Dengan deklarasi:

```python
@view_defaults(renderer='home.pt')
```

Kita menghindari penulisan ulang renderer di setiap view.
Bisa juga digunakan untuk:

* `permission='view'`
* `request_method='GET'`
* `context=SomeClass`
* dll.

---

### 3. Test perlu diperbarui

Karena view kini method, unit test harus:

* instantiate class
* inject DummyRequest
* memanggil method langsung

Ini memberikan test yang lebih realistis, karena view class dapat berisi state dan helper functions.

---

### 4. Functional Test stabil

Karena deklarasi route & template tidak berubah, functional test tidak perlu disesuaikan.

---

# **9. Conclusion**

Pada langkah ini Anda telah mempelajari:

* bagaimana mengelompokkan view menjadi **view class**
* cara menggunakan decorator `@view_defaults` untuk konfigurasi global
* cara memodifikasi unit test untuk bekerja dengan objek class
* bagaimana Pyramid memanggil view class secara otomatis

Langkah ini adalah dasar penting untuk struktur aplikasi besar, terutama aplikasi dengan:

* banyak endpoint REST
* logic kompleks
* common helpers untuk view tertentu
* template yang digunakan bersama

---
<img width="960" height="1078" alt="praktikumpaw9" src="https://github.com/user-attachments/assets/987c0b09-e18e-4415-bd6e-21b2e618d1d5" />
<img width="960" height="1078" alt="praktikumpaw9s" src="https://github.com/user-attachments/assets/f67e23db-924b-43fe-9ec5-96bfa064d34c" />


