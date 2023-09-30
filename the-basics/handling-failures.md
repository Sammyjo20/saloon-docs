# 🔥 Handling Failures

Saloon has a powerful exception handler that has lots of exceptions you can use in your application. It can also be customised on a per-connector and per-request basis. This is especially useful when you have an API which returns an OK (200) response with an error in the body.

When you send a request, Saloon will not do anything if the request fails, but by default, it will use the status code to determine if a request is successful or not. The only exception to this is if Saloon cannot connect to an API, which will throw a `FatalRequestException`.

```php
<?php

$forge = new ForgeConnector;
$response = $forge->send(new ErrorRequest);

$response->failed(); // true
$response->status(); // 500
$response->body(); // {"message": "Server Error"}
```

### Always throw exceptions on failed requests

You may wish to always throw an exception if a request fails. You may add the `AlwaysThrowOnErrors` trait on your connector, and every request that fails will throw an exception, just like if you were to use the `throw` method. You may also add this trait to a request.

```php
<?php

use Saloon\Traits\Plugins\AlwaysThrowOnErrors;

class ForgeConnector extends Connector
{
    use AlwaysThrowOnErrors;
    
    // {...}
}
```

### Using the throw method

On a per-response basis, you may use the `throw` method after sending your response. This method will throw an exception if the response has a "failed" HTTP status code like 4xx or 5xx.

```php
<?php

$forge = new ForgeConnector;
$response = $forge->send(new ErrorRequest);

// throws InternalServerErrorException (extends ServerException)

$response->throw();
```

### Response Exceptions

Saloon's default exception handler contains the following exceptions based on the status code and severity of the exception. These are thrown depending on the method you use below.

```
SaloonException
├── FatalRequestException (Connection Errors)
└── RequestException (Request Errors)
    ├── ServerException (5xx)
    │   ├── InternalServerErrorException (500)
    │   ├── ServiceUnavailableException (503)
    │   └── GatewayTimeoutException (504)
    └── ClientException (4xx)
        ├── UnauthorizedException (401)
        ├── ForbiddenException (403)
        ├── NotFoundException (404)
        ├── MethodNotAllowedException (405)
        ├── RequestTimeOutException (408)
        ├── UnprocessableEntityException (422)
        └── TooManyRequestsException (429)
```

### Using the onError method

You may wish to write some custom logic in your application if a request fails, but you don't want to throw an exception. You may use the `onError` method from the response and provide a callable to be executed if an error happens.

```php
<?php

use Saloon\Contracts\Response;

$response = $forge->send(new ErrorRequest);

$response->onError(function (Response $response) {
    // Handle any logic when an error happens.
});

// Application logic is continued
```

{% hint style="info" %}
The `onError` method will only work if your HTTP status is either 4xx or 5xx.
{% endhint %}

### Handling failures with promises

When sending requests using `sendAsync` or using request pooling, you will receive a `PromiseInterface` instance. Since this class catches exceptions, Saloon will automatically pass the request exception in the `otherwise` block, and you do not have to use the `throw` method.

```php
<?php

use Saloon\Contracts\Response;

$forge = new ForgeConnector('api-token');
$promise = $forge->sendAsync(new GetServersRequest);

$promise
    ->then(function (Response $response) {
        // Handle successful response
    })
    ->otherwise(function (RequestException $exception) {
        // Handle failed request
    });
```

### Other response methods

Saloon offers some other methods to handle failed responses.

| Method             | Description                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------ |
| failed             | Determines if a request has failed. By default, it will return true if the response status is not 2xx. |
| serverError        | Returns true if the response status is >= 500.                                                         |
| clientError        | Returns true if the response status is between 400 and 500.                                            |
| toException        | Creates an exception if the response is considered "failed".                                           |
| throw              | Will throw an exception if the response is considered "failed".                                        |
| getSenderException | Get the sender exception if a request failed.                                                          |
| onError            | Allows you to define a callback if the response is considered "failed".                                |

### Customising when Saloon thinks a request has failed

By default, Saloon will consider a request as failed if the status code is 4xx or 5xx, for both client and server errors. You may choose to change how Saloon considers a request as failed, For example, you may integrate with an API which still returns a 2xx response status but with an error message in the response body. Just extend the `hasRequestFailed` method on your connector or request.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;
use Saloon\Contracts\Response;

class ForgeConnector extends Connector
{
    // {...}
    
    public function hasRequestFailed(Response $response): ?bool
    {
        return str_contains($response->body(), 'Server Error');
    }
}
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Response;

class ErrorRequest extends Request
{
    // {...}
    
    public function hasRequestFailed(Response $response): ?bool
    {
        return str_contains($response->body(), 'Server Error');
    }
}
```
{% endtab %}
{% endtabs %}

### Customising when exceptions are thrown

If you use the `throw` method or the `AlwaysThrowsOnErrors` trait, Saloon will throw an exception if the status code is 4xx or 5xx. Sometimes you may wish to change this behaviour. You should extend the `shouldThrowRequestException` method to change the default behaviour.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;
use Saloon\Contracts\Response;

class ForgeConnector extends Connector
{
    // {...}
    
    public function shouldThrowRequestException(Response $response): bool
    {
        return str_contains($response->body(), 'Server Error');
    }
}
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Response;

class ErrorRequest extends Request
{
    // {...}
    
    public function shouldThrowRequestException(Response $response): bool
    {
        return str_contains($response->body(), 'Server Error');
    }
}
```
{% endtab %}
{% endtabs %}

### Customising the request exception

By default, Saloon will use the exceptions [listed above](handling-failures.md#default-exceptions), but you may choose to return your own exception if a request has failed. Just extend the `getRequestException` method on either your connector or request. You will receive an instance of the response and a sender exception, which may be nullable.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;
use Saloon\Contracts\Response;
use \Throwable;

class ForgeConnector extends Connector
{
    // {...}
    
    public function getRequestException(Response $response, ?Throwable $senderException): ?Throwable
    {
        return new CustomException('Oh yee-naw!', $response, $senderException);
    }
}
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Response;
use \Throwable;

class ErrorRequest extends Request
{
    // {...}
    
    public function getRequestException(Response $response, ?Throwable $senderException): ?Throwable
    {
        return new CustomException('Oh yee-naw!', $response, $senderException);
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
When the `getRequestException` method is defined on both the connector and the request, the request method will take priority.
{% endhint %}
