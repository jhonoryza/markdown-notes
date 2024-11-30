```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

in tailwind.config.js

```js
 darkMode: ["class"],
 content: [
   './inertia/**/*.{ts,tsx,vue}',
],
```

edit edge base template file in `resources/views/inertia_layout.edge`

edit `inertia/css/app.css` and add this code

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

edit `inertia/app/app.ts`

```ts
import { Link, Head } from '@inertiajs/vue3'

setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .component('Link', Link)
      .component('Head', Head)
      .mount(el)
  },
```

to test edit `home.vue`

```vue
<script setup lang="ts">
import { Head } from '@inertiajs/vue3'
</script>

<template>

  <Head title="Homepage" />

  <p class="text-red-500">test</p>
</template>
```
