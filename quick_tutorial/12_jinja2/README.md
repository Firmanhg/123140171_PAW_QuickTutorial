B# **README – Templating With Jinja2 (Step 12)**

##  1. Overview

Pada langkah ini, kita membuktikan bahwa Pyramid **tidak mengikat Anda pada satu template engine**.
Setelah menggunakan Chameleon di langkah sebelumnya, sekarang kita menambahkan dukungan untuk:

### **Jinja2**, template engine populer yang digunakan oleh Flask dan Django.

Tujuan step ini:

* Menunjukkan bagaimana Pyramid mendukung berbagai renderer
* Menginstal add-on Pyramid (pyramid_jinja2)
* Menggunakan renderer Jinja2 di view
* Membuat template `.jinja2` sederhana

---

## 2. Struktur Proyek

```
jinja2/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    ├── home.jinja2
    └── tests.py
```

---

## 3. Menambahkan `pyramid_jinja2` ke setup.py

```python
requires = [
    'pyramid',
    'pyramid_chameleon',
    'pyramid_jinja2',
    'waitress',
]
```

Install perubahan:

```
$VENV/bin/pip install -e .
```

---

## 4. Mengaktifkan Jinja2 pada **init**.py

```python
config.include('pyramid_jinja2')
```

Ini membuat Pyramid mengenali file `.jinja2` sebagai template renderer.

---

## 5. View Menggunakan Renderer Jinja2

```python
@view_defaults(renderer='home.jinja2')
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

Satu-satunya perubahan dari langkah sebelumnya:

* `renderer='home.pt'` → `renderer='home.jinja2'`

View tetap mengembalikan dictionary — template yang memutuskan cara menampilkannya.

---

## 6. Template Jinja2 (`home.jinja2`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: {{ name }}</title>
</head>
<body>
<h1>Hi {{ name }}</h1>
</body>
</html>
```

Perbedaan utama dibanding Chameleon:

* Chameleon: `${name}`
* Jinja2: `{{ name }}`

---

## 7. Test Tetap Berjalan Tanpa Perubahan

Menjalankan test:

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
....
4 passed in 0.40 seconds
```

Tidak ada perubahan pada test karena:

* struktur view sama
* data contract sama
* functional test tetap memeriksa output HTML

---

## 8. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Akses:

```
http://localhost:6543/
```

---

# **9. Analisis**

Langkah ini menunjukkan bahwa:

---

### 1. Pyramid Memiliki Renderer yang Dapat Diganti

Pyramid tidak mengharuskan template engine tertentu. Anda cukup:

* Install package Python (misal: `pyramid_jinja2`)
* Tambahkan ke configurator (`config.include`)
* Gunakan renderer dengan file extension yang sesuai

Ini membuat Pyramid sangat fleksibel.

---

### 2. Jinja2 Menjadi Renderer Baru

Ketika Pyramid menjalankan:

```python
config.include('pyramid_jinja2')
```

Add-on tersebut:

* mendaftarkan file `.jinja2`
* menambahkan environment Jinja2
* memungkinkan penggunaan renderer `'home.jinja2'`

---

### 3. View Class Tidak Perlu Diubah

Anda hanya mengubah renderer, bukan logika view:

* data contract tetap dictionary
* template yang memutuskan tampilan

---

### 4. Chameleon dan Jinja2 Sama-Sama Mudah Dipakai

Secara sintaks dasar, keduanya sangat mirip, sehingga mudah berpindah template engine.

---

# **10. Extra Credit – Jawaban Lengkap**

## 1. Cara lain mengasosiasikan pyramid_jinja2 tanpa menginstalnya manual?

Selain install manual seperti:

```
pip install pyramid_jinja2
```

Anda dapat:

### **Menambahkannya ke install_requires di setup.py**

Artinya begitu Anda menjalankan:

```
pip install -e .
```

dependency akan otomatis ikut di-install.

Ini adalah cara **yang disarankan** untuk project nyata.

---

### **Menggunakan extras_require**

Contoh:

```python
extras_require={
    'jinja2': ['pyramid_jinja2']
}
```

Lalu install:

```
pip install -e ".[jinja2]"
```

Ini memungkinkan pemilihan template engine secara modular.

---

### **Menggunakan project scaffolding**

Jika Anda membuat project Pyramid dengan cookiecutter default:

```
pcreate -s starter myproj
```

Anda bisa memilih template language, termasuk Jinja2, dan scaffolding akan langsung mengonfigurasi semuanya.

---

## 2. Selain `config.include`, cara lain mengaktifkan pyramid_jinja2?

Ada dua cara lain:

---

### A. Menggunakan `pyramid.includes` di `.ini` file

Di `development.ini`:

```ini
pyramid.includes =
    pyramid_jinja2
```

Keuntungan:

* Tidak perlu menyentuh kode Python
* Bisa diaktifkan atau dimatikan per-environment
* Konfigurasi tetap bersih

---

### B. Menggunakan ZCML (jarang digunakan)

Pyramid mendukung ZCML configurator (ala Plone/zope), namun tidak umum untuk project modern.

---

### C. Menggunakan scan() jika add-on mendukung autodiscovery

Beberapa add-on bisa ditemukan via scanning, namun untuk pyramid_jinja2 tetap disarankan `config.include`.

---

# **11. Conclusion**

Pada langkah ini Anda mempelajari:

* bagaimana Pyramid mendukung berbagai template engine
* cara menambahkan renderer baru melalui add-on (pyramid_jinja2)
* perbedaan dasar sintaks Jinja2 vs Chameleon
* fleksibilitas konfigurasi via setup.py dan .ini file
* bahwa view code dapat tetap sama meskipun template engine berubah

Ini menegaskan kekuatan Pyramid sebagai framework yang:

* fleksibel,
* modular,
* tidak mengunci Anda pada satu cara,
* siap untuk berbagai kebutuhan aplikasi.

---
<img width="960" height="1078" alt="praktikumpaw12" src="https://github.com/user-attachments/assets/6b65a135-5505-4910-8583-5c8c9d2360da" />

