# SOLID Principles

First, let's discuss what SOLID stands for:

- Single-responsibility principle
- Open-closed principle
- Liskov substitution principle
- Interface segregation principle
- Dependency inversion principle

## Single-Responsibility Principle

Each class should have only one reason to change

a bad one

This is a modern example of mixing the data and the representation layer in one
class

```php
class UserResource extends JsonResource
{
	publicfunctiontoArray($request)
	{
		$mostPopularPosts = $user->posts()
			->where('like_count', '>', 50)
			->where('share_count', '>', 25)
			->orderByDesc('visits')
			->take(10)
			->get();
		
		return [
			'id' => $this->id,
			'full_name' => $this->full_name, 
			'most_popular_posts' => $mostPopularPosts,
		];
	}
}
```

a good one

```php
class UserResource extends JsonResource
{
	publicfunctiontoArray($request)
	{
		return [
			'id' => $this->id,
			'full_name' => $this->full_name, 
			'most_popular_posts' => $this->when(
					$request->most_popular_posts,
					GetMostPopularPosts::execute($user), 
			),
		];
	}
}

class GetMostPopularPosts 
{
	/**
	* @return Collection<Post> 
	*/
	public static function execute(User $user): Collection {
		return $user->posts()
			->where('like_count', '>', 50)
			->where('share_count', '>', 25)
			->orderByDesc('visits')
			->take(10)
			->get();
	} 
}
```

Now we have two well-defined classes:

- `UserResource` is responsible only for the representation and it has one
  reason to change.
- `GetMostPopularPosts` is responsible only for the query and it has one reason
  to change.

## Open-Closed Principle

A class should be open for extension but closed for modification

example:

```php
trait Likeable 
{
	public function like(): void 
	{

	}
	
	public function dislike(): void 
	{

	}
}

class Post extends Model 
{
	use Likeable; 
}

class Comment extends Model 
{
	use Likeable; 
}
```

This is pretty standard, right? But think about what happened here.

We just added new functionality to multiple classes without changing them! We
extended our classes instead of modifying them

## Liskov Substitution Principle

Each base class can be replaced by its subclasses

example

```php
abstract class EmailProvider 
{
		abstract public function addSubscriber(User $user): array;
}
	
class MailChimp extends EmailProvider 
{
	public function addSubscriber(User $user): array 
	{ 
		// Using MailChimp API
	}
}

class ConvertKit extends EmailProvider 
{
		public function addSubscriber(User $user): array 
		{
			// Using ConvertKit API
		}
}
```

We have an abstract EmailProvider and we use both MailChimp and ConvertKit for
some reason. These classes should behave exactly the same way, no matter what.

```php
class AuthController 
{
	public function register( RegisterRequest $request, MailChimp $emailProvider){}
}

class AuthController 
{
	public function register( RegisterRequest $request, ConvertKit $emailProvider){}
}
```

## Interface Segregation Principle

You should have many small interfaces instead of a few huge ones

example

```php
class TextInput extends Field implements CanHaveNumericState, Contracts\CanBeLengthConstrained, Contracts\HasAffixActions
{
    use Concerns\CanBeAutocapitalized;
    use Concerns\CanBeAutocompleted;
    use Concerns\CanBeLengthConstrained;
    use Concerns\CanBeReadOnly;
    use Concerns\HasAffixes;
    use Concerns\HasDatalistOptions;
    use Concerns\HasExtraInputAttributes;
    use Concerns\HasInputMode;
    use Concerns\HasPlaceholder;	
}
```

Each of those traits has a pretty small and well-defined interface and it adds a
small chunk of functionality to the class

## Dependency Inversion Principle

Depend upon abstraction, not concretions.

example

```php
interface MarketDataProvider 
{
	public function getPrice(string $ticker): float; 
}

class IexCloud implements MarketDataProvider 
{
	public function getPrice(string $ticker): float 
	{
		// Using IEX API
	} 
}

class Finnhub implements MarketDataProvider 
{
	public function getPrice(string $ticker): float 
	{
		// Using Finnhub API
	} 
}

class CompanyController 
{
		public function show(Company $company, MarketDataProvider $marketDataProvider)
		{
			$price = $marketDataProvider->getPrice();
			return view('company.show', compact('company', 'price')); 
		}
}
```

So every class should depend on the abstract MarketDataProvider not on the
concrete implementation.
