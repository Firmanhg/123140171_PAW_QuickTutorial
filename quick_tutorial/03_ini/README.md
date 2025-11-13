---

# **README – Application Configuration with .ini Files (Step 03)**

##  1. Overview

Pada langkah ini, kita mulai menggunakan **file konfigurasi .ini** dan perintah **pserve** untuk menjalankan aplikasi Pyramid.
Pendekatan ini memperkenalkan mekanisme konfigurasi tingkat lanjut yang memisahkan **logika aplikasi** dari **pengaturan aplikasi**.

Pyramid memanfaatkan konsep *entry point* dari Setuptools untuk memberi tahu dimana WSGI app berada, sehingga aplikasi dapat dijalankan menggunakan perintah seperti:

```
pserve development.ini --reload
```

---

## 2. Struktur Proyek

```
ini/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    └── app.py   (dihapus)
```

---

## 3. Konfigurasi pada setup.py

Kita menambahkan *entry point* baru:

```python
entry_points={
    'paste.app_factory': [
        'main = tutorial:main'
    ],
},
```

Entry point inilah yang digunakan `pserve` untuk menemukan fungsi `main()` yang akan membangun WSGI application.

---

## 4. Contoh development.ini

```ini
[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

* `[app:main]` → lokasi aplikasi WSGI
* `[server:main]` → konfigurasi server (Waitress)

---

## 5. Kode Aplikasi di **init**.py

```python
from pyramid.config import Configurator
from pyramid.response import Response

def hello_world(request):
    return Response('<body><h1>Hello World!</h1></body>')

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
```

Fungsi `main()` adalah *factory function* yang membangun dan mengembalikan WSGI app.

---

## 6. Menjalankan Aplikasi

1. Install project:

   ```
   $VENV/bin/pip install -e .
   ```
2. Jalankan:

   ```
   $VENV/bin/pserve development.ini --reload
   ```
3. Buka:

   ```
   http://localhost:6543/
   ```

---

# **7. Analisis**

### Bagaimana pserve bekerja?

1. `pserve` membaca file **development.ini**
2. Pada bagian `[app:main]`, ditemukan:

   ```
   use = egg:tutorial
   ```
3. Egg tersebut memiliki entry point:

   ```
   main = tutorial:main
   ```
4. Pyramid memanggil `tutorial.main(global_config, **settings)`
5. Fungsi `main()` membuat dan mengembalikan WSGI app
6. `[server:main]` menginstruksikan pserve untuk menggunakan waitress sebagai server
7. Logging juga dikonfigurasi via .ini

---

### Mengapa memindahkan kode startup ke **init**.py?

Karena:

* Struktur ini mengikuti pola Pyramid standar
* Memudahkan penggunaan entry point
* Memudahkan deploy, otomatis ditemukan oleh pserve
* Code menjadi lebih rapi dan terorganisir

---

### Apa fungsi `--reload`?

Pserve akan:

* memantau perubahan file .py dan .ini
* otomatis merestart server saat file berubah

Ini sangat membantu saat development.

---

# **8. Extra Credit (Jawaban Lengkap)**

---

## 1. Jika tidak suka .ini file, apakah bisa semuanya dilakukan dari Python?

**Ya, bisa.**

Kita dapat:

* mengonfigurasi server WSGI dari Python,
* memanggil fungsi `main()` secara manual,
* menjalankan Waitress langsung:

```python
from waitress import serve
app = main({}, **{})
serve(app, host='0.0.0.0', port=6543)
```

Namun, kamu kehilangan:

* logging configuration
* fleksibilitas (dev/prod/staging)
* entry point otomatis
* kemudahan pserve (reload, hot reload)
* standar industri Pyramid

Karenanya `.ini` tetap best practice.

---

## 2. Bisakah kita punya banyak file .ini? Mengapa?

**Bisa, dan ini sangat umum.**

Contohnya:

```
development.ini
production.ini
testing.ini
staging.ini
docker.ini
```

Mengapa perlu banyak?

* Konfigurasi **development** biasanya memakai `--reload` dan debug mode
* **Production** memerlukan port berbeda, worker berbeda, logging ketat
* **Testing** butuh database lain atau mock service
* Environment berbeda = konfigurasi berbeda

File .ini memberi fleksibilitas tinggi tanpa mengubah kode.

---

## 3. Mengapa entry point di setup.py tidak menyebutkan **init**.py?

Karena Python package import bekerja berdasarkan **package name**, bukan file path.

`tutorial:main` berarti:

* import module `tutorial`
* cari objek bernama `main` di dalamnya

Python tahu bahwa `tutorial` merujuk ke `tutorial/__init__.py`.
File tersebut otomatis diperlakukan sebagai module.

Jadi tidak perlu menulis:

```
tutorial.__init__:main
```

Karena itu redundant.

---

## 4. Apa tujuan dari `**settings`? Apa arti `**`?

### Fungsi `**settings`

`settings` berisi semua pasangan key-value dari file `.ini`, misalnya:

```
[app:main]
foo = bar
debug = true
```

Akan dipetakan menjadi dictionary:

```python
{
  'foo': 'bar',
  'debug': 'true'
}
```

Lalu diteruskan ke Configurator.

---

### Arti `**`

`**` berarti:

* *dictionary unpacking*
* menerima argument variadic key-value
* contoh pemanggilan:

```python
main({}, foo='bar', debug=True)
```

Akan masuk ke parameter:

```python
settings = {'foo': 'bar', 'debug': True}
```

Ini mekanisme Python untuk menerima opsi konfigurasi yang fleksibel.

---

# **9. Kesimpulan**

Pada langkah ini kamu belajar:

* menjalankan Pyramid dengan `.ini` file
* menggunakan entry points pada setup.py
* memahami cara pserve menemukan WSGI app
* memindahkan kode bootstrap ke **init**.py
* memahami konsep konfigurasi berbasis file

<img width="957" height="340" alt="praktikumpaw3" src="https://github.com/user-attachments/assets/98479d7b-9cd2-483a-9e2d-b406f2ff1992" />
