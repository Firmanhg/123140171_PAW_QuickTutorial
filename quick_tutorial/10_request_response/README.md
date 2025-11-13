# **README – Handling Web Requests and Responses (Step 10)**

## 1. Overview

Pada langkah ini, kita fokus pada inti web development: **request** dan **response**.
Pyramid menggunakan library WebOb sebagai fondasi untuk kedua hal ini, sehingga setiap request dan response:

* konsisten,
* powerful,
* mudah dimanipulasi,
* kompatibel dengan seluruh ecosystem WSGI.

Pada step ini kita belajar:

* mengambil data dari request (query string)
* membaca info URL dari request
* mengembalikan response khusus (plain text)
* melakukan HTTP redirect

---

## 2. Struktur Proyek

```
request_response/
├── setup.py
├── development.ini
└── tutorial/
    ├── __init__.py
    ├── views.py
    └── tests.py
```

---

## 3. Routing di **init**.py

```python
config.add_route('home', '/')
config.add_route('plain', '/plain')
```

Kita hanya menggunakan dua route:

* `/` → redirect ke `/plain`
* `/plain` → menampilkan konten berdasarkan query string

---

## 4. View Class Dengan Request dan Response

File: `views.py`

```python
from pyramid.httpexceptions import HTTPFound
from pyramid.response import Response
from pyramid.view import view_config

class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return HTTPFound(location='/plain')

    @view_config(route_name='plain')
    def plain(self):
        name = self.request.params.get('name', 'No Name Provided')

        body = 'URL %s with name: %s' % (self.request.url, name)
        return Response(
            content_type='text/plain',
            body=body
        )
```

Penjelasan penting:

### 1. Redirect

`home()` mengembalikan objek:

```python
HTTPFound(location='/plain')
```

Ini memberi respons HTTP 302.

### 2. Mengambil query param

```python
name = self.request.params.get('name', 'No Name Provided')
```

Jika `/plain?name=alice`, maka:

* `name` = `"alice"`

Jika tidak ada param, fallback default digunakan.

### 3. Mengakses URL penuh

```python
self.request.url
```

Contoh output:

```
URL http://localhost:6543/plain?name=alice with name: alice
```

### 4. Response plain text

Kita membuat respons manual:

```python
Response(content_type='text/plain', body=body)
```

Ini memproduksi output bukan HTML.

---

## 5. Unit Tests

Unit test memeriksa:

* redirect 302
* response body tanpa query param
* response body dengan query param

Contoh:

```python
response = inst.home()
self.assertEqual(response.status, '302 Found')
```

---

## 6. Functional Tests

Functional test memeriksa aplikasi sebenarnya melalui WebTest:

```python
res = self.testapp.get('/plain?name=Jane%20Doe')
self.assertIn(b'Jane Doe', res.body)
```

Karena body berupa bytes, kita cek menggunakan `b'...'`.

---

## 7. Menjalankan Tests

```
$VENV/bin/pytest tutorial/tests.py -q
```

Output:

```
.....
5 passed in 0.30 seconds
```

---

## 8. Menjalankan Aplikasi

```
$VENV/bin/pserve development.ini --reload
```

Lihat:

* [http://localhost:6543](http://localhost:6543) → redirect
* [http://localhost:6543/plain?name=alice](http://localhost:6543/plain?name=alice) → menampilkan info

---

# **9. Analisis**

Pada langkah ini, Anda mempraktikkan elemen fundamental aplikasi web:

---

### 1. Redirect Menggunakan HTTPFound

Pyramid dapat melakukan redirect melalui:

* mengembalikan objek redirect, atau
* melempar exception redirect

Keduanya valid.

---

### 2. Mengambil Data dari Request

Menggunakan WebOb:

```python
self.request.params.get('name')
```

Params mencakup gabungan:

* GET params
* POST params (jika ada)

---

### 3. Response Objek Bisa Dikonfigurasi

Kita mengatur:

* header
* body
* content type

secara manual ketika tidak menggunakan template.

---

### 4. Testing request/response sangat mudah

Karena semua berbasis WebOb (yang sudah matang), maka:

* DummyRequest bisa dipakai untuk unit test
* WebTest bisa dipakai untuk functional test
* Response object memudahkan pemeriksaan headers, body, status

---

# **10. Extra Credit – Jawaban Lengkap**

## **Apakah kita bisa *raise* `HTTPFound(location='/plain')` daripada mengembalikannya?**

**YA, bisa.**

Contoh:

```python
raise HTTPFound(location='/plain')
```

Pyramid mengizinkan dua gaya konfigurasi:

---

### Mengembalikan objek:

```python
return HTTPFound(location='/plain')
```

### Melempar exception:

```python
raise HTTPFound(location='/plain')
```

---

## Apa perbedaannya?

### 1. Secara fungsional: **tidak ada perbedaan dalam hasil akhir**

Keduanya akan menghasilkan:

* status 302
* header Location: /plain
* browser akan redirect

---

### 2. Perbedaan di alur eksekusi kode

| Return                                             | Raise                                                     |
| -------------------------------------------------- | --------------------------------------------------------- |
| Mengembalikan nilai dari fungsi view               | Menghentikan eksekusi fungsi secara langsung              |
| Cocok jika ingin menjalankan logika sebelum return | Cocok jika redirect harus terjadi segera                  |
| Lebih eksplisit sebagai “ini response saya”        | Lebih mirip error handling tetapi untuk aksi dan redirect |

---

### 3. Raise lebih berguna dalam kasus tertentu:

* validasi form gagal → redirect atau rerender
* API yang punya banyak kondisi branching
* ingin keluar dari logic nested tanpa mengeksekusi kode berikutnya

---

### 4. Return lebih natural untuk view sederhana

Tutorial ini menggunakan `return` karena mudah dipahami.

---

# **11. Conclusion**

Pada langkah ini Anda mempelajari:

* konsep request dan response berbasis WebOb
* mengambil query parameters
* mengakses URL lengkap dari request
* membuat response plain text custom
* melakukan redirect
* menguji request/response menggunakan unit test dan functional test

Ini adalah fondasi penting sebelum masuk ke topik routing lanjutan, forms handling, security, dan database integration.

---
<img width="957" height="1078" alt="praktikumpaw10" src="https://github.com/user-attachments/assets/a78d599d-ea23-4199-9639-df332e659bc0" />
<img width="957" height="1078" alt="praktikumpaw10s" src="https://github.com/user-attachments/assets/a71449a1-3d24-4796-b9ea-824e98a168e6" />

