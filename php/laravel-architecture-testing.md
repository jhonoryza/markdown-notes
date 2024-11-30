## What is Architecture Testing?

Architecture testing differs from the traditional tests you might normally write
for your projects. Whereas your traditional tests test the functionality of your
application, architecture tests test the structure of your application. They
allow you to enforce standards and consistency in your projects.

Depending on the type of project you're working on, you may have a set of
standards or guidelines set by your employer, client, or yourself. For example,
you might want to enforce things such as:

- Strict type-checking is used in all files.
- The app/Interfaces directory only contains interfaces.
- All classes in the app/Services directory are suffixed with Service
  `(e.g., UserService, BillingService, etc.)`.
- All models in the app/Models directory extend the
  `Illuminate\Database\Eloquent\Model class.`
- All action classes in the app/Actions directory are invokable (meaning they
  have an `__invoke method`).
- Controllers aren't used anywhere apart from the routes files in the
  application.
- Only traits exist in the app/Traits directory.
- All classes inside the app/DataTransferObjects directory are readonly.

Generally, you should enforce these things manually via code reviews and other
similar processes. With architecture testing, you can enforce these standards
automatically and know for sure that they're followed.

For example, take this simple test that enforces that all interfaces exist
within the App\Interfaces namespace:

```php
test('interfaces namespace only contains interfaces')
    ->expect('App\Interfaces')
    ->toBeInterfaces();
```

The above test can be run like any other Pest test. The test will pass if all
the files in the App\Interfaces namespace are interfaces. However, the test will
fail if any of the files in this namespace aren't interfaces.

## Benefits of Architecture Testing

Now that we have a basic understanding of architecture testing let's look at its
benefits in your projects.

### Reduce Code Review Time

As we've already mentioned, architecture testing is an excellent way of
automating the process of enforcing standards in your codebase.

### Improve Code Quality and Consistency

Enforcing standards and consistency early on in a project can help to improve
the overall code quality. It helps implement a sensible set of rules that
developers can follow to ensure they write code that other developers can
understand.

Every developer has their own way of writing code. I'm sure you'll agree that
once you've worked on a project for a long enough time, you can usually tell who
wrote a particular piece of code by the variable and method names or how the
classes are structured. Minor differences like these aren't typically a big deal
as long as they follow the general standards of the project that the other
developers are also following.

However, the codebase may have some alarming differences if you don't enforce
standards. For example, you might have one developer who prefers to keep their
interfaces in an app/Interfaces directory. However, you might have another
developer who prefers to keep the interfaces in a folder related to the class.
For instance, they might put a UserServiceInterface interface inside an
app/Services/User directory so it lives alongside the UserService class instead
of placing it in the app/Interfaces directory.

These types of differences can cause consistency issues and make it difficult
for other developers to understand and work on the codebase. Although
inconsistencies such as these should have been caught during a manual code
review process, they may not have been. However, if you want to enforce that
there are no interfaces outside of the app/Interfaces directory, you could write
some tests like this:

```php
test('services namespace only includes classes')
    ->expect('App\Services')
    ->toBeClasses();
 
test('interfaces namespace only contains interfaces')
    ->expect('App\Interfaces')
    ->toBeInterfaces();
```

Now, if there's an interface in the App\Services namespace, the test will fail
and must be fixed before merging the code into the main branch. As a result, it
can help ensure that every developer uses the same approach.

## Get Started with Architecture Testing

Now that we've learned about architecture testing and its benefits in your
projects, we'll learn how to install Pest and write our first architecture
tests.

You'll need to install Pest in your Laravel project using Composer. You can do
this by running the following command in your project's root directory:

```bash
composer require pestphp/pest-plugin-laravel --dev
```

Pest should now be installed and ready to run.

Next, we'll look at typical examples of architecture tests you might want to add
to your Laravel applications. We'll begin by creating a simple test that asserts
that the `App\Models` namespace contains only classes and that each class
extends the Illuminate\Database\Eloquent\Model class.

Where you place your architecture tests is a personal preference, but I like to
put mine in a `tests/Architecture` directory. Doing this allows me to keep my
architecture tests separate from my feature and unit tests. For our tests inside
that directory to be detected and run, we'll need to update the phpunit.xml file
to include it. We can do this by adding a new Architecture testsuite to the
`<testsuites>` section:

phpunit.xml

```xml
<testsuites>
    ...
    <testsuite name="Architecture">
        <directory>tests/Architecture</directory>
    </testsuite>
</testsuites>
```

If you've not previously made any changes to the default phpunit.xml file that
ships with Laravel, this will result in the file looking something like this:

phpunit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
>
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
        <testsuite name="Architecture">
            <directory>tests/Architecture</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>app</directory>
        </include>
    </source>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="BCRYPT_ROUNDS" value="4"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="MAIL_MAILER" value="array"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="TELESCOPE_ENABLED" value="false"/>
    </php>
</phpunit>
```

I like to structure my architecture tests in a way that's similar to my
application code's directory structure. By doing this, it makes it easier to
find the tests that are related to a particular part of the project. For
example, I place my architecture tests for models (found in `app/Models`) in a
`tests/Architecture/ModelsTest.php` test file. I put my controller architecture
tests (in `app/Http/Controllers`) in a
`tests/Architecture/Http/ControllersTest.php` test file. And so on.

So we will create a `tests/Architecture/ModelsTest.php` file and add the
following code to it:

```php
tests/Architecture/ModelsTest.php
use Illuminate\Database\Eloquent\Model;
 
test('models extends base model')
    ->expect('App\Models')
    ->toExtend(Model::class);
```

We can now run this test using the following command:

```bash
php artisan test
```

If you configured everything correctly, you should see an output like this when
you run the test:

```bash
PASS  Tests\Architecture\ModelsTest
✓ models extends base model                                                      0.06s
Tests:    1 passed (2 assertions)
Duration: 0.14s
```

We'll purposely break the tests to ensure your test is working correctly and
detecting any architectural issues. To do this, let's add an empty PHP class
that doesn't extend the `Illuminate\Database\Eloquent\Model` class. The class
may look something like this:

```php
app/Models/UserService.php
namespace App\Models;
 
class UserService
{
 
}
```

If you re-run the tests, the tests should fail, and you will get an output that
looks like this:

```bash
FAIL  Tests\Architecture\ModelsTest
⨯ models extends base model                                                      0.07s
──────────────────────────────────────────────────────────----------------------------
FAILED  Tests\Architecture\ModelsTest > models          ArchExpectationFailedException
Expecting 'app/Models/UserService.php' to extend 'Illuminate\Database\Eloquent\Model'.
at app/Models/UserService.php:5
    1▕
    2▕
    3▕ namespace App\Models;
    4▕
➜   5▕ class UserService
    6▕ {
    7▕
    8▕ }
    9▕
1   app/Models/UserService.php:5
Tests:    1 failed (2 assertions)
Duration: 0.18s
```

## Common Examples of Architecture Tests

Now that we've looked at how to install Pest and write our first architecture
tests let's look at some common examples of architecture tests you might want to
add to your Laravel applications.

Here are some of the common methods that you'll likely be using in your tests:

- toBeAbstract - Assert the classes in the given namespace are abstract (using
  the keyword abstract).
- toBeClasses - Assert the files in the given namespace are PHP classes.
- toBeEnums - Assert the files in the given namespace are PHP enums.
- toBeInterfaces - Assert the files in the given namespace are PHP interfaces.
- toBeTraits - Assert the files in the given namespace are PHP traits.
- toBeInvokable - Assert the classes in the given namespace are invokable
  (meaning they have an __invoke method).
- toBeFinal - Assert the classes in the given namespace are final (using the
  keyword final).
- toBeReadonly - Assert the classes in the given namespace are readonly (using
  the keyword readonly).
- toExtend - Assert the classes in the given namespace extend the given class.
- toImplement - Assert the classes in the given namespace implement the given
  interface.
- toHaveMethod - Assert the classes in the given namespace have the given
  method.
- toHavePrefix - Assert the classes in the given namespace start with the given
  string in their classnames.
- toHaveSuffix - Assert the classes in the given namespace end with the given
  string in their classnames.
- toUseStrictTypes - Assert the files in the given namespace use strict types
  (using the keywords declare(strict_types=1)).

You can check out the Pest documentation for a complete list of available
assertions.

Now, let's look at some common tests you may want to write. This is not an
exhaustive list, but it should provide an insight into the different assertions
you might want to use. You can also use these as a starting point for your own
tests and modify them to suit your needs.

## Tests for Commands

You may want to write a test for your Artisan commands to assert that the
App\Console\Commands namespace only contains valid command classes. To do this,
you may want to write a test like this:

```php
declare(strict_types=1);
 
use Illuminate\Console\Command;
 
test('commands')
    ->expect('App\Console\Commands')
    ->toBeClasses()
    ->toUseStrictTypes()
    ->toExtend(Command::class)
    ->toHaveMethod('handle');
```

In this test, we're defining that we only expect classes in the
`App\Console\Commands` namespace. We're then asserting that each class uses
strict types (using declare(strict_types=1)), extends the
`Illuminate\Console\Command` class, and has a handle method.

If we were to add any files in this namespace that don't meet these requirements
(such as adding a class that doesn't extend the `Illuminate\Console\Command`
class), the test would fail.

Tests for controllers You may also want to write a test to assert that your
controllers follow specific standards. Let's take a look at an example test you
could write, and then we'll discuss what we're doing:

```php
declare(strict_types=1);
 
use Illuminate\Routing\Controller;
 
test('controllers')
    ->expect('App\Http\Controllers')
    ->toBeClasses()
    ->toUseStrictTypes()
    ->toBeFinal()
    ->classes()
    ->toExtend(Controller::class);
```

In the test above, we assert that every PHP file in the App\Http\Controllers
namespace is a class. We then assert that each of the classes uses strict types
(using `declare(strict_types=1)`) and are final (using the final keyword).
Finally, we assert that each class extends the `Illuminate\Routing\Controller`
class.

If we were to add any other classes in this namespace that don't meet these
requirements (such as adding a class that doesn't extend the
`Illuminate\Routing\Controller` class), the test would fail.

Tests for Interfaces You may also want to write a test to assert that your
project's App\Interfaces namespace only contains interfaces. To do this, you may
want to write a test like this:

```php
declare(strict_types=1);
 
test('interfaces')
    ->expect('App\Interfaces')
    ->toBeInterfaces();
```

If we were to add any files in this namespace that don't meet these requirements
(such as adding a class or trait), the test would fail.

## Tests for Global Functions

As well as testing that PHP files follow specific standards, we can also test
that we don't use certain PHP global functions in our codebase. This can be a
handy way to ensure that we don't accidentally leave debugging statements (such
as dd()) in the code, which could cause issues in production.

You may want to write a test like this:

```php
declare(strict_types=1);
 
test('globals')
    ->expect(['dd', 'ddd', 'die', 'dump', 'ray', 'sleep'])
    ->toBeUsedInNothing();
```

In the test above, we assert that the functions dd, ddd, die, dump, ray, and
sleep aren't used anywhere in the application. If it finds one of them, the test
will fail.

## Tests for Jobs

Another helpful place you may want to write some architecture tests for is your
job classes. You may wish to assert that all of your job classes implement the
`Illuminate\Contracts\Queue\ShouldQueue` interface and have a handle method. To
do this, you may want to write a test like this:

```php
declare(strict_types=1);
 
use Illuminate\Contracts\Queue\ShouldQueue;
 
test('jobs')
    ->expect('App\Jobs')
    ->toBeClasses()
    ->toImplement(ShouldQueue::class)
    ->toHaveMethod('handle');
```

In the test above, we assert that the `App\Jobs` namespace only includes classes
that implement the `Illuminate\Contracts\Queue\ShouldQueue` interface and have a
handle method. If we were to add an invalid file (such as a class that didn't
implement the interface), the test would fail.

## Tests for Models

Let's look at one of the modifiers that Pest allows us to use to make our
assertions more specific. We'll take a look at the ignoring modifier.

Let's imagine that we have an app/Models directory to demonstrate how we might
use the 'ignoring' modifier. Within this directory, we have a Traits directory
(using the `App\Models\Traits` namespace). Imagine that this directory contains
traits used by our models and nowhere else in the codebase.

We may want to write three tests that look like this:

```php
test('models')
    ->expect('App\Models')
    ->toBeClasses()
    ->ignoring('App\Models\Traits');
 
test('models extends base model')
    ->expect('App\Models')
    ->toExtend(Model::class)
    ->ignoring('App\Models\Traits');
 
test('model traits')
    ->expect('App\Models\Traits')
    ->toBeTraits()
    ->toOnlyBeUsedIn('App\Models');
```

In the first test (named models), we assert that every PHP file within the
`App\Models` namespace is a class. We then use the ignoring modifier to ignore
the `App\Models\Traits` namespace. That means that this test will ignore any
files in this namespace.

In the second test (named models extends base model), we're asserting that every
class (except those in the `App\Models\Traits` namespace) extends the
`Illuminate\Database\Eloquent\Model` class, thus making it a valid model class.

Finally, in the third test (named model traits), we assert that every PHP file
within the `App\Models\Traits` namespace is a trait. We then assert that the
traits found in this namespace do not live outside of the `App\Models`
namespace. This means that the test would fail if we used any of these traits in
another part of the application (such as in a controller)
