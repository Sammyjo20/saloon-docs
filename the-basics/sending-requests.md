# 🚀 Sending Requests

### Getting Started

To start sending requests, instantiate your connector class and request class and use the `send` or `sendAsync` methods. When using the `send` method. you will receive a `Response` class.

```php
<?php

$connector = new ForgeConnector('api-token');
$response = $connector->send(new GetServersRequest);

```

### Sending requests in one line

Reducing the number of lines in your code can help with readability. You can use the `make` static method on the connector or request to instantiate the object without using "new". Note - this means you don't have a reusable connector instance.

```php
<?php

ForgeConnector::make('api-token')->send(new GetServersRequest);
```

### Request Properties

You may also overwrite any headers, query parameters, HTTP client config and request body on the connector or request. Read through the sections above for all the methods on the request property methods.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

$connector = new ForgeConnector('api-token');

// All requests sent will have the header and query parameter applied

$connector->headers()->add('X-Custom-Header', 'Hello'!);
$connector->query()->add('page', 5);
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

$connector = new ForgeConnector('api-token');
$request = new GetServersRequest;

// The single request will have the additional header and query parameter.

$request->headers()->add('X-Custom-Header', 'Hello'!);
$request->query()->add('page', 5);

$response = $connector->send($request);
```
{% endtab %}
{% endtabs %}

### Asynchronous Requests

Saloon supports asynchronous requests out of the box. Use the `sendAsync` method, and you will receive an instance of `PromiseInterface`. Saloon uses Guzzle's Promises library, which uses the A+ standard. [Click here to read more](https://github.com/guzzle/promises).

```php
<?php

$connector = new ForgeConnector('api-token');
$promise = $connector->sendAsync(new GetServersRequest);

$promise
   ->then(function (Response $response) {
      // Handle Response
   })
   ->otherwise(function (RequestException $exception) {
        // Handle Exception
    });
```

{% hint style="info" %}
Saloon supports all the features Guzzle offers for asynchronous requests, including unwrapping promises and request pooling for high-performance API calls. [Click here to learn more.](../digging-deepeer/concurrency-and-pools.md)
{% endhint %}

### Sending Solo Requests

Please make sure to read the section on [solo requests](../digging-deepeer/solo-requests.md) first to configure your request. You can send solo requests directly. You have the following methods available on the solo request.

* send(MockClient $mockClient = null)
* sendAsync(MockClient $mockClient = null)
* createPendingRequest
* connector

```php
<?php

$request = new GetServersRequest;
$response = $request->send();
```

### Sending requests without instantiating the connector

With previous versions of Saloon, you could send a request directly without having to use a connector to send a request. While sending requests through the connector has many benefits, you may wish to add this feature with the `HasConnector` trait on your request.

#### Adding the trait

Once you have added the trait to your request, make sure to add the `connector` protected property and define your connector class. You may instead extend the `resolveConnector` method if you need a more advanced solution.

{% tabs %}
{% tab title="Connector Property" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Sammyjo20\Saloon\Http\Request;
use Saloon\Traits\Request\HasConnector;
use App\Http\Integrations\LaravelForge\ForgeConnector;

class GetServersRequest extends Request
{
<strong>    use HasConnector;
</strong><strong>    
</strong><strong>    protected string $connector = ForgeConnector::class;
</strong>
    protected string $method = 'GET';

    public function resolveEndpoint(): string
    {
        return '/servers';
    }
}
</code></pre>

{% hint style="warning" %}
When defining a connector with a property, you must not have any constructor properties on your connector.
{% endhint %}
{% endtab %}

{% tab title="Using resolveConnector" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Sammyjo20\Saloon\Http\Request;
use Saloon\Traits\Request\HasConnector;
use App\Http\Integrations\LaravelForge\ForgeConnector;

class GetServersRequest extends Request
{
<strong>    use HasConnector;
</strong>
    protected string $method = 'GET';
    
<strong>    protected function resolveConnector(): Connector
</strong><strong>    {
</strong><strong>        return new ForgeConnector;
</strong><strong>    }
</strong>
    public function resolveEndpoint(): string
    {
        return '/servers';
    }
}
</code></pre>
{% endtab %}
{% endtabs %}

#### Sending requests

Now you can use the following methods on your request.

* send(MockClient $mockClient = null)
* sendAsync(MockClient $mockClient = null)
* createPendingRequest
* connector

```php
<?php

$request = new GetServersRequest;
$response = $request->send();
```
