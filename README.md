Tentu, ini adalah konten lengkap dokumentasi teknis **Structify Music**. Anda bisa menyalin seluruh teks di dalam blok kode di bawah ini, lalu simpan sebagai file dengan nama `README.md` atau `TECHNICAL_DOCS.md`.

```markdown
# 🎵 Structify Music - Technical Documentation

**Structify Music** adalah aplikasi pemutar musik modern berbasis web yang dirancang untuk memenuhi tugas besar mata kuliah Struktur Data. Aplikasi ini menggabungkan struktur data klasik (Linked List, Queue) dengan antarmuka pengguna yang interaktif dan fitur cerdas.

Dokumen ini menjelaskan arsitektur teknis, mekanisme penyimpanan, serta algoritma yang digunakan dalam pengembangan aplikasi.

---

## 🏗️ Arsitektur & Penyimpanan (Storage)

Structify Music menggunakan pendekatan **Hybrid Storage** untuk menyeimbangkan kebutuhan integritas data relasional dan efisiensi operasi file.

### 1. Database Relasional (SQLite)
Digunakan untuk data entitas yang memerlukan relasi kuat dan integritas data.
* **Implementasi:** `Flask-SQLAlchemy`
* **Data:** `Album`, `Artist`, `PublicPlaylist` (Playlist Admin)
* **File:** `vibestream.db` *(Default configuration)*

### 2. File-Based Storage (CSV)
Digunakan sebagai simulasi *repository* file untuk Library Lagu utama, meniru sistem *legacy* atau pengelolaan file datar.
* **Implementasi:** Modul `csv` Python
* **Data:** Metadata Lagu (Judul, Artis, Genre, File Path, Nayara Key)
* **File:** `data/songs.csv`

### 3. Static File System
Penyimpanan aset media fisik yang diakses langsung oleh browser.
* **Audio:** `static/audio/` (.mp3)
* **Covers:** `static/covers/` (.jpg/.png)

### 4. Client-Side Storage (LocalStorage)
Digunakan untuk menyimpan preferensi pengguna dan playlist pribadi secara lokal di browser pengguna (tanpa membebani server database untuk sesi sementara).
* **Data:** `userPlaylists` (Playlist User), `musicHistory` (Riwayat putar), `vibestream_saved_collection`.

---

## 🧠 Algoritma & Struktur Data

Inti dari Structify Music adalah penerapan struktur data yang efisien untuk menangani operasi pemutar musik.

### 1. Doubly Linked List (DLL)
**Digunakan pada:** Library Lagu Utama (`song_library`).

**Alasan:** Memungkinkan penelusuran lagu (*Traversal*) dua arah (`Next` dan `Previous`) dengan efisien. Operasi penambahan lagu di akhir list memiliki kompleksitas waktu **O(1)** karena penggunaan *tail pointer*.

**Implementasi Code (Python - `music_service.py`):**
```python
# Menambahkan lagu ke Linked List
def add_song(library: DoublyLinkedSongList, data: dict) -> Song:
    # ... pembuatan objek song ...
    
    # Append ke Doubly Linked List 
    # O(1) karena menggunakan pointer tail
    library.append(song) 
    return song

# Update lagu (Traversal O(n) untuk pencarian, O(1) untuk update node)
def update_song(library: DoublyLinkedSongList, song_id: int, data: dict) -> bool:
    success = library.update_song(int(song_id), **fields)
    return success

```

###2. Queue (Singly Linked List)**Digunakan pada:** Antrean Pemutaran (*Playback Queue*) dan Playlist User.

**Alasan:** Sifat pemutaran musik adalah **FIFO (First-In-First-Out)** atau linear. Singly Linked List cukup efisien untuk operasi *enqueue* (tambah antrean) dan traversal satu arah.

**Implementasi Code (Python - `playlist_service.py`):**

```python
# Menambah lagu ke antrean
def add_to_queue(library: DoublyLinkedSongList, queue: Playlist, song_id: int) -> bool:
    node = library.find_by_id(int(song_id))
    if node is None:
        return False
    
    # Enqueue ke Queue (Menambah di ekor)
    queue.enqueue(node.song) 
    return True

```

###3. Levenshtein Distance (String Matching Algorithm)**Digunakan pada:** Fitur *Smart Search* (Pencarian).

**Alasan:** Memberikan toleransi terhadap kesalahan ketik (*typo*). Algoritma ini menghitung jumlah minimal operasi (insert, delete, replace) yang dibutuhkan untuk mengubah satu string menjadi string lain.

**Implementasi Code (Python - `music_service.py`):**

```python
def levenshtein_distance(s1, s2):
    if len(s1) < len(s2):
        return levenshtein_distance(s2, s1)
    
    if len(s2) == 0:
        return len(s1)
    
    previous_row = range(len(s2) + 1)
    for i, c1 in enumerate(s1):
        current_row = [i + 1]
        for j, c2 in enumerate(s2):
            insertions = previous_row[j + 1] + 1
            deletions = current_row[j] + 1
            substitutions = previous_row[j] + (c1 != c2)
            current_row.append(min(insertions, deletions, substitutions))
        previous_row = current_row
    
    return previous_row[-1]

```

###4. Graph-Based Logic / Weighted Heuristic**Digunakan pada:** Sistem Rekomendasi (*Auto Next*).

**Alasan:** Menentukan lagu selanjutnya bukan berdasarkan urutan ID, melainkan berdasarkan bobot kedekatan atribut ("Edge Weight"). Lagu dianggap "tetangga" jika memiliki kesamaan atribut.

* **Artist Sama:** Bobot +5
* **Genre Sama:** Bobot +3

**Implementasi Code (Python - `app.py`):**

```python
@app.route('/api/recommend/<song_id>')
def recommend_song(song_id):
    candidates = []
    # Iterasi untuk mencari tetangga yang relevan
    for song in all_songs_data:
        if str(song['id']) == str(song_id): continue
            
        weight = 0
        # Memberikan bobot jika memiliki atribut yang sama (Edge creation)
        if song.get('artist') == current_song.get('artist'): weight += 5
        if song.get('genre') and song.get('genre') == current_song.get('genre'): weight += 3
            
        if weight > 0: candidates.append(song)
    
    # Memilih acak dari kandidat yang memiliki bobot (koneksi graph)
    if candidates:
        next_song = random.choice(candidates)
        # ... return song

```

---

##✨ Fitur Utama###1. Manajemen Akses (Role-Based)* **Guest:** Akses terbatas untuk browsing dan pencarian lagu.
* **User:** Akses penuh untuk membuat playlist pribadi, memutar lagu, dan menyimpan ke koleksi.
* **Admin:** Dashboard khusus untuk operasi CRUD (Create, Read, Update, Delete) pada Lagu, Album, dan Artis.

###2. Music Player Canggih* **Kontrol Penuh:** Play, Pause, Next (Smart/Queue), Prev (History/Restart), Shuffle, Loop.
* **Visualizer 3D (Nayara Stage):** Integrasi `model-viewer` dengan Web Audio API untuk visualisasi 3D yang bereaksi terhadap *beat/bass* musik secara real-time.
* **Lirik Otomatis:** Fitur parsing file `.lrc` untuk menampilkan lirik yang sinkron dengan waktu lagu.

###3. Manajemen Playlist (Premium)* **LocalStorage System:** Playlist User disimpan di browser, memungkinkan manajemen state yang cepat tanpa *round-trip* ke server database setiap saat.
* **UI Glassmorphism:** Tampilan modern dengan efek blur dan transparansi.
* **Validasi:** Mencegah duplikasi lagu dalam satu playlist.

###4. Smart Search System* **Instant Search:** Hasil pencarian muncul seketika saat mengetik (AJAX Fetch).
* **Kategorisasi:** Hasil dipisah secara cerdas berdasarkan Lagu, Artis, Album, dan Playlist.
* **Typo Tolerance:** Menggunakan algoritma Levenshtein agar pengguna tetap menemukan lagu meski salah ketik.

---

##🛠️ Teknologi* **Backend:** Python, Flask, SQLAlchemy.
* **Frontend:** HTML5, CSS3 (Glassmorphism), Vanilla JavaScript (SPA Logic).
* **Database:** SQLite, CSV.
* **Media:** Web Audio API, Google Model Viewer.

```

```
