# Erlang, Elixir, dan Gleam

## Perbandingan Singkat

|                | Erlang   | Elixir   | Gleam            |
|----------------|----------|----------|------------------|
| Tahun lahir    | 1986     | 2012     | 2019             |
| Sintaks        | Prolog-like (jadul) | Ruby-like (ramah) | Elm/Rust-like (bersih) |
| Typing         | Dynamic  | Dynamic  | Static, strong ✅ |
| Compile target | BEAM     | BEAM     | BEAM atau JS     |
| Ekosistem      | Tua, stabil, battle-tested | Modern, Phoenix, aktif | Masih kecil, berkembang |
| Fokus          | Telecom, distribusi | Web dev, DX bagus | Type safety, DX modern |
| Popularitas    | Industri telco/infra | Startup & web framework | Komunitas niche, growing |

---

## Analogi Kasar

- **Erlang** → Bahasa C: tua, low-level di BEAM, powerful tapi kaku.
- **Elixir** → Bahasa Ruby: sintaks manis, banyak dipakai web dev, cepat bikin produk.
- **Gleam** → Bahasa Rust/TypeScript: aman, modern, compiler bantu banget cegah bug.

**Jadi:**
- Battle-tested infra → Erlang.
- Produktif bikin web/app → Elixir.
- Suka type safety & syntax modern → Gleam.

---

## BEAM VM vs JVM

|                | BEAM (Erlang VM)         | JVM (Java VM)           |
|----------------|--------------------------|-------------------------|
| Asal           | Erlang (Ericsson, 1986)  | Java (Sun, 1995)        |
| Compile target | .beam files (Erlang/Gleam/Elixir) | .class / .jar (Java/Kotlin/Scala) |
| Execution      | Run bytecode di VM        | Run bytecode di VM      |
| Fokus desain   | Concurrency, fault-tolerance, distribusi | Performance umum, portability |
| Concurrency    | Lightweight processes (jutaan actor) | Thread OS (lebih berat) |
| Crash handling | "Let it crash", supervisor trees | Exception handling tradisional |
| Distribusi     | Node Erlang bisa cluster out-of-the-box | Java cluster pakai framework eksternal |
| Garbage collection | Per-process GC (independen, kecil-kecil) | Global GC (1 heap besar) |

---

## 1. Erlang

- Bahasa asli untuk BEAM VM (Ericsson, 1986).
- Sintaks agak tua (mirip Prolog), cukup sulit buat pemula.
- Kuat di telekomunikasi, distribusi, fault tolerance.
- Banyak library / tool bawaan (OTP, gen_server, supervisor).
- Digunakan di dunia nyata: WhatsApp, RabbitMQ, CouchDB.

**Erlang = "bahasa klasik" untuk sistem real-time, sangat battle-tested, tapi sintaks agak old-school.**

---

## 2. Elixir

- Dibuat 2012 oleh José Valim.
- Compile ke BEAM VM, dapet semua power Erlang (concurrency, fault-tolerance).
- Sintaks modern, mirip Ruby (lebih ramah developer).
- Ekosistem: Phoenix Framework (mirip Rails/Express untuk Elixir).
- Fokus ke web dev, produktivitas, readability.

**Elixir = "Erlang modern" → lebih enak nulisnya, lebih populer di web dev.**

---

## 3. Gleam

- Masih muda (2019+).
- Compile ke BEAM (kayak Erlang/Elixir), atau ke JavaScript.
- Strong static typing (mirip Rust/TypeScript/Elm) → lebih aman dari bug, compiler cek semua.
- Sintaks simpel, mirip Elm / Rust.
- Ekosistem masih kecil, tapi tumbuh cepat.
- Fokus: developer experience yang aman (type-safe) + nyaman.

**Gleam = "Erlang yang strongly typed + syntax modern" → aman kayak Rust, jalan di BEAM/JS.**

---

## Hello World

### Erlang

```erlang
-module(hello).
-export([main/0]).

main() ->
    io:format("Hello, world!~n").
```

➡️ Sintaks mirip Prolog, banyak tanda baca (-module, . di akhir, dsb).

### Elixir

```elixir
defmodule Hello do
  def main do
    IO.puts("Hello, world!")
  end
end
```

➡️ Lebih manis, mirip Ruby. defmodule, def, kurung kurawal opsional.

### Gleam

```gleam
import gleam/io

pub fn main() {
  io.println("Hello, world!")
}
```

➡️ Sangat sederhana, mirip Rust / Elm. Compiler cek tipe otomatis.

---

## Fungsi Tambah (typed vs dynamic)

### Erlang

```erlang
add(A, B) ->
    A + B.
```

➡️ Dynamic, tidak ada type annotation, bisa runtime error kalau salah tipe.

### Elixir

```elixir
def add(a, b) do
  a + b
end
```

➡️ Sama, dynamic typing. Kalau add(1, "x") → crash runtime.

### Gleam

```gleam
pub fn add(a: Int, b: Int) -> Int {
  a + b
}
```

➡️ Static typing ✅ → kalau salah (misalnya add(1, "x")), langsung error waktu compile.