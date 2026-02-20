# MusicPlayer (Spotify-style React Native App)

Ứng dụng nghe nhạc mobile (React Native + Expo) theo phong cách Spotify, đi kèm backend Node.js/Express để cung cấp dữ liệu bài hát, nghệ sĩ, album, tìm kiếm và streaming audio.

## 1) Tổng quan

Project gồm 2 phần:

- **Mobile App** (`React Native + Expo`): phát nhạc, quản lý queue, liked songs, playlist, tìm kiếm và duyệt thư viện.
- **Backend API** (`Node.js + Express`): trả metadata từ Supabase, hỗ trợ stream audio, artwork, ingest/scan nhạc local và enrich metadata từ Spotify API.

## 2) Tính năng chính

### Mobile

- Player đầy đủ: `play/pause`, `seek`, `next/prev`, `shuffle`, `repeat`, mini player, queue screen.
- Màn hình: Home, Search, Artists, Library, Album, Artist, Playlist, Now Playing.
- Lưu trạng thái local với `AsyncStorage`:
  - Trạng thái player (`queue`, vị trí track đang phát)
  - `Liked Songs`
  - `My Playlist`
- Search realtime có debounce ở client.
- Haptic feedback cho tương tác chính.

### Backend

- API resources: `tracks`, `artists`, `albums`, `search`, `ingest`, `health`.
- Streaming endpoint: hỗ trợ HTTP Range (`206 Partial Content`) cho tua/phát ổn định.
- Artwork endpoint: ưu tiên ảnh từ storage, fallback ảnh Spotify.
- Ingest pipeline:
  - Scan file audio local (`mp3`, `m4a`, `flac`, `wav`)
  - Đọc metadata với `music-metadata`
  - Enrich thông tin track/artist/album từ Spotify API
  - Upsert vào Supabase

## 3) Công nghệ sử dụng

### Mobile

- `React Native 0.73`
- `Expo 50`
- `TypeScript`
- `Zustand`
- `React Navigation` (bottom tabs + native stack)
- `react-native-track-player`
- `react-native-reanimated`
- `react-native-gesture-handler`
- `@react-native-async-storage/async-storage`

### Server

- `Node.js + Express`
- `TypeScript`
- `Supabase` (`@supabase/supabase-js`)
- `Spotify Web API` (`axios`)
- `music-metadata`
- `mime-types`

## 4) Cấu trúc thư mục

```text
MusicPlayer/
├─ src/                  # React Native app source
├─ assets/
├─ server/
│  ├─ src/               # Express API source
│  ├─ scripts/           # Upload audio/artwork scripts
│  └─ music/             # (tuỳ chọn) local media cho ingest/dev
├─ App.tsx
└─ package.json
```

## 5) Yêu cầu môi trường

- `Node.js >= 18`
- `npm` hoặc `yarn`
- Expo CLI (qua `npx expo`)
- Android Studio/Xcode (nếu chạy simulator/device native)
- Supabase project (bắt buộc cho backend chạy đầy đủ)

## 6) Biến môi trường

### 6.1 Mobile app

Tạo file `.env` ở root project:

```env
EXPO_PUBLIC_API_URL=http://<YOUR_LOCAL_IP>:4000
```

> Dùng IP LAN thay vì `localhost` khi chạy app trên thiết bị thật.

### 6.2 Server

Tạo file `.env` trong thư mục `server/`:

```env
PORT=4000
MEDIA_DIR=./music
NODE_ENV=development

SUPABASE_URL=...
SUPABASE_SERVICE_ROLE=...

SPOTIFY_CLIENT_ID=...         # optional nhưng nên có để enrich metadata/search
SPOTIFY_CLIENT_SECRET=...

SUPABASE_BUCKET_MUSIC=track-audio
SUPABASE_BUCKET_ARTWORK=track-artwork
CONCURRENCY=2
```

## 7) Cài đặt & chạy project

### 7.1 Chạy backend

```bash
cd server
npm install
npm run dev
```

Backend mặc định chạy tại `http://localhost:4000` (theo `PORT`).

### 7.2 Chạy mobile app

```bash
cd ..
npm install
npm run start
```

Chạy trên Android:

```bash
npm run android
```

## 8) Ingest dữ liệu nhạc

### Cách 1: scan local media bằng API

Gọi endpoint:

```http
POST /ingest/scan
Content-Type: application/json

{
  "root": "<duong_dan_thu_muc_nhac>"
}
```

### Cách 2: upload asset lên Supabase bằng scripts

```bash
cd server
node scripts/upload-music.js
node scripts/upload-artwork.js
```

## 9) API chính

- `GET /health`
- `GET /tracks?limit=200&order=title|recent`
- `GET /tracks/:id`
- `GET /tracks/:id/stream`
- `GET /tracks/:id/artwork`
- `PATCH /tracks/:id`
- `GET /artists`
- `GET /artists/:id`
- `GET /artists/:id/tracks`
- `GET /albums`
- `GET /albums/:id`
- `GET /albums/:id/tracks`
- `GET /search?q=...`
- `POST /ingest/scan`

## 10) Ghi chú triển khai

- Server cần Supabase để trả dữ liệu chính.
- `stream` ưu tiên `audio_url` (cloud), fallback file local cho môi trường dev.
- Search ở app hiện chạy theo dữ liệu tracks lấy về client (debounce), trong khi backend cũng có endpoint `/search` (Spotify).

## 11) Định hướng mở rộng

- Đồng bộ auth/user profile thực tế.
- Tối ưu cache/offline playback.
- Bổ sung test cho API + store.
- Tối ưu indexing/search phía backend cho thư viện lớn.

---

Nếu bạn dùng README này cho CV/portfolio, có thể thêm nhanh:

- Demo gif/screenshot từng màn hình
- Kiến trúc data flow (App ↔ API ↔ Supabase/Spotify)
- Các số liệu benchmark (thời gian load, số track ingest, v.v.)
