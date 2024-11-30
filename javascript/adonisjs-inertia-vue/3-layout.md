lets create `inertia/layouts/AppLayout.vue`

```vue
<script setup lang="ts">
</script>

<template>
    <header>
        <nav class="flex justify-between py-2 px-4 shadow text-xl">
            <div>
                <Link href="/">Home</Link>
            </div>
            <div class="flex gap-4">
                <Link href="/login">Login</Link>
                <Link href="#">Register</Link>
            </div>
        </nav>
    </header>
    <main class="mx-auto container">
        <slot />
    </main>
</template>
```

use above layout as global layout here

```ts
// for spa
    resolve: async (name) => {
        const page = await resolvePageComponent(
            `../pages/${name}.vue`,
            import.meta.glob<DefineComponent>("../pages/**/*.vue"),
        );
        page.default.layout = page.default.layout || AppLayout;
        return page;
    },

// for ssr
        resolve: (name) => {
            const pages = import.meta.glob<DefineComponent>(
                "../pages/**/*.vue",
                { eager: true },
            );
            const resolvedPage = pages[`../pages/${name}.vue`];
            resolvedPage.default.layout = resolvedPage.default.layout ||
                AppLayout;
            return resolvedPage;
        },
```

try it lets change `home.vue` page

```vue
<script setup>
</script>

<template>
    <Head title="Homepage" />
</template>
```
