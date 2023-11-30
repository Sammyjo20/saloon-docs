# 📡 Responses

Depending on how you sent your request (synchronous/asynchronous) you will either receive an instance of `Response` or a `PromiseInterface.`

### Handling synchronous responses

By default, Saloon will return an instance of `Saloon\Http\Response`. This response class contains many helpful methods for interacting with your HTTP response. You can see a list of the available methods below.

```php
<?php

$forge = new ForgeConnector('api-token');
$response = $forge->send(new GetServersRequest);

$body = $response->body();
$decodedBody = $response->json();
```

{% hint style="warning" %}
By default, Saloon will not throw an exception if a synchronous request fails. [Refer to the handling failures section for handling errors.](handling-failures.md)
{% endhint %}

### Key Methods

Below are some of the key methods provided by Saloon's `Response` class.

| Method                                                                 | Description                                                                                                                                                                                           |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `getPendingRequest`                                                    | Returns the `PendingRequest` class that was built up for the request.                                                                                                                                 |
| `getConnector`                                                         | Return the `Connector` that was used to send the request.                                                                                                                                             |
| `getRequest`                                                           | Returns the `Request` that was sent.                                                                                                                                                                  |
| `getPsrRequest`                                                        | Returns the PSR-7 request that was built up by Saloon                                                                                                                                                 |
| `getPsrResponse`                                                       | Return the PSR-7 response that was built up by the HTTP client/sender.                                                                                                                                |
| `status`                                                               | Returns the response status code.                                                                                                                                                                     |
| `headers`                                                              | Returns all response headers                                                                                                                                                                          |
| `header`                                                               | Returns a given header                                                                                                                                                                                |
| `body`                                                                 | Returns the raw response body as a string                                                                                                                                                             |
| `json`                                                                 | Retrieves a JSON response body and json\_decodes it into an array.                                                                                                                                    |
| `array`                                                                | Alias of `json`                                                                                                                                                                                       |
| `collect`                                                              | Retrieves a JSON response body and json\_decodes it into a Laravel collection. Requires `illuminate/collections`.                                                                                     |
| `object`                                                               | Retrieves a JSON response body and json\_decodes it into an object.                                                                                                                                   |
| `xml`                                                                  | Retrieves the response body and creates a `SimpleXmlElement` for easier XML parsing                                                                                                                   |
| `dom`                                                                  | Used for HTML responses - returns a Symfony Crawler instance. Requires `symfony/dom-crawler`.                                                                                                         |
| `stream`                                                               | Returns the response body as an instance of `StreamInterface`                                                                                                                                         |
| `getRawStream`                                                         | Returns the response body as a file resource.                                                                                                                                                         |
| `saveBodyToFile`                                                       | Allows you to save the raw body to a file or open file resource.                                                                                                                                      |
| `dto`                                                                  | Converts the response into a data-transfer object. You must define your DTO first, [click here to read more.](data-transfer-objects.md)                                                               |
| `dtoOrFail`                                                            | Will work just like `dto` but will throw an exception if the response is considered "failed".                                                                                                         |
| `ok`, `successful`, `redirect`, `failed`, `clientError`, `serverError` | Methods used to determine if a request was successful or not based on status code. The `failed` method [can be customised](handling-failures.md#customising-when-saloon-thinks-a-request-has-failed). |
| `onError`                                                              | Allows you to define a callback if the response is considered "failed".                                                                                                                               |
| `throw`                                                                | Will throw an exception if the response is considered "failed".                                                                                                                                       |
| `isCached`                                                             | Will return `true` if the response is cached.                                                                                                                                                         |
| `isFaked`                                                              | Will return `true` if the response was created from a `FakeResponse` class.                                                                                                                           |
| `isMocked`                                                             | Will return `true` if the response was created from a `MockResponse` class.                                                                                                                           |

### Handling asynchronous responses

When using concurrent requests/pooling or `sendAsync` , Saloon will respond with a `GuzzleHttp\Promise\PromiseInterface.` The promise will contain a `Response` a class described above. When the request fails, Saloon will not use the `then` method but return an instance of `RequestException`in the `otherwise` block.

```php
<?php

use Saloon\Http\Response;

$forge = new ForgeConnector('api-token');
$promise = $forge->sendAsync(new GetServersRequest);

$promise
    ->then(function (Response $response) {
        // Handle successful response
    })
    ->otherwise(function (Exception $exception) {
        // Handle failed request
    });
```

### Custom responses

Sometimes you may want to use your response class. This is useful if you want to add your methods or overwrite Saloon's response methods. Saloon allows you to overwrite the response at a connector level for all requests or at a per-request level for a granular response.

You may extend the `Saloon\Http\Response` class or provide your own implementation with the `Saloon\Http\Response` class. You may use the `HasResponseHelpers` middleware when making your own implementation to save defining every method.

#### Using the response property

The simplest way of registering a custom response is to use the `$response` property on either the connector or request.

{% tabs %}
{% tab title="Connector" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    // {...}
    
<strong>    protected ?string $response = CustomResponse::class;
</strong>}
</code></pre>
{% endtab %}

{% tab title="Request" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
<strong>    protected ?string $response = CustomResponse::class;
</strong>}
</code></pre>
{% endtab %}
{% endtabs %}

#### Using the resolveResponseClass method

When you need a more advanced way to define a custom response, use the `resolveResponseClass` method on either the connector or request.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    // {...}
    
    public function resolveResponseClass(): string
    {
        return CustomResponse::class;
    }
}
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
    public function resolveResponseClass(): string
    {
        return CustomResponse::class;
    }
}
```
{% endtab %}
{% endtabs %}
