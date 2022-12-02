# 🚀 Sending Requests

### Getting Started

To start sending requests, instantiate your connector class and request class and use the `send` or `sendAsync` methods. When using the `send` method. you will receive a `Response` class.

```php
<?php

$connector = new ForgeConnector('api-token');
$response = $connector->send(new GetServersRequest);

```

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
$promise = $connector->send(new GetServersRequest);

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
