# Lustre: Framework UI Gleam

## Apa itu Lustre?

- Lustre adalah framework UI untuk Gleam.
- Konsep mirip React/Elm: bikin UI dengan fungsi `view()` → return tree element (div, button, dll).
- Dipakai bareng TailwindCSS supaya styling gampang.

---

## Compile ke Apa?

- Lustre **tidak menghasilkan binary** seperti Go/Rust.
- Compile ke JavaScript, lalu dijalankan di browser (frontend).

### Proses Compile

1. Gleam (kode Lustre)
2. ↓ compile
3. JavaScript
4. ↓ bundling (pakai lustre_dev_tools)
5. HTML + JS + CSS
6. ↓
7. Jalan di browser

- Hasil akhirnya: web app (HTML + JS + CSS).
- Backend Gleam (target Erlang/BEAM) → binary `.beam` di server Erlang/Elixir.
- Lustre khusus frontend → outputnya JavaScript.

---

## Contoh Sederhana Lustre + Tailwind

```gleam
import lustre/element.{div, button, text}
import lustre/attribute.{class, on_click}

pub fn view() {
  div([class("flex flex-col items-center p-8")], [
    text("Halo dari Lustre 👋"),
    button([
      class("mt-4 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"),
      on_click(fn(_event) { io.println("Tombol diklik!") })
    ], [
      text("Klik Saya")
    ])
  ])
}
```

- Compile → hasil akhirnya JS + HTML, bisa dibuka di browser.

---

## Ringkasan

- **Gleam** → bahasa (compile ke Erlang atau JS).
- **Lustre** → framework UI untuk Gleam (compile ke JS).
- **Hasil akhir Lustre** → bukan binary, tapi web app (seperti React/Vue/Svelte).

---

## Lustre Fullstack

- **Frontend** → Lustre (ditulis dengan Gleam, compile ke JavaScript).
- **Backend** → Gleam (ditargetkan ke Erlang/BEAM).

Karena Gleam bisa compile ke dua dunia:
- BEAM (server, concurrency, seperti Elixir/Erlang)
- JS (browser, UI, seperti React/Svelte)

→ Gleam + Lustre bisa dipakai untuk bikin fullstack web app dengan satu bahasa.

---

## Contoh Arsitektur Fullstack Lustre

### Backend (Gleam on BEAM)

- Bikin REST API atau GraphQL (pakai framework Gleam seperti wisp atau OTP style Erlang).

```gleam
import gleam/http

pub fn handle_request(req: http.Request) {
  case req.path {
    "/api/ping" -> http.ok("pong")
    _ -> http.not_found()
  }
}
```

- Compile → jadi `.beam` file, jalan di server Erlang/Elixir.

### Frontend (Lustre)

- UI ditulis pakai Lustre + Tailwind.
- Bisa fetch data dari API backend Gleam.

```gleam
import lustre/element.{div, text}
import lustre/attribute.{class}
import lustre/http

pub fn view() {
  div([class("p-4")], [
    text("Frontend Lustre 🚀"),
  ])
}

pub fn init() {
  // contoh fetch API backend
  http.get("/api/ping", fn(result) {
    case result {
      Ok(resp) -> io.println("Backend jawab: " <> resp.body)
      Error(e) -> io.println("Error: " <> e)
    }
  })
}
```

- Compile → jadi JS → disajikan bareng backend.

---

## Alur Fullstack

- Backend (BEAM binary) serve API + static files (HTML, JS, CSS).
- Frontend (Lustre → JS) render UI di browser.
- Semua ditulis pakai Gleam → stack full Gleam.

---

## Ringkasan Fullstack Lustre

- "Fullstack" di Lustre docs: pakai Gleam di backend (BEAM) dan frontend (Lustre → JS).
- Mirip Elixir + Phoenix LiveView, atau TypeScript + Next.js/Express, tapi 100% Gleam.

---

## Cara Deploy Lustre Fullstack ke Production

### 1. Frontend Only (SPA)

- Hasil akhir: JavaScript + HTML + CSS (seperti deploy React/Vue/Svelte).

**Step di production:**
1. Compile proyek:
   ```sh
   gleam build
   gleam run -m lustre/dev build
   ```
   (atau pakai lustre_dev_tools buat bundling)
2. Hasil file statis:
   ```
   build/
     ├── index.html
     ├── app.js
     └── app.css
   ```
3. Deploy ke hosting statis:
   - Netlify / Vercel
   - Cloudflare Pages
   - Nginx / Apache (serve index.html, app.js, app.css)

---

### 2. Fullstack (Frontend + Backend Gleam)

- Backend Gleam jalan di BEAM VM (seperti Elixir/Erlang).
- Frontend (Lustre) tetap di-compile jadi JS, disajikan via backend.

**Step di production:**
1. Compile backend Gleam (target Erlang/BEAM):
   ```sh
   gleam build
   ```
   → hasilnya `.beam` files.
2. Buat release (paket BEAM, bisa digabung pakai rebar3 / erlang.mk / gleam export erlang-shipment).
3. Deploy ke server:
   - Jalankan di Erlang runtime (BEAM VM)
   - Bisa pakai Docker
   - Bisa taruh di server bare metal, Fly.io, Heroku, Gigalixir, dsb.
4. Backend serve API + static files (hasil compile Lustre).

---

### 3. Contoh Flow Fullstack di Production

Misal bikin app Todo:

- Frontend (Lustre) → compile ke app.js + app.css
- Backend (Gleam on BEAM) → endpoint `/api/todos`
- Server serve index.html + asset (frontend), dan handle `/api/...` routes

**Production flow:**
- Client Browser → index.html + app.js + app.css (hasil Lustre)
- Client Browser → `/api/todos` (Gleam backend di BEAM)

---

### 4. Docker Deployment Contoh

Dockerfile untuk fullstack Gleam + Lustre:

```dockerfile
FROM ghcr.io/gleam-lang/gleam:latest as build

WORKDIR /app
COPY . .

# Build backend + frontend
RUN gleam build --target erlang
RUN gleam run -m lustre/dev build

# Stage runtime
FROM erlang:26
WORKDIR /app

# Copy beam files & static assets
COPY --from=build /app/build/erlang .
COPY --from=build /app/build/* /app/priv/static/

CMD ["erl", "-pa", "ebin", "-s", "my_app"]
```

---

## Ringkasan

- **Frontend Lustre saja** → deploy ke hosting statis (Netlify, Vercel, Nginx).
- **Fullstack Gleam + Lustre** →
  - Backend jalan di BEAM VM (seperti deploy Elixir/Erlang).
  - Frontend tetap file statis hasil compile.
  - Bisa bungkus semua ke Docker dan jalankan di server.

**Jadi, running di production mirip banget dengan Elixir Phoenix kalau fullstack, atau mirip React SPA kalau frontend only.**