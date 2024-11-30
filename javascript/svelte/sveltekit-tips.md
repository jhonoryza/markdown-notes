---
title: 'Sveltekit Tips'
date: '2023-08-11 14:35:00'
---

Beberapa tips sveltekit

- gunakan `+server.js` ketika akan membuat api atau melakukan http request ke service lain
- server side render gunakan `+server.js` atau `+page.server.js`
- client side render gunakan `+page.svelete` atau `+page.js`

## Template Project

```bash
pnpm create svelete@latest app
cd app
pnpm install
pnpx svelte-add tailwindcss
pnpm install
git init && git add -A && git commit -m "Initial commit" #optional
pnpm run dev
```
