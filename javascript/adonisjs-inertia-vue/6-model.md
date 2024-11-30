```bash
node ace make:model post
node ace make:model category
```

post model add this column

```ts
    @column()
    declare title: string;

    @column()
    declare summary?: string;

    @column()
    declare slug: string;

    @column()
    declare content: string;

    @column()
    declare imageUrl: string;

    @column()
    declare isPublished: boolean;

    @column()
    declare publishedAt: DateTime;
```

category model add this column

```ts
    @column()
    declare name: string;

    @column()
    declare slug: string;
```
