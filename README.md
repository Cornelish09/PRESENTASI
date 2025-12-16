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
