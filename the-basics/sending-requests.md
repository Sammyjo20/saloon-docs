# 🚀 Sending Requests

To start sending requests, instantiate your connector and use the `send` method. This method accepts a request class.

```php
<?php

$forge = new ForgeConnector;
$request = new GetServersRequest;

$response = $forge->send($request);
```

### Asynchronous Requests

To send an asynchronous request, use the `sendAsync` method, and you will receive an instance of `PromiseInterface`. You should define a `then` and `otherwise` method to receive the response or exception.&#x20;

```php
<?php

$forge = new ForgeConnector('api-token');
$promise = $forge->sendAsync(new GetServersRequest);

$promise
   ->then(function (Response $response) {
      // Handle Response
   })
   ->otherwise(function (RequestException $exception) {
      // Handle Exception
   });
```

{% hint style="info" %}
Saloon uses Guzzle's Promises library. [Click here to learn more](https://github.com/guzzle/promises)
{% endhint %}
