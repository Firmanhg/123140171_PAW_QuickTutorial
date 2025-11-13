# **README – Dispatching URLs to Views With Routing (Step 11)**

## 1. Overview

Pada langkah ini, kita mempelajari fitur routing Pyramid yang lebih canggih, yaitu:

* **Menangkap bagian dari URL** menggunakan *replacement pattern* `{variable}`
* **Mengambil nilai hasil tangkapan** dari `request.matchdict`
* **Menggunakannya dalam view & template**

Routing adalah bagian penting dalam web framework karena menentukan **bagaimana URL dipetakan ke kode Python**.
Dengan langkah ini, aplikasi Anda dapat menangani URL dinamis seperti:

```
/howdy/Jane/Doe
```

dan mengambil:

* `first = Jane`
* `last = Doe`

---

## 2. Struktur Proyek

```
routing/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    ├── home.pt
    └── tests.py
```

---

## 3. Routing Dengan Replacement Pattern

Dalam `__init__.py`:

```python
config.add_route('home', '/howdy/{first}/{last}')
```

URL pattern:

```
/howdy/{first}/{last}
```

membuat dua variable dynamic:

* `{first}`
* `{last}`

Ketika user mengakses:

```
/howdy/amy/smith
```

Maka Pyramid menghasilkan:

```python
request.matchdict = {
    'first': 'amy',
    'last': 'smith'
}
```

---

## 4. View Class Mengakses matchdict

File: `views.py`

```python
@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return {
            'name': 'Home View',
            'first': first,
            'last': last
        }
```

Kunci penting:

### `self.request.matchdict`

Berisi data yang ditangkap dari URL berdasarkan pola yang Anda definisikan.

---

## 5. Template home.pt

```html
<h1>${name}</h1>
<p>First: ${first}, Last: ${last}</p>
```

Data dari view langsung tersedia di template.

---

## 6. Unit Test

```python
request = testing.DummyRequest()
request.matchdict['first'] = 'First'
request.matchdict['last'] = 'Last'
inst = TutorialViews(request)
response = inst.home()
self.assertEqual(response['first'], 'First')
self.assertEqual(response['last'], 'Last')
```

Anda menyimulasikan matchdict yang biasanya datang dari routing.

---

## 7. Functional Test

```python
res = self.testapp.get('/howdy/Jane/Doe', status=200)
self.assertIn(b'Jane', res.body)
self.assertIn(b'Doe', res.body)
```

Functional test memastikan sistem full-stack (routing → view → template) bekerja.

---

## 8. Menjalankan Tests

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
..
2 passed in 0.39 seconds
```

---

## 9. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Coba:

```
http://localhost:6543/howdy/amy/smith
```

Anda akan melihat informasi sesuai URL.

---

# **10. Analisis**

Langkah ini memperkenalkan konsep besar:

---

### 1. Replacement Patterns (`{variable}`)

Pyramid memetakan URL ke variabel Python menggunakan tanda `{}`.

---

### 2. matchdict

Nilai hasil match tersimpan pada:

```python
request.matchdict
```

Ini adalah dict yang hanya berisi nilai dari URL—bukan query param.

---

### 3. Pemisahan yang jelas antara routing & view

Alasan Pyramid memisahkan:

```python
add_route(...)
```

dan

```python
@view_config(route_name='...')
```

adalah agar Anda dapat:

* mengontrol urutan route
* menambah banyak view untuk satu route
* menghindari ambigu matching
* membuat aplikasi besar lebih stabil

---

### 4. Routing Dynamic Sangat Powerful

Anda bisa membuat pola seperti:

* `/users/{id}`
* `/blog/{year}/{month}/{slug}`
* `/files/{filepath:.+}` (regex route)

Pyramid menyediakan routing fleksibel dan kuat.

---

# **11. Extra Credit – Jawaban**

### *Apa yang terjadi jika Anda membuka:*

```
http://localhost:6543/howdy
```

### Jawabannya:

Anda akan mendapatkan **404 Not Found**.

Mengapa?

Karena route yang didaftarkan adalah:

```
/howdy/{first}/{last}
```

yang berarti:

* dua segmen **WAJIB** ada setelah `/howdy`
* tidak boleh kurang, tidak boleh lebih

URL `/howdy` **tidak memenuhi pola tersebut**, sehingga tidak ada route yang cocok → Pyramid mengembalikan 404.

---

### Apakah ini sesuai harapan?

Ya, ini benar dan merupakan perilaku yang diharapkan.

Routing Pyramid bersifat **eksplisit dan ketat**, sehingga:

* URL harus cocok tepat dengan pola yang didefinisikan
* tidak ada fallback otomatis
* ini membantu menghindari kecocokan yang ambigu

Jika Anda ingin `/howdy` tetap valid, Anda harus membuat route khusus:

```python
config.add_route('howdy_root', '/howdy')
```

atau membuat pola opsional (menggunakan route predicate atau regex).

---

# **12. Conclusion**

Pada langkah ini Anda mempelajari:

* bagaimana membuat URL dynamic dengan replacement pattern
* bagaimana mengambil nilai path parameter via `request.matchdict`
* bagaimana menghubungkan routing → view → template
* bagaimana mengetes matchdict lewat unit dan functional tests
* memahami bagaimana Pyramid menangani URL yang tidak cocok

Ini adalah dasar penting untuk membangun aplikasi web kompleks yang membutuhkan routing RESTful atau struktur URL yang rapi.

---
<img width="958" height="1076" alt="praktikumpaw11" src="https://github.com/user-attachments/assets/298ad742-dc36-4366-ab17-d1c356be8dd7" />
<img width="960" height="1078" alt="praktikumpaw11s" src="https://github.com/user-attachments/assets/6509daee-bed2-41e7-9f98-cdd505130f8f" />

