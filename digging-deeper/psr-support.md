# ⛩ PSR Support

When you send a request, Saloon will construct a [PSR-7](https://www.php-fig.org/psr/psr-7/) request behind the scenes and pass it to the HTTP client. Saloon also uses [PSR-17](https://www.php-fig.org/psr/psr-17/) factories defined by the sender to create the URI (the full URL) and streams for the request body. When you make a request to Saloon, the response will contain a PSR-7 request and response pair.&#x20;

```php
<?php

$response = $forgeConnector->send(new GetServersRequest);

$psrResponse = $response->getPsrResponse();
$psrRequest = $response->getPsrRequest();
```

This design decision was made because the PSR-7 and 17 standards are very well-known and have been implemented by many popular HTTP clients. While Saloon uses Guzzle as the default way to send requests, if Guzzle suddenly becomes abandoned - Saloon can use one of many other HTTP clients without needing an entire rebuild from scratch. While Guzzle is amazing, and we'd never want it to go away - making Saloon independent of the HTTP client was a big step toward future-proofing the library.

Another significant benefit to having PSR support is a better experience for you, the developer. With previous versions, Saloon heavily relied on Guzzle's config options to build the PSR-7 request internally, which meant that debugging was difficult because you never really knew what was sent to the third party. With version three, you can see the PSR request right before it is sent. If you want to learn more about debugging the PSR request, [click here](debugging.md).

### Modifying the PSR-7 Request

There may be a situation where you need to modify the PSR-7 request before it's sent to the HTTP client. You can use a special hook on either the connector or the request. This hook provides you with the built PSR-7 request that you can modify and return. To use this hook, extend the `handlePsrRequest` method on either your connector or request.

If you use the method on the connector - the method will be run for every request sent through the connector. The method will give you access to both the PSR request and the Saloon `PendingRequest` if you need to use it for additional information, like the Saloon request class.&#x20;

```php
<?php

use Saloon\Http\PendingRequest;
use Psr\Http\Message\RequestInterface;

class ForgeConnector extends Connector
{
    public function handlePsrRequest(RequestInterface $request, PendingRequest $pendingRequest): RequestInterface
    {
        $request->withHeader('X-API-Key', 'Howdy!');
        
        return $request;
    }
}
```

{% hint style="warning" %}
You may find it easier to modify the `Connector`,  `Request`, or `PendingRequest` classes instead of the underlying PSR-7 request. You should be cautious when using this hook as there are many nicer, higher-level APIs in Saloon to modify the request.
{% endhint %}
