# 🔎 Debugging

While building API integrations, sometimes you will send a request and the API will return an error. This could be due to sending the request badly or incorrect data. Sometimes debugging the request can be difficult - so Saloon has some helpers to resolve this.

### Prerequisites

Saloon's default debugging driver uses [Symfony's Var Dumper](https://github.com/symfony/var-dumper) library. If you do no have it installed, you can install it below.

```
composer require symfony/var-dumper
```

{% hint style="info" %}
Laravel already includes this library.
{% endhint %}

### Debugging Request & Response

The simplest way to debug a request and response is to use the `debug()` method on your connector before sending a request. This will output an easy-to-understand array of the request and the response.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$connector = new ForgeConnector;

<strong>$connector->debug();
</strong>
$connector->send($request);
</code></pre>

You can provide the `die` argument if you would like to terminate the application after receiving the response.

```php
$connector->debug(die: true);
```

This will provide an output in your terminal/browser like this:

```
Saloon Request (UserRequest) -> array:6 [
  "connector" => "Saloon\Tests\Fixtures\Connectors\TestConnector"
  "request" => "Saloon\Tests\Fixtures\Requests\UserRequest"
  "method" => "GET"
  "uri" => "https://tests.saloon.dev/api/user"
  "headers" => array:2 [
    "Host" => "tests.saloon.dev"
    "Accept" => "application/json"
  ]
  "body" => ""
]

Saloon Response (UserRequest) -> array:3 [
  "status" => 200
  "headers" => []
  "body" => "{"name":"Sam"}"
]
```

{% hint style="warning" %}
This output will show the request just before it is sent to the sender. If the sender (like Guzzle) adds any additional headers or changes the request, these changes will not be displayed.
{% endhint %}

### Separate Debuggers

You may also use `debugRequest` and `debugResponse` independently if you would like to debug just the request or response respectively.

```php
<?php

$connector->debugRequest(); // $connector->debugRequest(die: true);

$connector->debugResponse(); // $connector->debugResponse(die: true);
```

### Custom Debugger Handlers

You may also provide a closure to the `debugRequest` and `debugResponse` methods if you would like to write your own debugging implementations.

```php
<?php

use Saloon\Http\Response;
use Saloon\Http\PendingRequest;
use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

// ...

$connector->debugRequest(function (PendingRequest $pendingRequest, RequestInterface $psrRequest) {
    ray($psrRequest);
});

$connector->debugResponse(function (Response $response, ResponseInterface $psrResponse) {
    ray($psrResponse);
});
```

