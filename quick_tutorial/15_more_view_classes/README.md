# **README – More With View Classes (Step 15)**

## 1. Overview

Pada langkah ini, kita memanfaatkan **fitur lanjutan view classes** di Pyramid.
View classes memungkinkan:

* grouping beberapa view dalam satu class
* berbagi state, logic, dan default configuration
* dispatching satu route ke berbagai view berdasarkan **predicate**
* penggunaan atribut class di template (`view.some_value`)
* pemanggilan metode sebagai properti (dengan `@property`)

Contoh yang diberikan: URL `/howdy/{first}/{last}` memiliki **4 kemungkinan view**, bergantung pada:

* apakah request adalah GET atau POST
* apakah ada parameter POST tertentu
* apakah view override konfigurasi default

Ini menunjukkan fleksibilitas Pyramid dalam menangani operasi REST-like pada satu URL.

---

## 2. Struktur Proyek

```
more_view_classes/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    ├── home.pt
    ├── hello.pt
    ├── edit.pt
    └── delete.pt
```

---

## 3. Routing Dengan Replacement Pattern

Di `__init__.py`:

```python
config.add_route('hello', '/howdy/{first}/{last}')
```

URL seperti:

```
/howdy/jane/doe
```

menghasilkan:

```python
request.matchdict = {
    'first': 'jane',
    'last': 'doe'
}
```

---

## 4. View Class Dengan Banyak View

File: `views.py`

```python
@view_defaults(route_name='hello')
class TutorialViews:
    def __init__(self, request):
        self.request = request
        self.view_name = 'TutorialViews'

    @property
    def full_name(self):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return first + ' ' + last
```

View method:

### 1. GET → hello view

```python
@view_config(renderer='hello.pt')
def hello(self):
    return {'page_title': 'Hello View'}
```

### 2. POST → edit view

```python
@view_config(request_method='POST', renderer='edit.pt')
def edit(self):
    new_name = self.request.params['new_name']
    return {'page_title': 'Edit View', 'new_name': new_name}
```

### 3. POST dengan parameter tertentu → delete view

```python
@view_config(request_method='POST', request_param='form.delete', renderer='delete.pt')
def delete(self):
    print('Deleted')
    return {'page_title': 'Delete View'}
```

### 4. Route berbeda → home view

```python
@view_config(route_name='home', renderer='home.pt')
def home(self):
    return {'page_title': 'Home View'}
```

---

## 5. Template Menggunakan Atribut Class

Template dapat mengakses `view.view_name` atau `view.full_name`:

```xml
<h1>${view.view_name} - ${page_title}</h1>
<p>Welcome, ${view.full_name}</p>
```

Karena `full_name` adalah @property, ia dipanggil seperti atribut biasa.

---

## 6. Tests

Unit test:

```python
inst = TutorialViews(request)
response = inst.home()
self.assertEqual('Home View', response['page_title'])
```

Functional test:

```python
res = self.testapp.get('/', status=200)
self.assertIn(b'TutorialViews - Home View', res.body)
```

---

## 7. Menjalankan Tests

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
..
2 passed in 0.40 seconds
```

---

## 8. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Tes di browser:

* `/howdy/jane/doe`
* klik "Save" untuk memicu edit view
* klik "Delete" untuk memicu delete view

Console akan menampilkan log `"Deleted"`.

---

# **9. Analisis**

Langkah ini menunjukkan kemampuan canggih Pyramid:

---

### 1. Multi-View Dispatch dengan Predicates

Satu URL:

```
/howdy/jane/doe
```

dapat menghasilkan:

| Kondisi                   | View   |
| ------------------------- | ------ |
| GET                       | hello  |
| POST tanpa parameter      | edit   |
| POST dengan `form.delete` | delete |

Predicates yang digunakan:

* `request_method='POST'`
* `request_param='form.delete'`

Pyramid memilih view **paling spesifik** yang cocok.

---

### 2. class-level @view_defaults

Deklarasi:

```python
@view_defaults(route_name='hello')
```

Berlaku untuk SEMUA view dalam class, kecuali yang override.

---

### 3. Template Mengakses Properti Class

`@property` memungkinkan:

```xml
${view.full_name}
```

bukan:

```
${view.full_name()}
```

Ini membuat template lebih bersih.

---

### 4. URL generation lebih aman dengan route_url()

Dari:

```
/howdy/jane/doe
```

Menjadi:

```xml
${request.route_url('hello', first='jane', last='doe')}
```

Keuntungannya:

* tidak hardcoded
* lebih aman jika route berubah
* refactoring-friendly

---

# **10. Extra Credit – Jawaban Lengkap**

---

## 1. Mengapa template dapat memakai `${view.full_name}` tanpa `()`?

Karena:

```python
@property
def full_name(self):
```

`@property` membuat method diperlakukan sebagai **attribute**, bukan fungsi.
Jadi:

* `view.full_name` → memanggil method & mengembalikan hasil
* `view.full_name()` → ERROR (karena properti tidak callable)

Chameleon (template engine) mengikuti aturan normal Python, sehingga `@property` bekerja otomatis.

---

## 2. Mengapa edit view tidak menangkap POST milik delete?

Karena delete view lebih spesifik:

```python
@view_config(request_method='POST', request_param='form.delete')
```

Predicate Pyramid bekerja berdasarkan **specificity order**.

Hierarki kespesifikan:

1. `request_method + request_param`
2. `request_method` saja
3. tidak ada predicate

Jadi:

* Jika POST dengan parameter `form.delete` → delete view
* Jika POST tanpa `form.delete` → edit view

---

## 3. Apakah Pyramid menyediakan caching untuk @property?

Ya.

Gunakan **`@reify`**, dekorator dari Pyramid:

```python
from pyramid.decorator import reify

@reify
def full_name(self):
    return self.request.matchdict['first'] + ' ' + self.request.matchdict['last']
```

Perilaku:

* dihitung sekali
* disimpan
* akses berikutnya mengambil nilai cached, bukan menghitung ulang
* hanya berlaku per-instance view class

Ini ideal untuk komputasi yang:

* mahal
* dipanggil berkali-kali
* tidak berubah

Reify sangat berguna dalam view class Pyramid.

---

## 4. Bisakah satu view terkait dengan lebih dari satu route?

Ya.

Anda dapat menumpuk beberapa `@view_config` pada method yang sama:

```python
@view_config(route_name='route1')
@view_config(route_name='route2')
def my_view(self):
    return {...}
```

Atau kumpulkan di class-level view_defaults.

---

## 5. Apa perbedaan `request.route_path` dan `request.route_url`?

| Fungsi         | Output                                           | Penggunaan                        |
| -------------- | ------------------------------------------------ | --------------------------------- |
| **route_path** | path relatif server                              | HTML internal, link relatif       |
| **route_url**  | URL absolut lengkap (dengan skema, domain, port) | email, redirect, absolute linking |

Contoh:

```python
request.route_path('hello', first='jane', last='doe')
```

Hasil:

```
/howdy/jane/doe
```

Sedangkan:

```python
request.route_url('hello', first='jane', last='doe')
```

Hasil:

```
http://localhost:6543/howdy/jane/doe
```

---

# **11. Conclusion**

Pada step ini Anda mempelajari konsep lanjutan dari view classes:

* grouping view dengan route defaults
* dispatch berdasarkan predicate (GET/POST/params)
* penggunaan @property dan @reify
* pemilihan view otomatis berdasarkan kondisi request
* template mengakses atribut view class
* teknik URL generation yang fleksibel

View classes adalah fitur kuat Pyramid untuk membangun aplikasi besar, modular, dan REST-style.

---

Jika mau, saya siap buatkan README untuk **Step 16** atau membuat satu **README komprehensif untuk Step 1–15** dalam satu dokumen.


<img width="956" height="1078" alt="praktikumpaw15" src="https://github.com/user-attachments/assets/0ab781bf-5c5b-4211-abc7-4359632c4d2a" />
<img width="957" height="1078" alt="praktikumpaw15s" src="https://github.com/user-attachments/assets/f044b71a-241e-4067-892f-7cff7132b0e3" />
<img width="955" height="1078" alt="praktikumpaw15ss" src="https://github.com/user-attachments/assets/8931b237-b61f-478c-9442-3efaee72a243" />
