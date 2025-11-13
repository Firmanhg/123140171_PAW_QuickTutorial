Berikut **README.md** lengkap yang berisi **ringkasan materi, analisis, dan jawaban extra credit** berdasarkan teks yang kamu berikan.

---

# **README – Pyramid Single-File Web Application**

## **1. Overview**

Tutorial ini menunjukkan cara membuat aplikasi web berbasis **Pyramid** hanya dalam **satu file Python**. Pendekatan ini sangat cocok untuk pemula karena tidak membutuhkan struktur package, tidak butuh `setup.py`, dan bisa langsung dijalankan.

Meski sederhana seperti *microframework*, Pyramid tetap fleksibel dan dapat diskalakan menjadi aplikasi besar seiring kebutuhan.

---

## **2. Struktur Proyek**

```
hello_world/
└── app.py
```

---

## **3. Cara Menjalankan**

1. Buat folder:

   ```bash
   mkdir hello_world
   cd hello_world
   ```
2. Isi file `app.py` dengan kode tutorial.
3. Jalankan aplikasi:

   ```bash
   $VENV/bin/python app.py
   ```
4. Buka:

   ```
   http://localhost:6543/
   ```

---

# **4. Analisis Kode**

### 🔹 **a. Baris `if __name__ == '__main__':`**

Baris ini memastikan bahwa blok kode di dalamnya hanya dijalankan ketika file dieksekusi secara langsung, bukan ketika di-*import* modul lain.
Ini membuat aplikasi lebih modular dan aman digunakan ulang.

---

### 🔹 **b. Baris konfigurasi (Configurator)**

```python
with Configurator() as config:
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
```

Bagian ini:

* membuat *route* (URL `/`)
* menghubungkan *view function* (`hello_world`) ke route tersebut
* menghasilkan objek WSGI app dengan `config.make_wsgi_app()`

**Configurator** adalah komponen inti Pyramid. Ia menggabungkan semua bagian aplikasi: routes, views, renderers, assets, security, dll.

---

### 🔹 **c. View Function**

```python
def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')
```

View menerima objek `request` dan harus mengembalikan objek **Response**.
Tanpa Response yang valid, server tidak akan dapat mengirim jawaban HTTP ke browser.

---

### 🔹 **d. Menjalankan WSGI App**

```python
serve(app, host='0.0.0.0', port=6543)
```

`waitress` adalah server WSGI yang digunakan untuk menjalankan aplikasi.

---

# **5. Extra Credit (Jawaban Lengkap)**

### **1. Mengapa memakai `print('Incoming request')` dan bukan `print 'Incoming request'`?**

Karena:

* `print 'text'` adalah **syntax Python 2**
* `print('text')` adalah **function pada Python 3**

Pyramid dan Python modern menggunakan Python 3, sehingga sintaks Python 2 akan error:

```
SyntaxError: Missing parentheses in call to 'print'
```

---

### **2. Apa yang terjadi jika view mengembalikan string HTML atau daftar angka?**

#### Mengembalikan **string HTML**

Contoh:

```python
return "<h1>Hello</h1>"
```

Ini **akan error** karena **WSGI mewajibkan return berupa objek Respons**, bukan string.
Error yang muncul biasanya:

```
TypeError: The view callable did not return a Response object
```

#### Mengembalikan **list / sequence of integers**

Contoh:

```python
return [1, 2, 3]
```

Ini juga salah karena WSGI output harus berupa **iterable of bytes**, bukan integer.
Akan muncul error seperti:

```
TypeError: sequence item 0: expected a bytes-like object, int found
```

---

### **3. Masukkan error seperti `print xyz`, lalu refresh browser**

Jika kamu mengetik:

```python
print xyz
```

Python akan error:

```
SyntaxError: invalid syntax
```

Ketika aplikasi dijalankan ulang (`Ctrl + C`, lalu jalankan lagi), saat browser mengakses route tersebut, server akan menampilkan traceback penuh di terminal.
Ini membantu debugging view function.

---

### **4. GI pada WSGI berarti “Gateway Interface”. Ini dimodelkan setelah standar apa?**

WSGI dimodelkan setelah standar lama dari dunia web:

### **CGI – Common Gateway Interface**

WSGI adalah penerus CGI tetapi lebih efisien:

* tidak membuat proses baru untuk tiap request
* lebih cepat dan sangat cocok untuk server modern

---

# **6. Kesimpulan**

Pyramid dapat digunakan sebagai microframework sederhana hanya dengan satu file Python.
Konsep-konsep seperti WSGI, request, response, dan routing diperkenalkan secara mendasar.
Meskipun sederhana, fondasi yang dipakai di sini adalah konsep inti dari aplikasi web Python skala besar.

---
<img width="957" height="1078" alt="SCREENSHOT BAGIAN 1" src="https://github.com/user-attachments/assets/949b4ebc-8ff2-4265-893b-ad57795ea133" />

