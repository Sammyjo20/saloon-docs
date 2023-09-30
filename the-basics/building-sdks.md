# 🪝 Building SDKs

Saloon provides everything you need to build a great SDK or library for an API. It already offers the ability to mock responses, provide authentication, implement OAuth2 boilerplate and even record your API requests in your tests. With Saloon you won't need to write the same boilerplate code over and over again. Saloon comes with just three dependencies making your library or SDK lightweight.

### Example SDK

This documentation will be using PokéAPI as an example API. [Click here to see a full SDK example repository.](https://github.com/Sammyjo20/pokeapi-sdk)

### Getting Started

To start building an SDK with Saloon, we recommend that you create a connector as your SDK class. Once you are familiar with connectors, create a class and extend the `Saloon\Http\Connector` class. This class allows you to configure the base URL and any defaults you may need like default headers or authentication.&#x20;

#### Example SDK Connector

This is an example SDK for a fun API the [PokéAPI](https://pokeapi.co/). As you can see, I have defined the API base URL, as well as used the constructor to require the person using the API always provides an authentication token. In the real world, the PokéAPI does not require an API token.

```php
<?php

namespace Pokeapi;

use Generator;
use Saloon\Http\Connector;
use Saloon\Contracts\Request;
use Pokeapi\Responses\PokeapiResponse;

class Pokeapi extends Connector
{
    /**
     * Resolve the base URL of the service.
     *
     * @return string
     */
    public function resolveBaseUrl(): string
    {
        return 'https://pokeapi.co/api/v2';
    }

    /**
     * Define default headers
     *
     * @return string[]
     */
    protected function defaultHeaders(): array
    {
        return [
            'Accept' => 'application/json',
            'Content-Type' => 'application/json',
        ];
    }
    
    /**
     * Constructor
     *
     * @return void
     */
    public function __construct(string $token)
    {
        $this->withTokenAuth($token);
    }
}
```

#### Using the SDK connector

Now that we have created the SDK class, all we need to do is instantiate it and provide the API token. We are now ready to send requests through our SDK. You may add your own methods to this class is it is the root class of your SDK.

```php
<?php

use Pokeapi\Pokeapi;

$pokeapi = new Pokeapi('my-api-token');

// Ready to make requests!
```

### Sending Requests

One of the ways that you can build SDKs in Saloon is by creating request classes and then calling them from the SDK connector. This is the simplest way and you will get up and running really quickly with Saloon. Make sure you are familiar with how requests work first.

#### Getting started

Firstly, you will need to [create a request](requests.md), this is exactly the same as making a normal request described in the documentation. Any requirements like data or pagination should just be provided in each request’s constructor.

#### Use your request

When you have created the request, all that developers would need to do is to instantiate and send the request on the connector. This means you only need to have two classes as a minimum for a fully-working SDK!

```php
<?php

use Pokeapi\Pokeapi;

$pokeapi = new Pokeapi('my-api-token');
$request = new GetAllPokemon(page: 1);

// Developers would just send the request.

$response = $pokeapi->send($request);
```

With this method, it’s really simple to build your SDK. All you would need to do is create all the requests and then document them in your README. Developers using your SDK can just instantiate your SDK and then use the `send` methods.

### Sending Requests With Methods

Sometimes you may want to make it easy for the developer to find all the methods that they need to call the API through your SDK. You can create methods on your connector which send an API request or you could write a "resource" class that contains lots of requests

{% tabs %}
{% tab title="Definition" %}
```php
<?php

class Pokeapi extends Connector
{
    // { ... }
    
    public function allPokemon(int $page): Response
    {
        return $this->send(new GetAllPokemon($page));
    }
}
```
{% endtab %}

{% tab title="Usage" %}
```php
<?php

$pokeapi = new Pokeapi('my-api-token');

$response = $pokeapi->allPokemon(page: 1);
```
{% endtab %}
{% endtabs %}

### Request Resources

The resource pattern can help you combine your SDK requests into simple groups that are easy for the developer to find and consume. The tutorial below will guide you through creating a resource class however you should customise it to your SDK.&#x20;

#### Creating the base resource class

Let's start by creating a `Resource` class. This class should just contain a constructor that passes in an instance of `Saloon\Contracts\Connector`.

```php
<?php

use Saloon\Contracts\Connector;

class Resource
{
    public function __construct(protected Connector $connector)
    {
        //
    }
}
```

#### Creating a resource

Now let's create a resource. For this API, I will create a Pokémon resource which will group all the Pokémon requests together. Imagine a resource like a controller in an MVC framework like Laravel. You should pass any arguments the request needs through the method arguments.

```php
<?php

class PokemonResource extends Resource
{
     public function all(int $page): Response
     {
         return $this->connector->send(new GetAllPokemon($page));
     }
     
     public function get(int $id): Response
     {
         return $this->connector->send(new GetSinglePokemon($id));
     }
}
```

#### Defining a resource on your connector

Now we'll define a method on the connector which returns this resource class. Don't forget to pass the connector's instance (`$this`) into the resource.

```php
<?php

class Pokeapi extends Connector
{
    // { ... }
    
    public function pokemon(): PokemonResource
    {
        return new PokemonResource($this);
    }
}
```

#### Using the resource

Now all our users have to do is access the `pokemon()` method on the SDK class to get access to all the various requests that our SDK has to offer.

```php
<?php

$pokeapi = new Pokeapi('my-api-token');

$allPokemon = $pokeapi->pokemon()->all(page: 1);

$giratina = $pokeapi->pokemon()->get(id: 'giratina');
```

### Custom Responses

You may wish to customise the `Response` class that is returned by Saloon from your SDK connector. [Click here to read more about custom responses.](building-sdks.md#custom-responses)

### Additional Features

Please read through the other sections of Saloon's documentation to see the other features that you can offer for your SDK. Every other feature like testing, authentication, failure handling is all supported for SDKs.

### Testing

When building an SDK, it's important to write tests that ensure the SDK calls the correct requests from an API and returns the right response, especially if you're converting the response into a data-transfer-object. [Read through the testing section of the documentation](../testing/manual-fake-responses.md) to get familiar with mocking and recording requests.
