The Pipeline Design Pattern is where data is passed through a sequences of tasks
or stages. The pipeline acts like an assembly-line, where the data is processed
and then passed on to the next stage.

Using a pipeline is advantageous because it is really easy to compose a complex
process as individual tasks. It also makes it easy to add, remove, or replace
stages within the pipeline without disturbing the entire process.

Laravel uses the Pipeline Design Pattern in a couple of places throughout the
framework. This means everything we need to implement this pattern is already
part of the foundation of your application!

## What is the Pipeline Design Pattern?

The Pipeline Design Pattern is where a complex process is broken down into
individual tasks. Each individual task is reusable and so the tasks can be
composed into complex processes.

This allows you to break up monolithic processes into smaller tasks that process
data and then pass that data to the next step.

Each task within the pipeline should receive and emit the same type of data.
This allows tasks to be added, removed, or replaced from the pipeline without
disturbing the other tasks.

If you are familiar with Unix operating systems you are probably already
familiar with piping the output from one command to be the input of another
command.

For example:

```bash
cat helloworld.txt | grep "hello world" | rev | > output.txt
```

In this example I’m reading the contents of file, searching for the string
“hello world”, reversing the string, and then adding it to a file called
output.txt

## How does Laravel use the Pipeline Design Pattern?

Laravel uses the Pipeline Design Pattern in a couple of different places
throughout the framework. By far the most prominent place is Laravel’s
implementation of middleware.

When a request comes into the application it is passed through a series of
middleware.

Each middleware has the responsibility of a single action. For example, setting
the cookies, checking for authentication, or preventing CSRF attacks.

Each stage will process the request and either pass it onto the next stage, or
reject the request and return the appropriate HTTP response.

## How do use the Laravel Pipeline

Using Laravel’s Pipeline is very easy. First you need to create a new
Illuminate\Pipeline\Pipeline object and inject it with an instance of
Illuminate\Contracts\Container\Container:

```php
$pipeline = app("Illuminate\Pipeline\Pipeline");
```

Next you pass in the object that you want to send through the pipeline:

```php
$pipeline->send($request);
```

Next you pass an array of tasks that should accept and process the request:

```php
$pipeline->through($middleware);
```

Finally you run the pipeline with a destination callback:

```php
$pipeline->then(function ($request) {
    // Do something
});
```

This is basically how the middleware functionality of the framework accepts HTTP
requests and then send the request through the defined middleware for that
route.

## What are the advantages of the Pipeline Design Pattern?

The Pipeline Design Pattern has a number of advantages.

Firstly, complex processes can be broken down into individual tasks. This makes
it very easy to test each task in isolation.

Tasks can be reused for different processes without duplicating logic. This is
useful if you are enforcing a business rule as you won’t have to duplicate that
logic.

It’s very easy to add, remove, or replace tasks of a complex process without
disturbing the existing process. Applications are constantly evolving and so
it’s likely that a change your code is going to change over time.

## What are the disadvantages of the Pipeline Design Pattern?

But of course, there are always disadvantages of any design pattern too.

Whilst composition makes each individual part simpler, it adds complexity when
you try to combine those parts as a single whole.

You also need to ensure that process works as a whole by having tests that can
run end-to-end, rather than just testing each individual task. The process is
going to be unreliable if it doesn’t work as a complete process.

Finally, it can also be difficult to understand a complex process when viewing
it as a set of discrete steps in isolation.

## another case studies

```php
<?php

class FictitiousRegisterController {

  public function register(Request $request): JsonResponse
  {
    $traveler = (new RegisterTraveler())->setRequest($request);
    
    $pipes = [
      ValidateInvitationCode::class,
      CreateUser::class,
      AssignPermissions::class,
      HandleMailingList::class,
      AssignToGroups::class,
      SendWelcomeEmail::class,
    ];
    
    return app(Pipeline::class)
      ->send($traveler)
      ->through($pipes)
      ->then(function ($traveler) {
        return response()->json([
          'message' => 'Success',
        ]);
      });
	}
}

class RegisterTraveler {

  private $request;

  private $user;

  public function setRequest($request)
  {
    $this->request = $request;
    return $this;
  }

  public function getRequest()
  {
    return $this->request;
  }

  public function setUser($user)
  {
    $this->user = $user;
    return $this;
  }

  public function getUser()
  {
    return $this->user;
  }
}

class CreateUser implements PipeInterface {
  public function handle($traveler, $next)
  {
    $traveler->setUser(
      User::create([
        'email' => $traveler->getRequest()->email,
        'password' => $traveler()->getRequest()->password,
      ])
    );

    return $next($traveler);
  }
}

class HandleMailingList implements PipeInterface
{
  public function handle($traveler, $next)
  {
    if ($traveler->getRequest()->subscribe) {
      MailingService::subscribe($traveler->getUser()->email);

      $traveler->getUser()->update([
        'subscriber' => true,
      ]);
    }

    return $next($traveler);
  }
}
```

How can we abort if something goes wrong in one of the steps? The simplest way
we’ve found is to throw an exception from a pipe. Wrapping the pipeline in a
try/catch has allowed us to handle potential errors from the pipes.

```php
<?php

class FictitiousRegisterController {

  public function register(Request $request): JsonResponse
  {
    $traveler = (new RegisterTraveler())->setRequest($request);

    $pipes = [
      ValidateInvitationCode::class,
      CreateUser::class,
      AssignPermissions::class,
      HandleMailingList::class,
      AssignToGroups::class,
      SendWelcomeEmail::class,
    ];
    
    try {
      return app(Pipeline::class)
        ->send($traveler)
        ->through($pipes)
        ->then(function ($traveler) {
          return response()->json([
            'message' => 'Success',
          ]);
        });
    } catch (Exception $e) {
      return response()->json([
        'message' => $e->getMessage(),
      ]);
    }
	}
}

class AssignToGroups implements PipeInterface
{
  public function handle($traveler, $next)
  {
    if ($traveler->getRequest()->has('group_uuids')) {
      collect($traveler->getRequest()->group_uuids)->each(function($uuid) use ($traveler) {
        if($group = Group::whereUuid($uuid)->first()) {
          $traveler->getUser()->groups()->attach($group);
        } else {
          throw new InvalidGroupException($uuid . ' is an invalid group uuid!');
        }
      });
    } else {
      $traveler->getUser()->groups()->attach(Group::whereDefault()->first());
    }
    
    return $next($traveler);
  }
}
```

Since we’ve set up the try/catch already, adding in a database transaction helps
clean up database state if an anomoly occurs. Start the transaction before the
pipeline kicks off, commit it if the process completes successfully, and
rollback in the exception catcher if there’s a problem

```php
<?php

class FictitiousRegisterController {

  public function register(Request $request): JsonResponse
  {
    $traveler = (new RegisterTraveler())->setRequest($request);

    $pipes = [
      ValidateInvitationCode::class,
      CreateUser::class,
      AssignPermissions::class,
      HandleMailingList::class,
      AssignToGroups::class,
      SendWelcomeEmail::class,
    ];

    try {
      DB::beginTransaction();
      return app(Pipeline::class)
        ->send($traveler)
        ->through($pipes)
        ->then(function ($traveler) {
          DB::commit();
          return response()->json([
            'message' => 'Success',
          ]);
        });
    } catch (Exception $e) {
      DB::rollback();
      return response()->json([
        'message' => $e->getMessage(),
      ]);
    }
	}
}
```

Earlier I noted that it can be difficult to test a single method that calls a
number of private methods. With a pipeline approach we have the freedom of
testing individual pipes in isolation, as well as having higher-level feature
tests that ensure the given input produces the expected output.

A pipeline is one solution we’ve used to handle complex data flows. There are
other options that may work better for you. The biggest takeaway from this
experience was to think one step above the actual implementation and identify
what the main players were, define their responsibilities, and implement a
solution that maintained their integrity.

After using Laravel Pipelines to handle complex data flows, we saw a few
patterns emerge: Database transactions, Pipe interface, Responses and exception
handling, This package [pipeline packages](https://github.com/zaengle/pipeline)
adds niceties on top of the Laravel Pipeline and consolidates them into a single
reusable location.
