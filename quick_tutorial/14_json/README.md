# **README – AJAX Development With JSON Renderers (Step 14)**

## 1. Overview

Pada langkah ini kita memperluas aplikasi Pyramid agar mampu menghasilkan **JSON responses**, yang penting untuk:

* AJAX / Fetch API / XMLHttpRequest
* Single Page Applications (SPA)
* Dynamic UI updates
* REST-like endpoints

Pyramid menyediakan **JSON renderer** bawaan yang dapat digunakan hanya dengan konfigurasi sederhana—tanpa harus membuat `Response` atau memanggil `json.dumps()` secara manual.

Langkah ini menunjukkan:

* bagaimana menambahkan endpoint JSON
* bagaimana satu view dapat melayani **dua format** (HTML *dan* JSON)
* bagaimana Pyramid memilih renderer berdasarkan routing

---

## 2. Struktur Proyek

```
json/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    ├── home.pt
    └── tests.py
```

---

## 3. Menambahkan Route JSON

Di `__init__.py`:

```python
config.add_route('hello_json', '/howdy.json')
```

Ini membuat URL baru untuk format JSON.

---

## 4. Satu View, Dua Output (HTML dan JSON)

File: `views.py`

```python
@view_config(route_name='hello')
@view_config(route_name='hello_json', renderer='json')
def hello(self):
    return {'name': 'Hello View'}
```

Penjelasan:

### HTML route → `/howdy`

Menggunakan renderer default dari `@view_defaults(renderer='home.pt')`

### JSON route → `/howdy.json`

Menggunakan renderer JSON karena override:

```python
@view_config(route_name='hello_json', renderer='json')
```

Dengan strategi ini:

* tidak perlu membuat dua fungsi view
* tidak perlu duplicate logic
* Pyramid otomatis memilih renderer berdasarkan route

---

## 5. Apa itu JSON Renderer?

Renderer JSON Pyramid:

* mengubah dictionary → JSON string
* mengatur header `Content-Type: application/json`
* memastikan encoding UTF-8
* otomatis dipilih ketika renderer='json'

Tidak perlu membuat `Response()` manual.

---

## 6. Functional Test untuk JSON Endpoint

```python
res = self.testapp.get('/howdy.json', status=200)
self.assertIn(b'{"name": "Hello View"}', res.body)
self.assertEqual(res.content_type, 'application/json')
```

Test memverifikasi bahwa:

* response JSON sesuai
* content-type benar

---

## 7. Menjalankan Tests

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
.....
5 passed in 0.47 seconds
```

---

## 8. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Kunjungi:

```
http://localhost:6543/howdy.json
```

Anda akan melihat:

```json
{"name": "Hello View"}
```

---

# **9. Analisis**

### 1. Pyramid Merender Berdasarkan Renderer

View mengembalikan **dictionary**, tetapi output final bergantung pada renderer:

* HTML → Chameleon/Jinja2 template
* JSON → JSON renderer
* Anda bahkan bisa membuat renderer custom sendiri

Karena view tidak perlu peduli tentang format output, testing dan maintenance menjadi lebih mudah.

---

## 2. Satu View, Banyak Output

Pyramid menerapkan **multiview** secara alami:

* banyak `@view_config` untuk satu fungsi/method
* behavior ditentukan oleh route, predicate, atau renderer

Ini memungkinkan fleksibilitas sangat tinggi.

---

### 3. JSON Renderer Memiliki Batasan Python JSON

Karena menggunakan encoder bawaan Python:

* tidak bisa encode datetime, decimal, bytes, dsb
* perlu custom renderer bila Anda ingin behavior tambahan

Pyramid memungkinkan:

```python
config.add_renderer('json', my_custom_encoder)
```

---

### 4. Alternatif: Match Berdasarkan Accept Header

Jika ingin *AJAX-based behavior* tanpa route terpisah, gunakan:

```python
@view_config(route_name='hello', accept='application/json', renderer='json')
```

Browser modern mengirim:

```
Accept: application/json
```

Ini memungkinkan view yang sama menghasilkan HTML atau JSON tergantung siapa yang meminta.

---

# **10. Extra Notes: How JSON Selection Works**

Saat request masuk pada `/howdy.json`, Pyramid:

1. Mencari route `'hello_json'`
2. Mendapatkan view config dengan renderer JSON
3. Menjalankan view
4. Mengambil return dict
5. Merender sebagai JSON
6. Mengirim header `Content-Type: application/json`

Tidak perlu logic tambahan dalam view.

---

# **11. Conclusion**

Pada langkah ini Anda mempelajari:

* bagaimana menambahkan endpoint JSON
* bagaimana view class dapat melayani HTML dan JSON
* bagaimana JSON renderer Pyramid bekerja otomatis
* bagaimana testing JSON endpoint dilakukan
* bagaimana memanfaatkan multiview dan route-specific rendering

Pyramid sangat fleksibel dalam pemisahan data dan presentasi, membuatnya ideal untuk aplikasi web modern yang memerlukan UI dinamis.

---
<img width="956" height="1078" alt="praktikumpaw14" src="https://github.com/user-attachments/assets/492397bd-568b-4430-8a06-f1204378a047" />
