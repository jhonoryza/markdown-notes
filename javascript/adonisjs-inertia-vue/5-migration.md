```bash
node ace make:migration posts
node ace make:migration categories
node ace make:migration post_categories
```

adjust user migration change id using bigIncrements

```ts
table.bigIncrements("id").notNullable();
```

posts migration add this column

```ts
table.bigIncrements("id");
table
    .bigInteger("author_id")
    .unsigned()
    .nullable()
    .references("users.id")
    .onDelete("SET NULL");

table.string("title").notNullable();
table.string("slug").notNullable();
table.string("summary").nullable();
table.text("content").notNullable();

table.string("image_url").notNullable();

table.boolean("is_published").notNullable().defaultTo(false);
table.date("published_at").nullable();
```

categories migration add this column

```ts
table.bigIncrements("id").notNullable();

table.string("name").notNullable();
table.string("slug").notNullable();
```

post_categories migration add only this column and remove another column

```ts
table
    .bigInteger("post_id")
    .unsigned()
    .notNullable()
    .references("posts.id")
    .onDelete("CASCADE");
table
    .bigInteger("category_id")
    .unsigned()
    .notNullable()
    .references("categories.id")
    .onDelete("CASCADE");
```

run `node ace migration:run`
