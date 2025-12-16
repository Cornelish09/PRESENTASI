# 🎵 Structify Music — Technical Documentation

Structify Music adalah aplikasi pemutar musik modern berbasis web untuk tugas besar **Struktur Data**. Fokus utamanya: penerapan struktur data klasik (**Doubly Linked List**, **Queue**) + UI interaktif + fitur cerdas (search toleran typo & rekomendasi lagu).

---

## 📌 Table of Contents
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [Architecture & Storage](#️-architecture--storage)
- [Algorithms & Data Structures](#-algorithms--data-structures)
- [Key Features](#-key-features)
- [API Overview](#-api-overview)
- [Tech Stack](#-tech-stack)

---

## 🚀 Quick Start

### Prerequisites
- Python 3.10+ (recommended)
- pip / venv

### Setup & Run
```bash
# 1) create env
python -m venv .venv

# 2) activate env (Windows)
.venv\Scripts\activate

# 3) install deps
pip install -r requirements.txt

# 4) run (option A)
python app.py

# or run (option B - flask cli)
flask --app app.py --debug run
```

Open:
- `http://127.0.0.1:5000`

---

## 🗂 Project Structure

Contoh struktur folder yang rapi untuk GitHub:

```text
structify-music/
├─ app.py
├─ models.py
├─ services/
│  ├─ player_service.py
│  ├─ playlist_service.py
│  └─ music_service.py
├─ repository/
│  └─ songs_repository.py
├─ data/
│  └─ songs.csv
├─ static/
│  ├─ audio/
│  ├─ covers/
│  ├─ css/
│  │  └─ style.css
│  └─ js/
│     └─ app.js
├─ templates/
│  ├─ base.html
│  ├─ guest_dashboard.html
│  ├─ gatau.html
│  ├─ search_results.html
│  ├─ auth_login.html
│  └─ auth_register.html
├─ requirements.txt
└─ README.md
```

---

## 🏗️ Architecture & Storage

Structify Music memakai **Hybrid Storage**: relasional untuk entitas yang butuh integritas relasi + file-based untuk simulasi library lagu seperti sistem legacy.

### 1) Relational Database (SQLite)

Digunakan untuk data yang membutuhkan struktur hubungan (relasi) yang rapi dan fitur query yang kompleks.

Teknologi: SQLite (diakses via library Flask-SQLAlchemy).

File Fisik: vibestream.db (akan dibuat otomatis oleh app.py).

Data yang Disimpan:

Tabel Album: Menyimpan data album (Judul, Artis, Tahun) dan relasi ke lagu-lagu di dalamnya.

Tabel Artist: Menyimpan profil artis (Nama, Bio, Avatar, Status Verified).

Tabel PublicPlaylist: Menyimpan playlist yang dibuat oleh Admin (agar bisa dilihat semua orang).
Untuk data dengan relasi kuat & integritas data.
- Implementasi: `Flask-SQLAlchemy`
- Entitas: `Album`, `Artist`, `PublicPlaylist` (Playlist Admin)
- Default file: `vibestream.db`

### 2) File-Based Storage (CSV)
Simulasi repository lagu utama (flat-file).
- Implementasi: modul `csv` Python
- Isi: metadata lagu (judul, artis, genre, path audio, cover, key Nayara, dll.)
- File: `data/songs.csv`

### 3) Static File System
Aset media fisik yang diakses langsung oleh browser.
- Audio: `static/audio/` (`.mp3`)
- Cover: `static/covers/` (`.jpg`/`.png`)

### 4) Client-Side Storage (LocalStorage)
Penyimpanan preferensi & playlist pribadi di browser (ringan, cepat, tanpa beban server untuk state sementara).
- Contoh key: `userPlaylists`, `musicHistory`, `vibestream_saved_collection`

---

## 🧠 Algorithms & Data Structures

### 1) Doubly Linked List (DLL)
**Dipakai untuk:** Library Lagu Utama (`song_library`)

**Kenapa:** traversal dua arah (`next`/`prev`) efisien. Append O(1) karena ada *tail pointer*.

**Contoh (Python — `music_service.py`):**
```python
def add_song(library, data: dict):
    # ... create Song object ...
    # Append O(1) karena tail pointer
    library.append(song)
    return song

def update_song(library, song_id: int, data: dict) -> bool:
    # Search O(n), update node O(1)
    return library.update_song(int(song_id), **data)
```

---

### 2) Queue (Singly Linked List)
**Dipakai untuk:** Antrean pemutaran (*Playback Queue*) & playlist user

**Kenapa:** pemutaran bersifat FIFO. Enqueue/dequeue efisien.

**Contoh (Python — `playlist_service.py`):**
```python
def add_to_queue(library, queue, song_id: int) -> bool:
    node = library.find_by_id(int(song_id))
    if node is None:
        return False
    queue.enqueue(node.song)  # add to tail
    return True
```

---

### 3) Levenshtein Distance (String Matching)
**Dipakai untuk:** Smart Search (toleran typo)

**Kenapa:** menghitung jarak edit minimal (insert/delete/replace) agar hasil tetap relevan walau salah ketik.

**Contoh (Python — `music_service.py`):**
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

---

### 4) Weighted Heuristic (Graph-Like Recommendation)
**Dipakai untuk:** Auto Next / rekomendasi lagu

**Konsep:** lagu jadi “tetangga” jika punya atribut sama, diberi bobot (*edge weight*).
- Artist sama: +5
- Genre sama: +3

**Contoh (Python — `app.py`):**
```python
@app.route('/api/recommend/<song_id>')
def recommend_song(song_id):
    candidates = []
    for song in all_songs_data:
        if str(song["id"]) == str(song_id):
            continue

        weight = 0
        if song.get("artist") == current_song.get("artist"):
            weight += 5
        if song.get("genre") and song.get("genre") == current_song.get("genre"):
            weight += 3

        if weight > 0:
            candidates.append(song)

    if candidates:
        next_song = random.choice(candidates)
        # ... return serialized song ...
```

---

## ✨ Key Features

### 1) Role-Based Access
- **Guest:** browsing & search terbatas
- **User:** playlist pribadi, queue, koleksi
- **Admin:** dashboard CRUD (lagu/album/artis)

### 2) Advanced Music Player
- Kontrol: play, pause, next (smart/queue), prev, shuffle, loop
- **Nayara Stage (3D):** integrasi `model-viewer` + Web Audio API untuk visualisasi reaktif audio
- (Opsional) Lirik `.lrc` sinkron waktu jika tersedia

### 3) Playlist Management (Premium / User)
- Playlist user via LocalStorage (cepat, ringan)
- Validasi anti-duplikasi lagu
- UI modern (glassmorphism)

### 4) Smart Search
- Instant search (AJAX fetch)
- Kategorisasi hasil (lagu / artis / album / playlist)
- Typo tolerance (Levenshtein)

---

## 🔌 API Overview

> Catatan: nama endpoint bisa menyesuaikan implementasi kamu — ini format yang umum dari project-mu.

- `GET /api/next/<id>` → lagu berikutnya (smart next / queue mode)
- `GET /api/prev/<id>` → lagu sebelumnya
- `GET /api/playlist` → ambil isi queue
- `POST /api/playlist/add` → tambah ke queue
- `POST /api/playlist/remove` → hapus dari queue
- `POST /api/playlist/clear` → bersihkan queue
- `GET /api/playlists` → daftar playlist user
- `POST /api/playlists/create` → buat playlist
- `POST /api/playlists/<name>/add` → tambah lagu ke playlist
- `POST /api/playlists/<name>/remove` → hapus lagu dari playlist
- `POST /api/playlists/<name>/clear` → clear playlist
- `DELETE /api/playlists/<name>` → hapus playlist

---

## 🛠 Tech Stack
- Backend: Python, Flask, SQLAlchemy
- Frontend: HTML5, CSS3, Vanilla JS
- Storage: SQLite + CSV
- Media: Web Audio API, `model-viewer`

---

### ✅ Notes (untuk GitHub)
- Pastikan `data/songs.csv` ada (minimal 1 baris data) biar app tidak error saat load.
- Pastikan file audio & cover yang direferensikan di CSV memang ada di folder `static/`.

