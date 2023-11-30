# 🔎 Debugging

When building API integrations with Saloon, sometimes you will need to debug the request and the response. Since Saloon has some advanced logic like authenticators, plugins, middleware and merging properties from the connector - sometimes just dumping the `$request` class doesn't show what you expect. Saloon has some simple built-in helper methods that allow you to debug each request or response.&#x20;

### Debugging Requests

Saloon has a handy `debugRequest` method which can be used on your connector or request. This method can be called in your application and you can provide a closure to see what the `PendingRequest` class and the PSR request class looks like. This closure is executed as the very last step before a request is sent, so you will be able to see every header, config argument and option added by plugins, middleware and your own application.

Simply register the `debugRequest` method before sending your request and the closure will be executed before the request is sent.

```php
<?php

use Saloon\Http\PendingRequest;
use Psr\Http\Message\RequestInterface;

$forgeConnector = new ForgeConnector;

$forgeConnector->debugRequest(
    function (PendingRequest $pendingRequest, RequestInterface $psrRequest) {
        dd($pendingRequest->headers()->all(), $psrRequest);
    }
);

$forgeConnector->send(new GetServersRequest);
```

In the example above, I am using a common method `dd()` (dump and die) to see what the headers look like and what my PSR request looks like.

### Debugging Responses

Debugging responses work in a very similar way to debugging requests. You can call the `debugResponse` method in your application before you send a request and when the response comes back, the debugging closure is invoked. The closure passes in a `Response` class.

You may not need to debug responses like this, as sometimes you can just dump them `$response` after sending the request, but if you use a plugin like `AlwaysThrowOnError` there's a chance that the exception is thrown before the response is handed back to your application.

```php
<?php

use Saloon\Http\Response;

$forgeConnector = new ForgeConnector;

$forgeConnector->debugResponse(function (Response $response) {
    dd($response->body());
});

$forgeConnector->send(new GetServersRequest);
```
