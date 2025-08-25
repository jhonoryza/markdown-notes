# Ekosistem Elixir di BEAM VM

## 1. Web Framework: Phoenix

Framework utama untuk web development di Elixir.  
Setara dengan Rails (Ruby) atau Django (Python).

**Fitur bawaan:**
- MVC (Model-View-Controller)
- WebSockets
- Live reload dev server
- Built-in test tools

> Kalau mau bikin web app besar di Elixir, Phoenix adalah pilihan default.

---

## 2. Realtime UI: Phoenix LiveView

Bagian dari Phoenix untuk membuat UI interaktif tanpa JavaScript.  
Render UI di server, sinkronisasi ke browser via WebSocket.

**Dipakai untuk:**
- Dashboard
- Aplikasi bisnis
- Chat
- dsb.

Alternatif ringan dibanding SPA (React/Vue).

> LiveView mirip dengan Lustre, bedanya:
> - Lustre: compile ke JS → jalan di browser.
> - LiveView: render di server → update ke browser.

---

## 3. Database & Ecto

- Ecto = ORM + query builder Elixir.
- Mendukung PostgreSQL, MySQL, dsb.
- Integrasi erat dengan Phoenix.

---

## 4. Testing & OTP

- Elixir built-in support testing (ExUnit).
- Karena di BEAM, Elixir otomatis dapat OTP (supervisor, GenServer, task async, dsb).
- Sangat kuat untuk concurrent & fault-tolerant system.

---

## 5. Ecosystem Libraries

- **Absinthe** → GraphQL server (kayak Apollo/Hasura)
- **Broadway** → Data pipelines (Kafka, RabbitMQ, S3, dsb)
- **Nerves** → IoT / embedded system
- **Oban** → Job processing (mirip Sidekiq)
- **Plug** → Middleware layer HTTP (setara dengan Rack di Ruby)

---

## Ringkasan Ekosistem Elixir

| Layer            | Tool/Framework    | Fungsi                                      |
|------------------|------------------|----------------------------------------------|
| Web server       | Phoenix          | MVC, REST, WebSocket, LiveView               |
| UI realtime      | Phoenix LiveView | SPA tanpa JS, reactive UI                    |
| Database         | Ecto             | ORM/query builder                            |
| GraphQL          | Absinthe         | GraphQL server                               |
| Background jobs  | Oban             | Job queue & scheduler                        |
| Streaming        | Broadway         | Data pipelines & message queues              |
| IoT/Embedded     | Nerves           | Elixir di hardware kecil (Raspberry Pi)      |

---

## Perbandingan dengan Gleam

- **Elixir** sudah lengkap: Phoenix (web), LiveView (UI), Ecto (DB), Oban (jobs).
- **Gleam** masih baru: Mist, Wisp, Glee (backend), Lustre (frontend).
- Gleam sedang menuju ekosistem seperti Elixir, tapi dengan static typing ala Rust/Elm.

> Jadi, kalau di Elixir:
> - Mau bikin web app → Phoenix (pasti)
> - Mau UI reactive tanpa SPA → LiveView
> - Mau GraphQL → Absinthe
> - Mau job queue → Oban
> - Mau IoT → Nerves

Elixir sudah seperti “Rails on BEAM”, sedangkan Gleam masih di fase “bangun pondasi framework” (Mist/Wisp/Glee/Lustre).

---

## Framework Elixir

| Framework | Level      | Cocok Untuk                                   | Status Ekosistem      |
|-----------|------------|-----------------------------------------------|-----------------------|
| Phoenix   | Full-stack | Web app besar, real-time, REST, LiveView      | Super aktif ✅         |
| Plug      | Low-level  | Microservice kecil, custom middleware         | Stabil ✅              |
| Raxx      | Low-level  | HTTP/2, WebSocket, streaming                  | Aktif tapi niche      |
| Maru      | API-focused| REST API sederhana                            | Kurang populer        |
| Sugar     | Full-stack | (historis, jarang dipakai sekarang)           | Deprecated ❌          |
| Absinthe  | GraphQL-only| GraphQL API                                  | Sangat aktif ✅        |

---

## Ringkasan

- **Phoenix** → default & ekosistem paling lengkap.
- Kalau butuh ringan → Plug.
- Kalau butuh streaming / eksperimen → Raxx.
- Kalau GraphQL API only → Absinthe.
- Framework lain (Sugar, Maru) sudah jarang dipakai.