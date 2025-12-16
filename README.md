# PRESENTASI

# 🎵 VibeStream - Technical Documentation

**VibeStream** adalah aplikasi pemutar musik modern berbasis web yang menggabungkan struktur data klasik dengan antarmuka pengguna yang interaktif. Dokumen ini menjelaskan arsitektur teknis, mekanisme penyimpanan, serta algoritma yang digunakan dalam pengembangan aplikasi.

## 🏗️ Arsitektur & Penyimpanan (Storage)

VibeStream menggunakan pendekatan **Hybrid Storage** untuk mengoptimalkan kinerja antara manajemen data relasional yang kompleks dan kecepatan akses file.

### 1. Database Relasional (SQLite)
Digunakan untuk data yang memerlukan integritas tinggi dan relasi antar entitas.
* **Implementasi:** Flask-SQLAlchemy.
* **Data:** `Album`, `Artist`, `PublicPlaylist`.
* **File:** `vibestream.db`.

### 2. File-Based Storage (CSV)
Digunakan sebagai simulasi *repository* file untuk Library Lagu utama, meniru sistem *legacy* atau pengelolaan file datar.
* **Implementasi:** Modul `csv` Python.
* **Data:** Metadata Lagu (Judul, Artis, Genre, File Path).
* **File:** `data/songs.csv`.

### 3. Static File System
Penyimpanan aset media fisik.
* **Audio:** `static/audio/` (.mp3)
* **Covers:** `static/covers/` (.jpg/.png)

### 4. Client-Side Storage (LocalStorage)
Digunakan untuk menyimpan preferensi pengguna dan playlist pribadi tanpa membebani server database untuk sesi sementara.
* **Data:** `userPlaylists`, `musicHistory` (Riwayat putar), `vibestream_saved_collection`.

---

## 🧠 Algoritma & Struktur Data

Inti dari VibeStream adalah penerapan struktur data yang efisien untuk menangani operasi pemutar musik.

### 1. Doubly Linked List (DLL)
**Digunakan pada:** Library Lagu Utama (`song_library`).

**Alasan:** Memungkinkan penelusuran lagu (Traversal) dua arah (`Next` dan `Previous`) dengan efisien. Operasi penambahan lagu di akhir list memiliki kompleksitas waktu **O(1)** karena penggunaan *tail pointer*.

**Implementasi Code (Python):**
```python
# Menambahkan lagu ke Linked List (music_service.py)
def add_song(library: DoublyLinkedSongList, data: dict) -> Song:
    # ... pembuatan objek song ...
    
    # Append ke Doubly Linked List 
    # O(1) karena menggunakan pointer tail
    library.append(song) 
    return song

# Update lagu (Traversal O(n))
def update_song(library: DoublyLinkedSongList, song_id: int, data: dict) -> bool:
    success = library.update_song(int(song_id), **fields)
    return success
