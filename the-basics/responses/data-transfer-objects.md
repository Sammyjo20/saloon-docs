# Data Transfer Objects

You may wish to cast the data you receive in an API response to a data transfer object (DTO). Saloon has a built-in plugin that makes this easy. With this plugin, you can specify the DTO on a per-request basis.&#x20;

### Configuring

Firstly, add the **CastsToDto** trait to your request.

```php
<?php

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Plugins\CastsToDto;

class GetForgeServerRequest extends SaloonRequest
{
    use CastsToDto;

    protected ?string $method = Saloon::GET;

    protected ?string $connector = ForgeConnector::class;
    
    // ...
}
```

After that, add the `castToDto` method to your request. It has one argument which is an instance of **SaloonResponse.** This method is only run after a successful response has been received from the API. This method should return your data transfer object fully constructed from the response data you received.&#x20;

```php
<?php

// ...

use App\Data\Server;

class GetForgeServerRequest extends SaloonRequest
{
    use CastsToDto;
    
    // ...
    
    protected function castToDto(SaloonResponse $response): object
    {
        return Server::fromSaloon($response);
    }
}
```

This is how my data transfer object looks internally. I have created a `fromSaloon` method however you can use any method of populating your DTO.

```php
<?php

class Server
{
    public function __construct(
        public string $serverId,
        public string $name,
    ){}

    public static function fromSaloon(SaloonResponse $response): self
    {
        $data = $response->json();

        return new static($data['id'], $data['name']);
    }
}
```

### Retrieving your DTO

Finally, when you retrieve a successful response from the API, you can use the `dto()` method on the response to get the fully constructed DTO.

```php
<?php

$request = new GetForceServerRequest(serverId: '12345');
$response = $request->send();

$server = $response->dto();
```

{% hint style="warning" %}
Saloon will only cast the response to your DTO if the response was successful.
{% endhint %}
