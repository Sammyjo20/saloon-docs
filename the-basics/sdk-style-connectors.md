# Building SDKs

Saloon offers everything you need to build a great SDK for an API. Saloon already offers the ability to mock responses, provide authentication, implement OAuth2 boilerplate and even record your API requests in your tests so that you can write tests for the API integration without hitting the real API each time. With Saloon you will be up and running in minutes.

### Saloon SDK Template

Saloon has an official SDK template that you can clone straight from GitHub or download and use. It has everything you need to get started building an SDK, and even has a configuration command to get started. [Click here to get started.](https://github.com/Sammyjo20/saloon-sdk-template)

### Getting Started

To start building an SDK with Saloon, we recommend that you create your base SDK class and extend the **SaloonConnector** class. The **SaloonConnector** class \*\*\*\*allows you to configure the base URL, default headers, configuration and apply plugins. You can also configure request mocking really easily, so this will come in really handy when you want to write tests for the API without making real requests.

#### Example SDK Connector

Here is an example, I have created an SDK for the popular movie database API - TMDB. As you can see, I have defined the API base URL, as well as use the constructor to request that the person using the API always provides an authentication token.

```php
<?php

namespace Sammyjo20\TMDB;

use Sammyjo20\Saloon\Http\SaloonConnector;
use Sammyjo20\TMDB\Responses\TMDBResponse;
use Sammyjo20\Saloon\Traits\Plugins\AcceptsJson;

class TMDB extends SaloonConnector
{
    use AcceptsJson;

    protected string $apiBaseUrl = 'https://api.themoviedb.org/3';

    protected array $requests = [];

    // Define the base URL.

    public function defineBaseUrl(): string
    {
        return $this->apiBaseUrl;
    }

    // Constructor requires the token.
    // Also allow to overwrite the API base URL for local servers.

    public function __construct(string $token, string $baseUrl = null)
    {
        $this->withTokenAuth($token);

        if (isset($baseUrl)) {
            $this->apiBaseUrl = $baseUrl;
        }
    }

    // Headers that will be used on all requests

    public function defaultHeaders(): array
    {
        return [
	    'Content-Type' => 'application/json',
	];
    }

    // Default Guzzle Config Options

    public function defaultConfig(): array
    {
        return [
	    'timeout' => 30,
	];
    }
}
```

#### Using the SDK connector

Now that we have created the SDK class that extends the SaloonConnector class, all we need to do is instansiate it and provide the API token.

```php
<?php

use Sammyjo20\TMDB\TMDB;

$tmdb = new TMDB('my-api-token');

// Ready to make requests!
```

### Recommended Method: Using Requests Directly

One of the ways that you can build SDKs in Saloon is by creating requests and then calling them from the SDK connector. This is the simplest way and you will get up and running really quickly with Saloon.

#### 1. Make a Saloon Request

Firstly, you will need to make a SaloonRequest, this is exactly the same as making a normal request described in the documentation. Any requirements like data or pagination should just be provided in each request’s constructor.

#### 2. Call your request

When you have created the request, all that developers would need to do is run it! You can use the `send` method to send a request straight away, or the `request` method to instantiate the request.

```php
<?php

use Sammyjo20\TMDB\TMDB;

$tmdb = new TMDB('my-api-token');

// Send a request straight away.

$response = $tmdb->send(new GetPopularMovies(page: 1));

// Or if you would like to do something with the request before sending it.

$request = $tmdb->request(new GetPopularMovies(page: 1));
```

With this method, it’s really simple to build your SDK. All you would need to do is create all the requests and then document them in your README. Developers using your SDK can just instantiate your SDK and then use the `send` or`request` methods.

### Method Two: Create Request Collections

Alternatively, you can define request classes and groups of requests on your SDK class by using the `$requests` property to define requests. By using this method, you will have to register your API routes, but then developers can use methods to make API calls.

#### Example

Here is an example using the same TMDB SDK as used earlier. You are able to register requests and request collections using Saloon.

```php
$tmdb = new TMDB('my-api-token');

$request = $tmdb->movies()->getPopular(page: 1);
$response = $request->send();
```

#### Individual Requests

If you define a `$requests` array property on your connector. You can specify individual requests that will be converted into "magic" methods.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class TMDB extends SaloonConnector
{
    protected array $requests = [
        GetPopularMovies::class, // $tmdb->getPopularMovies()
    ];

    // ...
}
```

Saloon will create a method for the request based on its name in camelCase. For example, our registered **GetPopularMovies** class will now have a method for it on the connector called `GetPopularMovies`. When you call this method, Saloon will instantiate the **GetPopularMovies** class.

```php
<?php

$tmdb = new TMDB('my-api-token');

$request = $connector->getPopularMovies(page: 1);

$response = $request->send();
```

#### Customising the request methods

You may want to use your own method names for requests on your connector. If you would like to do this, just add a key for the request to rename the method.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class TMDB extends SaloonConnector
{
    protected array $requests = [
        'get_popular_movies' => GetPopularMovies::class, // $tmdb->get_popular_movies()
    ];

    // ...
}
```

#### Request Collections

You can also have many requests in an SDK, each separated into their own groups. Saloon also makes this easy with two ways of doing it. You can build custom request collections that will be returned automatically through magic methods. This is great if you would like to separate your SDK into different groups, like a "movies" group and a "tv shows" group containing different requests.

To get started, create a class and extend the base `RequestCollection` abstract class. Request collections will be given an instance of the connector which you can access.

```php
<?php

use Sammyjo20\Saloon\Http\RequestCollection;

class MoviesCollection extends RequestCollection
{
    public function getPopular(int $page): array
    {
        // You can access $this->connector to make requests...
        
        $request = $this->connector->request(new GetPopularMovies($page));
        
	return $request->send();
    }
    
    // ... Your other methods...
}
```

(hint) The base **RequestCollection** class contains a constructor which provides the class with the connector instance.

After you have created the class, specify the collection in your `$requests` array on your connector.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class TMDB extends SaloonConnector
{
    protected array $requests = [
        'movies' => MoviesCollection::class,
    ];

    // ...
}
```

Now you can call the custom collection and methods inside of it!

```php
<?php

$tmdb = new TMDB('my-api-token');

$response = $tmdb->movies()->getPopular();
```

#### IDE Auto-Completion

When using request collections and requests inside your SDK connector class, your IDE will not likely know that the methods exist since Saloon uses magic methods to call the requests. You can get around this by defining the methods in a doc-block above the class definition.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

/**
 * Define this in your doc-block.
 * 
 * @method MoviesCollection movies
 */
class TMDB extends SaloonConnector
{
    protected array $requests = [
        'movies' => MoviesCollection::class,
    ];

    // ...
}
```

### Custom Responses

You may wish to return a custom response instead of the default response to users to make it more developer-friendly. For example, if I wanted to return a TMDBResponse instead of SaloonResponse, I can create a custom response and define it on the SDK. [Click here to read more.](responses/#custom-responses)
