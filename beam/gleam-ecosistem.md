# Ekosistem Gleam di BEAM

## 1. Lustre

- Framework UI full-fledged untuk Gleam (mirip Elm/React).
- Compile ke JavaScript (frontend).
- Ada arsitektur ala Elm: Model, Msg, update, view.
- Bisa dipakai fullstack (frontend JS + backend BEAM).
- **Pilihan utama** untuk bikin SPA/website modern dengan Gleam.

## 2. Mist

- [`gleam/mist`](https://github.com/gleam-experiments/mist): framework HTTP backend untuk Gleam (mirip Express.js atau Plug di Elixir).
- Cocok untuk bikin REST API / web server.
- Fokus ke backend, bukan UI.
- **Pakai Mist** jika backend Gleam + frontend framework lain (React, Vue, atau Lustre).

## 3. Wisp

- [`gleam/wisp`](https://github.com/gleam-experiments/wisp): framework HTTP ringan untuk Gleam.
- Bisa serve HTML, JSON, static files.
- Cocok untuk microservices.
- **Pakai Wisp** jika butuh backend simpel, lebih ringan dari Mist.

## 4. Glee

- [`Glee`](https://github.com/gleam-experiments/glee): framework untuk GraphQL API di Gleam.
- Bikin GraphQL server langsung di Gleam.
- Cocok untuk fullstack GraphQL (frontend Lustre, backend Glee).

## 5. Native HTML/JS Interop

- Gleam bisa compile ke JavaScript, sehingga bisa:
  - Interop ke DOM langsung (tanpa Lustre).
  - Integrasi dengan React/Vue/Svelte.
- **Jarang dipakai**, karena Lustre lebih ergonomis.

## 6. Eksperimen Lain di Komunitas Gleam

- Beberapa project kecil untuk templating HTML server-side.
- Ada tool untuk SSR (server-side rendering) dengan Gleam, tapi belum mainstream.
- Ekosistem frontend Gleam masih muda, jadi Lustre jadi fokus utama.

---

## Ringkasan

### UI Frontend di Gleam

- **Lustre** → pilihan utama (Elm-like).
- **Raw JS interop** → bisa, tapi lebih ribet.

### Backend Gleam

- **Mist** → HTTP framework lengkap.
- **Wisp** → HTTP framework ringan.
- **Glee** → GraphQL.

**Jadi:** Lustre untuk frontend, Mist/Wisp/Glee untuk backend.  
Jika digabung, bisa bikin fullstack Gleam end-to-end.

---

## Framework / Library Populer di Gleam

| Nama              | Jenis         | Target                | Kapan Dipakai                                                                 |
|-------------------|---------------|-----------------------|-------------------------------------------------------------------------------|
| Lustre            | UI Framework  | Compile ke JS         | Frontend web app (SPA/komponen interaktif), mirip Elm/React                   |
| Mist              | Web Framework | BEAM                  | Backend web server full-featured (REST API, middleware, dsb)                  |
| Wisp              | Web Framework | BEAM                  | HTTP server ringan, cocok untuk microservices atau API kecil                  |
| Glee              | GraphQL       | BEAM                  | Bikin GraphQL API di Gleam, backend modern                                    |
| glailglind        | Dev tool      | BEAM/JS (build tool)  | Integrasi TailwindCSS di proyek Gleam/Lustre                                  |
| glailwind_merge   | Utility       | BEAM/JS               | Helper untuk menggabungkan class Tailwind tanpa konflik                       |

---

## Contoh Skenario

- **Frontend SPA** → Lustre (+ Tailwind)
- **Backend REST API** → Mist
- **Backend ringan / microservice** → Wisp
- **Backend GraphQL** → Glee
- **Fullstack Gleam** → frontend Lustre + backend Mist/Wisp/Glee

---

## Analogi Ekosistem

- Erlang punya OTP.
- Elixir punya Phoenix.
- Gleam punya Mist/Wisp/Glee (backend) + Lustre (frontend).