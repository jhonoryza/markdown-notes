install shadcn/vue using this command

```bash
npx shadcn-vue@latest init
```

or install vscode shadcn/vue plugin then run init command

when above command is running there will be a prompt question like this

- Would you like to use TypeScript? (recommended)? no / yes
- Which framework are you using? › Vite
- Which style would you like to use? › Default
- Which color would you like to use as base color? › Slate
- Where is your tsconfig.json file? `./inertia/tsconfig.json`
- Where is your global CSS file? (this file will be overwritten)
  `./inertia/css/app.css`
- Would you like to use CSS variables for colors? no / yes
- Are you using a custom tailwind prefix eg. tw-? (Leave blank if not)
- Where is your tailwind.config located? (this file will be overwritten)
  tailwind.config.js
- Configure the import alias for components: `~/components`
- Configure the import alias for utils: `~/lib/utils`
- Write configuration to components.json. Proceed? yes

`follow the highligted one for adonisjs compatibility`

adjust `tailwind.config.js`

```js
 content: [
   './inertia/pages/**/*.{ts,tsx,vue}',
   './inertia/components/**/*.{ts,tsx,vue}',
   './inertia/app/**/*.{ts,tsx,vue}',
   './inertia/src/**/*.{ts,tsx,vue}',
],
```

install depedency package

```bash
npm i -D tailwindcss-animate
```

register component shadcn/vue as global

```bash
npm i -D unplugin-vue-components
```

edit `vite.config.js`

```ts
import Components from "unplugin-vue-components/vite";
plugins: [
    Components({
        dirs: ["inertia/components"],
        dts: true,
    }),
];
```

install specific shadcn component that you need using `cmd+shift+p` then search
`add new component` enter it then search component like input, button or label

change `home.vue` like this

```vue
<script setup lang="ts">
</script>

<template>

  <Head title="Homepage" />

  <form class="grid gap-4">
    <div>
      <Label>Email</Label>
      <Input type="email" />
    </div>
    <div>
      <Label>Password</Label>
      <Input type="password" />
    </div>
    <Button>Login</Button>
  </form>
</template>
```
