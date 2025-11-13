# **README – HTML Generation With Templating (Step 08)**

## 1. Overview

Pada langkah ini, kita berpindah dari membuat HTML langsung di kode Python menjadi menggunakan **template engine**. Pyramid tidak memaksakan satu jenis template, namun tutorial ini menggunakan **pyramid_chameleon**, add-on resmi untuk templating.

Tujuan utama:

* Menggunakan template `.pt` untuk menghasilkan HTML
* Membuat view hanya mengembalikan *data*, bukan HTML
* Menghubungkan templates menggunakan fitur **renderer**
* Mendukung development yang lebih cepat dengan template reloading

Langkah ini menunjukkan cara membuat aplikasi Pyramid lebih terstruktur, lebih bersih, dan lebih scalable.

---

## 2. Struktur Proyek

```
templating/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    ├── home.pt
    └── tests.py
```

---

## 3. Menambahkan `pyramid_chameleon` ke setup.py

```python
requires = [
    'pyramid',
    'pyramid_chameleon',
    'waitress',
]
```

Kemudian install:

```
$VENV/bin/pip install -e .
```

---

## 4. Mengaktifkan Template Renderer di **init**.py

```python
config.include('pyramid_chameleon')
```

Ini wajib agar Pyramid mengenali file `.pt` dan memo rendering.

---

## 5. Views Tanpa HTML di views.py

View sebelumnya mengembalikan `Response(...)` berisi HTML.
Sekarang view cukup mengembalikan **dictionary**, yang akan diberikan ke template:

```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}

@view_config(route_name='hello', renderer='home.pt')
def hello(request):
    return {'name': 'Hello View'}
```

Perubahan penting:

* Tidak ada HTML di view
* Rendering dilakukan oleh template engine
* Reusable template untuk banyak view

---

## 6. Template home.pt

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
</head>
<body>
<h1>Hi ${name}</h1>
</body>
</html>
```

Anda dapat memanggil variabel Python dengan sintaks `${name}`.

---

## 7. Template Auto Reloading di development.ini

Tambahkan:

```
pyramid.reload_templates = true
```

Agar perubahan template langsung terlihat tanpa restart server.

---

## 8. Pengujian Unit Menguji Data, Bukan HTML

```python
response = home(request)
self.assertEqual('Home View', response['name'])
```

Karena view hanya mengembalikan dictionary, unit test cukup menguji data, bukan HTML.

---

## 9. Functional Test Menguji Output HTML

```python
res = self.testapp.get('/', status=200)
self.assertIn(b'<h1>Hi Home View', res.body)
```

Di functional test, kita memastikan HTML sudah benar setelah template diproses.

---

## 10. Menjalankan Tes

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
....
4 passed in 0.46 seconds
```

---

## 11. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Akses:

* [http://localhost:6543/](http://localhost:6543/)
* [http://localhost:6543/howdy](http://localhost:6543/howdy)

---

# **12. Analisis**

### 1. View kini fokus pada logika, bukan HTML

View sekarang hanya mengembalikan data:

```python
return {'name': 'Home View'}
```

Ini membuat view:

* lebih mudah diuji
* lebih bersih
* lebih terpisah dari tampilan (separation of concerns)

---

### 2. Renderer menentukan template mana yang akan digunakan

```python
@view_config(renderer='home.pt')
```

Renderer bertanggung jawab:

* memilih template
* memproses data dictionary
* menghasilkan HTML final

---

### 3. Template reuse

Satu template dapat digunakan untuk banyak view, misalnya:

```python
renderer='home.pt'
```

digunakan oleh:

* home view
* hello view

Hanya nilai data yang diganti, HTML tetap sama.

---

### 4. Pengujian menjadi lebih jelas

Unit test tidak perlu tahu HTML—cukup periksa data yang dikembalikan view.
Functional test menguji HTML akhir.

Ini memisahkan:

* **logic test**
* **template test**

---

# **13. Conclusion**

Pada langkah ini, Anda mempelajari:

* cara mengaktifkan `pyramid_chameleon`
* cara memisahkan logic view dengan tampilan HTML
* penggunaan renderer untuk template `.pt`
* cara testing view berbasis data
* cara functional testing memverifikasi HTML akhir

Langkah ini membuat struktur aplikasi Pyramid lebih modular, profesional, dan mudah dikembangkan.

---
<img width="955" height="1078" alt="praktikumpaw8" src="https://github.com/user-attachments/assets/b87d13e3-fb9a-4ecc-a696-2f77103f4589" />
<img width="960" height="1078" alt="praktikumpaw8s" src="https://github.com/user-attachments/assets/06ed63ae-9768-473b-bd9d-c261b7ea402c" />

