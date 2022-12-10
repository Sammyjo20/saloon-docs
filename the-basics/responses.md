# 📡 Responses

Depending on how you sent your request (synchronous/asynchronous) you will either receive an instance of `Response` or a `PromiseInterface.`

### Handling synchronous responses

By default, Saloon will return an instance of `Saloon\Contracts\Response`. this is the default response and is used by the GuzzleSender. This response class contains many helpful methods for interacting with your HTTP response.

```php
<?php

$connector = new ForgeConnector('api-token');
$response = $connector->send(new GetServersRequest);

$body = $response->body();
$decodedBody = $response->json();
```

{% hint style="warning" %}
By default, Saloon will not throw an exception if a synchronous request fails. [Refer to the handling failures section for handling errors.](handling-failures.md)
{% endhint %}

### Available Methods

| Method                      | Description                                                                                                                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| body                        | Returns the HTTP body as a string                                                                                                                                                             |
| stream                      | Returns the HTTP body as a stream                                                                                                                                                             |
| headers                     | Returns all the headers. You can interact with them just like you do with requests/connectors.                                                                                                |
| header                      | Returns a single header from the response.                                                                                                                                                    |
| status                      | Return the HTTP status messages                                                                                                                                                               |
| getPsrResponse              | Returns a PSR-compatible response                                                                                                                                                             |
| getPendingRequest           | Returns the PendingRequest created to send the request, containing everything sent like headers, body, and HTTP client config.                                                                |
| getRequest                  | Returns the original request class that was used to send the request.                                                                                                                         |
| json                        | Retrieves a JSON response body and json\_decodes it into an array.                                                                                                                            |
| object                      | Retrieves a JSON response body and json\_decodes it into an object.                                                                                                                           |
| xml                         | Retrieves the response body and creates a SimpleXmlElement.                                                                                                                                   |
| collect                     | Retrieves a JSON response body and json\_decodes it into a Laravel collection. **Requires illuminate/collections to be installed.**                                                           |
| dto                         | Converts the response into a data-transfer object. You must define your DTO first, [click here to read more.](../digging-deepeer/data-transfer-objects.md)                                    |
| successful                  | Returns true if the response status is between 200 and 300.                                                                                                                                   |
| ok                          | Returns true if the response status is 200.                                                                                                                                                   |
| redirect                    | Returns true if the response status is between 300 and 400.                                                                                                                                   |
| serverError                 | Returns true if the response status is >= 500.                                                                                                                                                |
| clientError                 | Returns true if the response status is between 400 and 500.                                                                                                                                   |
| failed                      | Determines if a request has failed. By default, it will return true if the response status is not 2xx. [To customise this behaviour, read more about handling failures](handling-failures.md) |
| onError                     | Allows you to define a callback if the response is considered "failed".                                                                                                                       |
| toException                 | Creates an exception if the response is considered "failed. [ Click here to read more about handling failures](handling-failures.md)                                                          |
| throw                       | Will throw an exception if the response is considered "failed".                                                                                                                               |
| isCached                    | Denotes if the response is cached. Only used when using the Saloon caching plugin.                                                                                                            |
| isMocked                    | Denotes if the response has been mocked.                                                                                                                                                      |
| isSimulated                 | Denotes if the response was "simulated", like when a MockResponse was used.                                                                                                                   |
| getSimulatedResponsePayload | When the response is simulated, get the underlying SimulatedResponsePayload class.                                                                                                            |
| getSenderException          | Get the sender exception if a request failed.                                                                                                                                                 |
| getRawResponse              | Get the raw response provided from the sender.                                                                                                                                                |
| \_\_toString                | Returns the HTTP body as a string                                                                                                                                                             |

### Handling asynchronous responses

When using concurrent requests/pooling or `sendAsync` , Saloon will respond with a `GuzzleHttp\Promise\PromiseInterface.` The result will be a `Response` a class described above. When the request fails, Saloon will not use the `then` method but return an instance of `RequestException`in the `otherwise` block.

```php
<?php

use Saloon\Contracts\Response;

$connector = new ForgeConnector('api-token');
$promise = $connector->sendAsync(new GetServersRequest);

$promise
    ->then(function (Response $response) {
        // Handle successful response
    })
    ->otherwise(function (RequestException|FatalRequestException $exception) {
        // Handle failed request
    });
```

### Custom responses

* You must extend the sender's response since the sender is responsible for instantiating the object
