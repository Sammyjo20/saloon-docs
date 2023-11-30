# 🔥 Handling Failures

Saloon has a powerful exception handler that has lots of exceptions you can use in your application. It can also be customised on a per-connector and per-request basis.

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

You may wish to throw an exception whenever a request fails (4xx or 5xx response). You can add the `AlwaysThrowOnErrors` trait on your connector, and then every request that fails will throw an exception.

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

### Handling failures with promises

When sending requests using `sendAsync` or using request pooling, you will receive a `PromiseInterface` instance. Since this class catches exceptions, Saloon will automatically pass the request exception in the `otherwise` block, and you do not have to use the `throw` method.

```php
<?php

use Saloon\Http\Response;

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

### Customising when Saloon thinks a request has failed

You may integrate with an API which returns a 200 response status but with an error message in the response body. To handle this, you can extend the `hasRequestFailed` method on your connector or request.

{% tabs %}
{% tab title="Connector" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Connector;
use Saloon\Http\Response;

class ForgeConnector extends Connector
{
    // {...}
    
<strong>    public function hasRequestFailed(Response $response): ?bool
</strong>    {
<strong>        return str_contains($response->body(), 'Server Error');
</strong>    }
}
</code></pre>
{% endtab %}

{% tab title="Request" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Http\Response;

class ErrorRequest extends Request
{
    // {...}
    
<strong>    public function hasRequestFailed(Response $response): ?bool
</strong>    {
<strong>        return str_contains($response->body(), 'Server Error');
</strong>    }
}
</code></pre>
{% endtab %}
{% endtabs %}

### Customising the request exception

By default, Saloon will use the exceptions [listed above](handling-failures.md#response-exceptions), but you may choose to return your own exception if a request has failed. Just extend the `getRequestException` method on either your connector or request. You will receive an instance of the response and a sender exception, which may be nullable.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;
use Saloon\Http\Response;
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
use Saloon\Http\Response;
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
Priority is given to the request when you extend the `getRequestException` method on both your connector and request.
{% endhint %}
