# 🚀 Sending Requests

### Getting Started

To start sending requests, instantiate your connector class and request class and use the `send` or `sendAsync` methods. When using the `send` method. you will receive a `Response` class, but when using `sendAsync` , you will receive an instance of `PromiseInterface`. Read more below to learn more about asynchronous requests.

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
```
{% endtab %}
{% endtabs %}

* Sending requests normally
* Through connectors
* Using sendAsync
