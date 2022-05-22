# Responses

Saloon requests will return with a **SaloonResponse** class that you can interact with the response with.

### Available Methods

* json(): array
* body(): string
* stream(): StreamInterface
* object(): object
* collect(): Collection
* dto(): object
* header($header): string
* headers(): array
* status(): int
* successful(): bool
* ok(): bool
* redirect(): bool
* clientError(): bool
* serverError(): bool
* onError(callable $callback)
* toException()
* throw()
* getRequestOptions(): array
* getGuzzleException(): ?RequestException
* isCached(): bool
* isMocked(): bool
* toGuzzleResponse(): Response
* toPsrResponse(): Response

### Handling Failures

Sometimes the API you are connecting with will fail. When this happens, Saloon has a really easy way to see the error response as well as handling it. If the response failed, you will still receive a **SaloonResponse**, but the status will represent the failure. To get the data from the response, you should use the `json()` or the `body()` methods to see the message from the server.

#### Throwing exceptions on failures

Sometimes you want to just let Saloon throw an exception if the request failed. To do this, you can use the `throw()` method on the response. This will create a **SaloonRequestException** containing the error as well as methods to access the original response.

```php
<?php

use App\Http\Saloon\Requests\GetForgeServerRequest;

$request = new GetForgeServerRequest(serverId: '123456');
$response = $request->send();

$response->throw(); // Will throw SaloonRequestException if the request fails.

$data = $response->json();
```

### Casting to Data Transfer Objects (DTOs)

You may wish to cast the data you receive in an API response to a data transfer object (DTO). Saloon has a built-in plugin that makes this easy. [Click here to read more about casting to DTOs.](data-transfer-objects.md)

### Custom Responses

Sometimes you may want to use your own response class. This is is useful if you would like to add your own methods or overwrite Saloon's response methods.&#x20;

To use a custom response, firstly create your custom response class and then overwrite the **$response** property on your connector or request.

{% hint style="info" %}
If you are using Laravel, you can use the **php artisan saloon:response** Artisan command to create a response.
{% endhint %}

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class ForgeConnector extends SaloonConnector
{
    // ...

    protected ?string $response = CustomResponse::class;
    
    // ...
}
```

{% hint style="info" %}
If the custom response is added to the connector, it will be applied to all requests made by the connector.
{% endhint %}
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;

class GetForgeServerRequest extends SaloonRequest
{
    // ...

    protected ?string $response = CustomResponse::class;
    
    // ...
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Your custom response must extend **SaloonResponse.**
{% endhint %}
