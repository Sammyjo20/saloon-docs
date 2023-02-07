# 🛤 Data Transfer Objects

When building API integrations, sometimes dealing with a raw response or a JSON response can be tedious and unpredictable. Data transfer objects are a good solution as they allow you to define a structure for a request and response. Saloon supports casting a response from an API request into a DTO.

### Casting responses into DTOs

Firstly, in your request or connector, extend the `createDtoFromResponse` method. Within this method, you get access to the `Response` object containing the response to cast into a data transfer object. In this example, I have created a `fromResponse` method so I can write all the logic&#x20;

```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Response;

class GetServerRequest extends Request
{
    // {...}
    
    public function createDtoFromResponse(Response $response): mixed
    {
        return Server::fromResponse($response);
    }
}
```

I have created a `Server` data transfer object that I will use. This is what the `Server` DTO looks like this:

```php
<?php

use Saloon\Contracts\Response;

class Server
{
    public function __construct(
        readonly public int $id,
        readonly public string $name,
        readonly public string $ipAddress,
    ){}

    public static function fromResponse(Response $response): self
    {
        $data = $response->json();

        return new static($data['id'], $data['name'], $data['ip']);
    }
}
```

### Retrieving your DTO

Now we have defined our DTO on our request or connector, we can use the build in `dto` or `dtoOrFail` methods on our response class.

```php
<?php

$connector = new ForgeConnector;

$response = $connector->send(new GetServerRequest(id: 12345));

// Create a DTO even if the response was a failure
$server = $response->dto();

// Create a DTO or throw an exception if the response was not successful
$server = $response->dtoOrFail();
```

{% hint style="warning" %}
When using the `dto` method, Salon will attempt to create a DTO from your response no matter the status of the response. This allows you to create "error" data transfer objects. If you don't want to use this functionality, you can use the `dtoOrFail` method which will throw a LogicException if the response was a failure. **You can customise what is considered a failed response** [**here**](../the-basics/handling-failures.md#customising-when-saloon-thinks-a-request-has-failed)**.**
{% endhint %}

### Accessing the response from your DTO

Sometimes debugging a DTO can be difficult, especially if you have passed the data object through your application and no longer have access to the original `Response` that you created the response from. Saloon can automatically insert the response for you if you use the `HasResponse` trait and the `WithResponse` interface. Let's add it to our existing `Server` DTO.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Contracts\Response;
use Saloon\Traits\Responses\HasResponse;
use Saloon\Contracts\DataObjects\WithResponse;

<strong>class Server implements WithResponse
</strong>{
<strong>    use HasResponse;
</strong>
    public function __construct(
        readonly public int $id,
        readonly public string $name,
        readonly public string $ipAddress,
    ){}

    public static function fromResponse(Response $response): self
    {
        $data = $response->json();

        return new static($data['id'], $data['name'], $data['ip']);
    }
}
</code></pre>

Now whenever we retrieve an instance of our data transfer object, you will be able to access the underlying response that was created with it!

```php
<?php

$server = $response->dto();

$response = $server->getResponse();
```
