# ☝ Solo Requests

While Saloon's typical setup of a connector and requests is great, sometimes all you need is to make a single request to a service. For scenarios like these, you may create a "SoloRequest" instead of making a connector and a single request. This saves you from having to create additional classes.&#x20;

### Setup

Create a class, but instead of extending `Saloon\Http\Request`, you should extend `Saloon\Http\SoloRequest.` Next, just define everything else like you would a normal request. Make sure to include the full URL of the service you are integrating with.

```php
<?php

use Saloon\Http\SoloRequest;
use Saloon\Enums\Method;

class GetAllPokemonRequest extends SoloRequest
{
    protected Method $method = Method::GET;
    
    public function resolveEndpoint(): string
    {
        return 'https://pokeapi.co/api/v2/pokemon';
    }
}
```

### Defaults

Saloon Requests allow you to define all your default headers, config, query parameters and define request body just like you would traditionally with a connector.

### Sending Solo Requests

As you don't have a connector for this request, you can use the `send` or `sendAsync`  methods directly on the request instance. This method works exactly the same as it would on the connector.&#x20;

```php
<?php

$request = new GetAllPokemonRequest;
$response = $request->send();
```
