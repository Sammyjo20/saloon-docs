# 🔌 Building SDKs

{% hint style="warning" %}
This documentation is still a work in progress while Saloon v2 is in beta.
{% endhint %}

Saloon provides everything you need to build a great SDK for an API. Saloon already offers the ability to mock responses, provide authentication, implement OAuth2 boilerplate and even record your API requests in your tests so that you can write tests for the API integration without hitting the real API each time. With Saloon you will be up and running in minutes. No more writing HTTP boilerplate code! Saloon also comes with just three dependencies making your library or SDK super lightweight.

### Saloon SDK Template

Saloon has an official SDK template that you can clone straight from GitHub or download and use. It has everything you need to get started building an SDK, and even has a configuration command to set up the class namespace and file names. [Click here to get started.](https://github.com/Sammyjo20/saloon-sdk-template)

### Example SDK

Saloon will be using PokéAPI as an example API. [Click here to see a full SDK example repository.](https://github.com/Sammyjo20/pokeapi-sdk)

### Getting Started

To start building an SDK with Saloon, we recommend that you create your base SDK class and extend the `Saloon\Http\Connector` class. This class allows you to configure the base URL, default headers, configuration and apply plugins. You can also configure request mocking really easily, so this will come in really handy when you want to write tests for the API without making real requests.

#### Example SDK Connector

Here is an example, I have created an SDK for a fun API for Pokémon - PokéAPI. As you can see, I have defined the API base URL, as well as use the constructor to request that the person using the API always provides an authentication token. In the real world, the PokéAPI does not require an API token.

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
     * Define the custom response
     *
     * @var string
     */
    protected string $response = PokeapiResponse::class;

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

Now that we have created the SDK class that extends the SaloonConnector class, all we need to do is instansiate it and provide the API token.

```php
<?php

use Pokeapi\Pokeapi;

$pokeapi = new Pokeapi('my-api-token');

// Ready to make requests!
```

### Sending Requests Directly

One of the ways that you can build SDKs in Saloon is by creating requests and then calling them from the SDK connector. This is the simplest way and you will get up and running really quickly with Saloon.

#### Getting started

Firstly, you will need to [create a request](../the-basics/requests.md), this is exactly the same as making a normal request described in the documentation. Any requirements like data or pagination should just be provided in each request’s constructor.

#### Use your request

When you have created the request, all that developers would need to do is run it! You can use the `send` method to send a request straight away, or the `request` method to instantiate the request.

```php
<?php

use Pokeapi\Pokeapi;

$pokeapi = new Pokeapi('my-api-token');

$response = $pokeapi->send(new GetAllPokemon(page: 1));
```

With this method, it’s really simple to build your SDK. All you would need to do is create all the requests and then document them in your README. Developers using your SDK can just instantiate your SDK and then use the `send` or`request` methods.

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

### Resource Classes

Resource classes are a pattern to help you combine your SDK requests into simple groups that are easy for the developer to find and consume. The tutorial below will guide you through creating a resource class however you should customise it to your SDK.&#x20;

#### Creating the base resource class

Let's start by creating a `Resource` class. This class should just contain a constructor with requires a connector.

```php
<?php

class Resource
{
    public function __construct(protected Connector $connector)
    {
        //
    }
}
```

#### Creating a resource

Now let's create a resource. In my example, I will create a Pokémon resource which will group all the Pokémon related routes together.

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

Next, we'll define a method on the connector which uses this resource class.

```php
<?php

class Pokeapi extends Connector
{
    // { ... }
    
    public function pokemon(): Resource
    {
        return new PokemonResource($this);
    }
}
```

#### Using the resource

Now we'll have access to our `$pokeapi->pokemon()` resource which contains all the requests we have defined!

```php
<?php

$pokeapi = new Pokeapi('my-api-token');

$allPokemon = $pokeapi->pokemon()->all(page: 1);

$giratina = $pokeapi->pokemon()->get(id: 'giratina');
```
