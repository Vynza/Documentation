# VYNZA System Architecture & API Specification

## 1. System Overview

VYNZA is a scalable, modern music streaming ecosystem consisting of:
- **Backend**: Go-based high performance RESTful API with SQLite database (zero-dependency local setup, expandable to PostgreSQL), JWT authentication, role-based access control (User & Admin), audio streaming endpoint with HTTP Range support, audio/cover file uploads and catalog management.
- **WebApp**: React + TypeScript + Vite web music player with modern dark/purple aesthetic, real-time audio playback bar, queue management, library/playlist support, discovery, and search.
- **AdminDashboard**: React + TypeScript + Vite administrative dashboard with statistics, song/album/artist catalog CRUD management, and user oversight.
- **AndroidApp**: Native Kotlin Android application architecture with Retrofit / OkHttp API clients, ExoPlayer media session streaming, and Jetpack MVVM architecture.
- **LandingPage**: Official modern landing page showcasing VYNZA branding, download links, and features.

```
                  +----------------------------------------------+
                  |               Client Tier                    |
                  |                                              |
                  |   [ WebApp (React) ]    [ AndroidApp (Kotlin)]
                  |            \                  /              |
                  |             \                /               |
                  |     [ AdminDashboard (React) ]               |
                  +-----------------------+----------------------+
                                          |
                                          | HTTPS / REST & Audio Streaming (HTTP Range)
                                          v
                  +----------------------------------------------+
                  |               VYNZA Backend (Go)             |
                  |  - Router & CORS Middleware                  |
                  |  - JWT Auth & Role Middleware                |
                  |  - Music Streaming & Static Assets           |
                  |  - Catalog & Playlist Handlers               |
                  |  - Analytics & Admin Endpoints               |
                  +-----------------------+----------------------+
                                          |
                                          v
                  +----------------------------------------------+
                  |                 Data Tier                    |
                  |  - SQLite (vynza.db)                         |
                  |  - Audio Storage (/uploads/audio)            |
                  |  - Artwork Storage (/uploads/covers)         |
                  +----------------------------------------------+
```

## 2. API Contract Specification

Base URL: `http://localhost:8080/api/v1`

### Authentication & User
- `POST /auth/register` - Register a new listener `{ username, email, password }`
- `POST /auth/login` - Authenticate `{ email, password }` -> `{ token, user: { id, username, email, role } }`
- `GET /auth/me` - Get current authenticated user profile

### Catalog & Music Discovery
- `GET /songs` - List songs (supports `?search=`, `?genre=`, `?artist_id=`)
- `GET /songs/:id` - Song details
- `GET /songs/:id/stream` - Audio streaming endpoint (supports `Range: bytes=start-end`)
- `GET /artists` - List artists
- `GET /artists/:id` - Artist detail with albums and top songs
- `GET /albums` - List albums
- `GET /albums/:id` - Album detail with tracklist

### Playlists & Favorites (User)
- `GET /playlists` - User's playlists
- `POST /playlists` - Create playlist `{ title, description }`
- `GET /playlists/:id` - Get playlist with tracks
- `POST /playlists/:id/songs` - Add song to playlist `{ song_id }`
- `DELETE /playlists/:id/songs/:song_id` - Remove song from playlist
- `GET /favorites` - User favorite songs
- `POST /favorites/:song_id` - Add song to favorites
- `DELETE /favorites/:song_id` - Remove song from favorites

### Admin Endpoints (Requires Admin Role)
- `GET /admin/stats` - Total songs, artists, albums, users, total streams
- `POST /admin/songs` - Upload/create song (multipart/form-data: title, artist_id, album_id, duration, audio file, cover file)
- `PUT /admin/songs/:id` - Edit song metadata
- `DELETE /admin/songs/:id` - Delete song
- `POST /admin/artists` - Create artist
- `POST /admin/albums` - Create album
- `GET /admin/users` - Manage users
