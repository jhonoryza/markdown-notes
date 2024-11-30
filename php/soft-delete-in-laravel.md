## Preparing the model and database

To get started with using soft deletes in your Laravel application, you'll need
to prepare your soon-to-be soft-deletable model and its database table.

Laravel provides a handy `Illuminate\Database\Eloquent\SoftDeletes` trait that
you can add to any model that you wish to make soft-deletable. This trait adds
some useful methods to the model, such as the following:

- forceDelete - This method permanently deletes the model from the database.
- forceDeleteQuietly - This method permanently deletes the model from the
  database without firing any model events.
- restore - This method restores the soft-deleted model in the database.
- restoreQuietly - This method restores the soft-deleted model in the database
  without firing any model events.
- trashed - This method returns a Boolean to indicate whether a given model has
  been soft-deleted.

It also registers some model events triggered when the model is being
soft-deleted, force deleted, and restored.

Another huge benefit of using this trait is that it registers a global scope
(using the `Illuminate\Database\Eloquent\SoftDeletingScope` class that's
included) that automatically excludes any soft-deleted models from your queries.
This class also registers some query macros that you can use to include
soft-deleted models in your queries. We'll go into more depth about these query
macros later in this article.

Now that we have some context about the SoftDeletes trait, let's add it to our
model. For the purposes of this article, we'll imagine we are building a
blogging application and have an Author model that we wish to make
soft-deletable. We'll add the SoftDeletes trait to the model like so:

```php
app/Models/User.php
declare(strict_types=1);
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
 
final class Author extends Model
{
    use SoftDeletes;
 
    // ...
}
```

Now that we've added the soft delete functionality to the model, we need to
prepare the database table so that we can keep track of whether a model has been
soft-deleted. As we've already mentioned, Laravel does this using a deleted_at
timestamp column. Thus, let's add this column to our authors table using a
migration.

We'll create a new migration by running the following Artisan command:

```bash
php artisan make:migration add_deleted_at_to_authors_table --table=authors
```

This command will create a new migration class with a file path, such as
`database/migrations/2023_07_10_111209_add_deleted_at_to_authors_table.php`.
Let's update the migration so that we can add the new deleted_at column. To do
this, we can use a handy softDeletes method that's available on the Blueprint
class provided by Laravel. Likewise, we can use the dropSoftDeletes method to
remove the column.

The migration should look something like this:

```php
declare(strict_types=1);
 
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
 
return new class extends Migration
{
    public function up()
    {
        Schema::table('authors', function (Blueprint $table) {
            $table->softDeletes();
        });
    }
 
    public function down()
    {
        Schema::table('authors', function (Blueprint $table) {
            $table->dropSoftDeletes();
        });
    }
};
```

We can now run the migration using the migrate Artisan command to add the
deleted_at column to the authors table:

```bash
php artisan migrate
```

## Soft deleting and restoring models

Your model and database should now be ready to use soft deletes. Let's take a
look at how we can soft-delete and restore models.

To soft-delete a model, you can call the delete method on the model instance.
Since the model is using the SoftDeletes trait, the delete method will
automatically set the deleted_at column to the current date and time rather
actually remove the row from the database. As an example, to soft delete an
Author model (which we'll assume has an ID of 1), we can do the following:

```php
$author = Author::find(1);
 
$author->delete();
```

The Author model will now be soft-deleted. It's worth noting that calling delete
on an already soft-deleted model will update the model's deleted_at column to
the current date and time. Although this may not cause any issues for your
application, it's worth keeping in mind if you need to keep a record of the
original time that a model soft-deleted, such as for analytical purposes.

If we wanted to restore the model (so that it's no longer soft-deleted), we can
call the restore method on the model instance:

```php
$author->restore();
```

If you'd like to check whether a given model is soft-deleted, you can use the
trashed method on the model instance. If the model is soft-deleted, it will
return true; otherwise, it will return false. Here is an example:

```php
$author = Author::find(1);
 
$author->trashed(); // false
 
$author->delete();
 
$author->trashed(); // true
```

### Deleting models permanently

If you want to permanently remove a model from the database (sometimes referred
to as "hard-deleting"), you can use the forceDelete method on the model. This
can be performed on both soft-deleted and non-soft-deleted models like so:

```php
$author = Author::find(1);
 
$author->forceDelete();
```

It's important to remember that once this method has been called, the row will
be permanently removed from the database and cannot be restored.

### Querying soft-deletable models

As we've already briefly mentioned, the SoftDeletes trait registers a global
scope that automatically excludes any soft-deleted models from your queries by
default. This means that if you run queries, such as `Author::find(1)`,
`Author::all()`, or `Author::where('name', 'John')->get()`,soft-deleted models
will not be returned.

In most cases, this is the desired behavior. However, there may be times when
you want to include soft-deleted models in your query or only want to return
soft-deleted models. Laravel registers some query macros that you can use to
achieve this. Let's take a look at them.

### Including soft-deleted models

There may be times when you want to retrieve all the models in the database
table, regardless of whether they're soft-deleted. An example of when you might
want to do this would be if you're writing a query to analyze the usage
statistics of a user's account.

To include soft-deleted models in your query, you can use the withTrashed method
on the query builder instance. For example, if you want to retrieve all the
Author models, including the soft-deleted models, you could write the following
query:

```php
$authors = Author::withTrashed()->get();
```

Likewise, if you wanted to retrieve a single model that may or may not be
soft-deleted, you could use the withTrashed method like so:

```php
$author = Author::withTrashed()->find(1);
```

### Only soft-deleted models

There may be times when you only want to fetch the soft-deleted models from the
database and exclude any non-soft-deleted models. For example, you might want to
do this if you're building a "recycle bin" feature for your application that
allows users to restore or permanently delete soft-deleted models.

To only retrieve soft-deleted models, you can use the onlyTrashed method on the
query builder instance. For example, if you wanted to retrieve only the
soft-deleted models, you could write the following query:

```php
$authors = Author::onlyTrashed()->get();
```

Likewise, if you wanted to retrieve a single soft-deleted model, you could use
the onlyTrashed method like so:

```php
$author = Author::onlyTrashed()->find(1);
```

### Using the DB facade

So far, our code examples have revolved around using functionality provided by
the SoftDeletes trait to query our soft-deleted models. However, there may be
times when you want to write a query using the DB facade. For example, if you're
building an in-depth report for your application using a more complex query, you
may want to do this.

If you do choose to query a soft-deletable model using the DB facade, there's a
common "gotcha" that you need to be aware of. I've seen this affect many
developers, including myself, so it's worth mentioning here.

The SoftDeletes trait registers the SoftDeletingScope query scope on the
soft-deletable model, which means it can be applied to the
`Illuminate\Database\Eloquent\Builder` object used when building an Eloquent
query (as shown above). However, when you use the DB facade, you're not using
the `Illuminate\Database\Eloquent\Builder` object; instead, you're using an
`Illuminate\Database\Query\Builder` object.

Thus, the SoftDeletingScope is not applied to the query, and if you write a
query using the DB facade, soft-deleted models will be returned in the results.
This means you'll need to make sure to manually exclude soft-deleted models from
your query.

Let's take a look at a basic example of this. Imagine we have the following
query using the DB facade to get all the authors:

```php
$authors = DB::table('authors')->get();
```

If we ran the above code, the soft-deleted and non-soft-deleted models would all
be fetched from the database. If we wanted to update the query to exclude
soft-deleted models, we could do the following:

```php
$authors = DB::table('authors')
    ->whereNull('deleted_at')
    ->get();
```

As you can see in the code example, we've used the whereNull method to exclude
any soft-deleted models from the query.

It's important that you remember to add this when building queries. A great way
of checking that you've written queries correctly is to ensure that you have a
good-quality test to assert that no soft-deleted models are returned in the
query. We'll cover this in more depth later in the article.

### Soft-deleted models and route model binding

In Laravel, you can use "route model binding" when defining routes to
automatically inject a model instance into your controller method. This makes it
easy to fetch a model instance from the database and have it ready for use
directly in your controller. For example, if you wanted to fetch an Author model
instance from the database and have it available in your controller, you could
define the following route:

```php
Route::get('/authors/{author}', [AuthorController::class, 'show']);
```

In the show method of the AuthorController, we can then type-hint the Author
model and have it injected into the method like so:

```php
public function show(Author $author)
{
    // ...
}
```

If a user now navigated to `/authors/1`, the $author variable would contain the
Author model instance for the author with an ID of 1. This is a really handy
feature!

However, by default, route model binding ignores soft-deleted models. This means
that if you were to navigate to `/authors/1`, but the author with an ID of 1 was
soft-deleted, you would receive an HTTP 404 response.

In most cases, this is likely the desired behavior. However, there may be times
when you'd like to include soft-deleted models in the binding. For example, you
may want to build an admin panel that allows you to view and restore
soft-deleted models. To do this, you can use the withTrashed method on the route
model binding definition. For example, we could update the route definition to
the following:

```php
Route::get('/authors/{author}', [AuthorController::class, 'show'])->withTrashed();
```

It's important to note that this won't use only soft-deleted models in the
bindings. Instead, it will include both soft-deleted and non-soft-deleted.
Therefore, if you'd only like to include soft-deleted models, you'll need to add
a check to your controller to ensure that the model is soft-deleted. For
example, we could update the show method in the AuthorController to the
following:

```php
public function show(Author $author)
{
    // If the author is not soft-deleted, return a 404 response.
    if (!$author->trashed()) {
        abort(404);
    }
 
    // Continue as normal...
}
```

### Testing soft deletes

Like any other part of your application, it's important that you remember to
test your code. It can help to give you confidence that your code runs as
expected and can reduce the chance of bugs being introduced when you make
changes in the future.

Thanks to some helpful methods provided by Laravel, we can easily test our
soft-deleted models. Let's take a look at some common tests you might want to
write for your soft-deleted models and tips for writing them.

Assert whether a model is soft-deleted A common assertion that you'll likely
want to make when testing soft-deleted models is testing whether a model is
soft-deleted. There are multiple ways that you can achieve this in Laravel.

First, as we've already covered, you can use the trashed method on the model to
check whether the model is soft-deleted. For example, let's imagine we have a
route and controller (accessible by DELETE /authors/{author} using the route
name authors.destroy) that allows a user to soft-delete a model. We could write
the following test to assert that the author is soft-deleted:

```php
use App\Models\Author;
use PHPUnit\Framework\Attributes\Test;
use Tests\TestCase;
 
#[Test]
public function author_can_be_soft_deleted(): void
{
    $author = Author::factory()->create();
 
    $this->delete(route('authors.destroy', $author))
        ->assertOk();
 
    $this->assertTrue($author->fresh()->trashed());
}
```

Alternatively, we could use the assertSoftDeleted helper method provided by
Laravel. We could rewrite the above test to use this method like so:

```php
use App\Models\Author;
use PHPUnit\Framework\Attributes\Test;
use Tests\TestCase;
 
#[Test]
public function author_can_be_soft_deleted(): void
{
    $author = Author::factory()->create();
 
    $this->delete(route('authors.destroy', $author));
        ->assertOk();
 
    $this->assertSoftDeleted($author);
}
```

Similar to our two approaches above, if we wanted to assert that a model is not
soft-deleted, we could use the assertNotSoftDeleted helper method provided by
Laravel. For example, we might have a route and controller (accessible by POST
`/authors/{author}/restore` using the route name `authors.restore`) that we can
use to restore soft-deleted authors. If we wanted to write a test to assert that
a model is not soft-deleted, we could write the following:

```php
use App\Models\Author;
use PHPUnit\Framework\Attributes\Test;
use Tests\TestCase;
 
#[Test]
public function author_can_be_restored(): void
{
    $author = Author::factory()
        ->trashed()
        ->create();
 
    $this->post(route('authors.restore', $author))
        ->assertOk();
 
    $this->assertNotSoftDeleted($author);
}
```

You may have noticed in the test above that we called a trashed method on the
Author model's factory. This method is provided by Laravel and allows us to
create soft-deleted models. This can be really useful when you want to quickly
create a soft-deleted model in your tests.

If you'd like to test that a model has been permanently deleted from the
database, you can use the assertModelMissing helper method provided by Laravel.
For example, if we had a route and controller (accessible by DELETE
`/authors/{author}/force` using the route name `authors.force`) that we could
use to permanently delete a soft-deleted model, we could write the following
test to assert that the model has been permanently deleted:

```php
use App\Models\Author;
use PHPUnit\Framework\Attributes\Test;
use Tests\TestCase;
 
#[Test]
public function soft_author_can_be_permanently_deleted(): void
{
    $author = Author::factory()
        ->trashed()
        ->create();
 
    $this->delete(route('authors.force', $author))
        ->assertOk();
 
    $this->assertModelMissing($author);
}
```

If the route only allows soft-deleted models to be permanently deleted and
returns an HTTP 404 response if we attempt to permanently delete a
non-soft-deleted model, we could use the assertModelExists to write the
following test to assert that the model is not permanently deleted:

```php
use App\Models\Author;
use PHPUnit\Framework\Attributes\Test;
use Tests\TestCase;
 
#[Test]
public function non_soft_author_cannot_be_permanently_deleted(): void
{
    $author = Author::factory()->create();
 
    $this->delete(route('authors.force', $author))
        ->assertNotFound();
 
    $this->assertModelExists($author);
}
```

Assert that soft-deleted models are included or excluded from queries You'll
likely want to write tests that assert that soft-deleted models are included or
excluded from your queries. These types of tests can provide peace of mind and
give you confidence that you're not accidentally including soft-deleted models
in your queries.

Let's take the following basic controller method as an example that returns a
view containing all the authors (excluding soft-deleted authors):

```php
class AuthorController extends Controller
{
    public function index()
    {
        return view('authors.index', [
            'authors' => Author::all(),
        ]);
    }
}
```

To ensure that we're not accidentally including soft-deleted authors in our
query, we could write the following test:

```php
use App\Models\Author;
use Illuminate\Database\Eloquent\Collection;
use PHPUnit\Framework\Attributes\Test;
use Tests\TestCase;
 
#[Test]
public function authors_view_is_returned(): void
{
    // Create two non-soft-deleted authors. They should be
    // included in the results.
    $authors = Author::factory()
        ->count(2)
        ->create();
 
    // Create a soft-deleted author. They should not be
    // included in the results.
    Author::factory()
        ->trashed()
        ->create();
 
    $this->get(route('authors.index'))
        ->assertOk()
        ->assertViewIs('authors.index')
        ->assertViewHas(
            key: 'authors',
            value: fn (Collection $authors): bool =>
                $authors->pluck('id')->toArray() === [
                    $authors[0]->id,
                    $authors[1]->id,
                ]
        );
}
```

As we can see in the test, we're explicitly checking that the Collection being
passed to the view only includes the two non-soft-deleted authors. By doing
this, we can be sure that we're not accidentally including soft-deleted authors
in our query.

In fact, this general approach of creating a model that doesn't satisfy one of
the query constraints is quite handy when testing any type of query. You can use
it across your all application's tests to ensure that your query constraints are
working as expected and that you're only returning the models that you expect to
return. This can be particularly useful when testing complex queries with many
conditional constraints.
