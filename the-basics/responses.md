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

{% hint style="danger" %}
By default, Saloon will not throw an exception if a synchronous request fails. [Refer to the handling failures section for handling errors.](handling-failures.md)
{% endhint %}

### Available Methods

| Method                      | Description                                                                                                                             |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| body                        | Returns the HTTP body as a string                                                                                                       |
| stream                      | Returns the HTTP body as a stream                                                                                                       |
| headers                     | Returns all the headers. You can interact with them just like you do with requests/connectors.                                          |
| header                      | Returns a single header from the response.                                                                                              |
| status                      | Return the HTTP status messages                                                                                                         |
| getPsrResponse              | Returns a PSR-compatible response                                                                                                       |
| getPendingRequest           | Returns the PendingRequest created to send the request, containing everything sent like headers, body, and HTTP client config.          |
| getRequest                  | Returns the original request class that was used to send the request.                                                                   |
| json                        | Retrieves a JSON response body and json\_decodes it into an array.                                                                      |
| object                      | Retrieves a JSON response body and json\_decodes it into an object.                                                                     |
| xml                         | Retrieves the response body and creates a SimpleXmlElement.                                                                             |
| collect                     | Retrieves a JSON response body and json\_decodes it into a Laravel collection. **Requires illuminate/collections to be installed.**     |
| dto                         | Converts the response into a data-transfer object. You must define your DTO first, [click here to read more.](data-transfer-objects.md) |
| successful                  | Returns true if the response status is between 200 and 300.                                                                             |
| ok                          | Returns true if the response status is 200.                                                                                             |
| redirect                    | Returns true if the response status is between 300 and 400.                                                                             |
| serverError                 | Returns true if the response status is >= 500.                                                                                          |
| clientError                 | Returns true if the response status is between 400 and 500.                                                                             |
| failed                      | Determines if a request has failed. By default, it will return true if the response status is not 2xx.                                  |
| onError                     | Allows you to define a callback if the response is considered "failed".                                                                 |
| toException                 | Creates an exception if the response is considered "failed. [Click here to read more about handling failures](handling-failures.md)     |
| throw                       | Will throw an exception if the response is considered "failed".                                                                         |
| isCached                    | Denotes if the response is cached. Only used when using the Saloon caching plugin.                                                      |
| isMocked                    | Denotes if the response has been mocked.                                                                                                |
| isSimulated                 | Denotes if the response was "simulated", like when a MockResponse was used.                                                             |
| getSimulatedResponsePayload | When the response is simulated, get the underlying SimulatedResponsePayload class.                                                      |
| getSenderException          | Get the sender exception if a request failed.                                                                                           |
| getRawResponse              | Get the raw response provided from the sender.                                                                                          |
| \_\_toString                | Returns the HTTP body as a string                                                                                                       |

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

You may extend the `Saloon\Http\Response` class or provide your own implementation with the `Saloon\Http\Response` interface. You may use the `HasResponseHelpers` middleware when making your own implementation to save defining every method.

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
