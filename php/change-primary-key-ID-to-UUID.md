## Step 1. Migration for users.

The main migration for the users table: change ->id() to uuid('id')->primary().

```php
database/migrations/2014_10_12_000000_create_users_table.php:

Schema::create('users', function (Blueprint $table) {    
    $table->id();     
    $table->uuid('id')->primary(); 
});
```

Two things to notice:

- The default ->uuid() would create a field uuid, so if you want to stick to the
  id, you need to specify it.
- Also, default uuid() doesn't make that field a primary key, so you need to
  specify ->primary().
- Step 2. Other Migrations to users.
- All the migrations with foreign keys to the users table should change to
  foreignUuid().

```php
$table->foreignId('user_id')->constrained(); 
$table->foreignUuid('user_id')->constrained();
```

Also, polymorphic relations for the default personal_access_tokens table:

```php
Schema::create('personal_access_tokens', function (Blueprint $table) {    
    $table->morphs('tokenable');     
    $table->uuidMorphs('tokenable'); 
});
```

Step 3. User Model: Add Trait Finally, add a trait HasUuids to the User model.

```php
app/Models/User.php:

use Illuminate\Database\Eloquent\Concerns\HasUuids; 
class User extends Model {    
   use HasUuids;
}
```

This will auto-generate UUIDs for every new User record.

This HasUuids feature appeared in the Laravel 9.30 version. Here's my video
about it.

And that's it. Every new User would get a UUID automatically, like
`"8f8e8478-9035-4d23-b9a7-62f4d2612ce5"`.

Trick to update column

```sql
UPDATE
    <tablename>
SET
    `uuid` = MD5(CONCAT(UUID(), `id`))
```
