# Apa itu Elm?

Elm adalah bahasa fungsional untuk frontend (web browser).

- Dirancang untuk bikin UI web tanpa bug runtime.
- Strong static typing → seperti Rust/TypeScript, tapi pure functional.
- Compiler Elm terkenal ramah: pesan error sangat jelas.
- Konsep utama: The Elm Architecture (TEA) → inspirasi untuk Redux, React Hooks, dan framework seperti Lustre.
- Syntax Elm mirip Haskell, tapi lebih sederhana.

## Contoh Elm

```elm
module Main exposing (..)

import Html exposing (div, text)

main =
  div [] [ text "Hello, Elm!" ]
```

👉 Di browser, Elm compile jadi JavaScript.