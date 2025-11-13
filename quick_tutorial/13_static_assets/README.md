# **README – CSS/JS/Images Files With Static Assets (Step 13)**

## 1. Overview

Pada langkah ini, kita belajar bagaimana Pyramid melayani **static assets** seperti:

* CSS
* JavaScript
* Gambar (PNG, JPG, SVG)

Static assets adalah bagian penting dari aplikasi web modern. Dengan Pyramid, Anda dapat:

* mendaftarkan direktori static
* melayani file dari URL tertentu
* menghasilkan URL static yang fleksibel (tidak hardcoded)
* memastikan template tetap sinkron dengan konfigurasi routing

Langkah ini memperkenalkan **config.add_static_view** serta penggunaan helper **request.static_url**.

---

## 2. Struktur Proyek

```
static_assets/
├── tutorial/
│   ├── __init__.py
│   ├── views.py
│   ├── home.pt
│   └── static/
│       └── app.css
├── setup.py
└── development.ini
```

---

## 3. Menambahkan Static View

Di `__init__.py`:

```python
config.add_static_view(name='static', path='tutorial:static')
```

Artinya:

* Semua request ke `/static/*`
* Akan dilayani oleh file di folder `tutorial/static`

Misalnya:

```
/static/app.css → tutorial/static/app.css
```

---

## 4. Menggunakan CSS di Template

Di `home.pt` tambahkan:

```xml
<link rel="stylesheet"
      href="${request.static_url('tutorial:static/app.css') }"/>
```

Penjelasan:

* Tidak hardcode URL seperti `/static/app.css`
* Menggunakan helper agar URL bisa otomatis berubah bila konfigurasi berubah
* Menghindari masalah saat aplikasi dipindahkan ke subfolder atau environment berbeda

---

## 5. Membuat File CSS

File: `tutorial/static/app.css`

```css
body {
    margin: 2em;
    font-family: sans-serif;
}
```

---

## 6. Functional Test Untuk Static File

Tambahkan:

```python
def test_css(self):
    res = self.testapp.get('/static/app.css', status=200)
    self.assertIn(b'body', res.body)
```

Test memastikan:

* Static file benar-benar dilayani oleh Pyramid
* Kontennya sesuai

---

## 7. Menjalankan Tests

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
.....
5 passed in 0.50 seconds
```

---

## 8. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Lalu buka:

```
http://localhost:6543/
```

Anda akan melihat font berubah (efek CSS).

---

# **9. Analisis**

### 1. Static Asset Served di /static/

Pyramid mengubah direktori:

```
tutorial/static
```

menjadi URL publik:

```
/static
```

Tidak membutuhkan view khusus — Pyramid langsung melayani file secara otomatis.

---

### 2. Menggunakan request.static_url()

Mengapa harus memakai:

```xml
${request.static_url('tutorial:static/app.css')}
```

daripada menulis:

```xml
/static/app.css
```

Karena:

#### URL bisa berubah suatu hari (misal deploy di subfolder)

```
/mysite/static/app.css
```

#### Bisa dipindahkan di filesystem—tetap berfungsi

#### Mendukung versi / cache busting otomatis (untuk production)

#### Lebih aman dan fleksibel

---

### 3. Static Path Diatur dengan Package Notation

Path:

```
tutorial:static
```

adalah *asset specification*, yaitu cara Pyramid menunjuk folder di package Python.

---

# **10. Extra Credit – Jawaban**

### Apa perbedaan antara `request.static_url` dan `request.static_path`?

---

## **1. request.static_url**

Menghasilkan **URL publik** yang bisa digunakan browser.

Contoh:

```python
request.static_url('tutorial:static/app.css')
```

Hasil:

```
http://localhost:6543/static/app.css
```

Dipakai untuk:

* HTML (CSS, JS, IMG)
* Templates
* Link publik
* Browser resources

---

## **2. request.static_path**

Menghasilkan **path filesystem**, bukan URL.

Contoh:

```python
request.static_path('tutorial:static/app.css')
```

Hasil:

```
/home/user/project/tutorial/static/app.css
```

Dipakai ketika:

* Anda perlu membaca file secara langsung dari disk
* Pengolahan backend
* Logging atau debugging

**Bukan** untuk HTML, karena browser tidak bisa membaca path filesystem.

---

### Ringkasan Perbedaan

| Fungsi                | Output                        | Untuk                                 |
| --------------------- | ----------------------------- | ------------------------------------- |
| `request.static_url`  | URL yang dapat dibuka browser | HTML, template, link publik           |
| `request.static_path` | path file di disk             | backend logic, akses file dari server |

---

# **11. Conclusion**

Pada langkah ini Anda mempelajari:

* Cara menambahkan static view di Pyramid
* Cara menyajikan CSS/JS/images dari direktori package
* Cara menghasilkan URL static yang aman dan fleksibel
* Cara mengetes delivery static assets
* Perbedaan `static_url` dan `static_path`

Static assets adalah bagian penting aplikasi web modern, dan Pyramid menyediakan cara rapi untuk mengelolanya.

---
<img width="957" height="1078" alt="praktikumpaw13" src="https://github.com/user-attachments/assets/ee43c296-9f41-4e3b-a445-59c4cf0cc8d7" />
