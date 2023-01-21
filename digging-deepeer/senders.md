# 🛩 Senders

### Introduction

Under the hood, Saloon uses a "Sender" to send a fully built pending request to the web. Saloon uses the GuzzleSender as its default sender as Guzzle provides lots of functionality out of the box and is well-tested and used among the PHP community.

#### Example Sender

Click here to see the [GuzzleSender's](https://github.com/Sammyjo20/Saloon/blob/v2/src/Http/Senders/GuzzleSender.php) code as an example of a sender.

### Creating your own sender

While Saloon's default sender is the GuzzleSender, Saloon is completely sender agnostic and different senders/HTTP clients can be built to be used by it. If you would like to build a sender for a different HTTP client or customise the Guzzle sender you are more than welcome to.

#### Sender Contract

Saloon has a `Sender` interface which defines the structure that a sender must be in. You must implement this contract for Saloon to accept the sender that you are creating. The contract contains just one method: `sendRequest.` This method will receive an instance of `PendingRequest` which is the object that contains everything ready to be sent. At this point, plugins and request middleware has run and the request is ready to be sent.

The `sendRequest` method must either return a `Saloon\Contracts\Response` or an instance of `GuzzleHttp\Promise\PromiseInterface`. These response types will change depending if the request was sent asynchronously or not.

```php
<?php

declare(strict_types=1);

namespace Saloon\Contracts;

use GuzzleHttp\Promise\PromiseInterface;

interface Sender
{
    /**
     * Send the request.
     *
     * @param PendingRequest $pendingRequest
     * @param bool $asynchronous
     * @return Response|PromiseInterface
     */
    public function sendRequest(PendingRequest $pendingRequest, bool $asynchronous = false): Response|PromiseInterface;
}
```

#### HTTP Client Instances

The Sender instance is kept alive against the connector instance so that the same sender is used to send all requests. This is required because Guzzle's HTTP client cannot be destructed if you are using request concurrency. It's recommended that your HTTP client is created when the sender is constructed and kept as a property on the sender.&#x20;

#### Headers, Query Parameters and HTTP Config

You will be able to access all the headers, query parameters and HTTP config from the pending request instance you are given. You can use the methods provided on these resources to send the headers, query parameters and config to your HTTP client.

```php
<?php

$pendingRequest->headers();
$pendingRequest->query();
$pendingRequest->config();
```

#### Request Body

Saloon's pending request class contains a `body()` method which can return a different `BodyRepository` depending on the request body that was used. Saloon provides a `__toString()` method on the repository to make it easy to send it, however, Saloon does not send a Content-Type, so it will be either up to your sender or the developer to provide this as a header.&#x20;

Saloon also does not provide a `__toString()` method for multipart requests. You must handle multipart value objects yourself and convert them into a multipart body.

#### Responses

When creating your response, you should use the `$pendingRequest->getResponseClass()` method as this contains the response class that needs to be constructed. You should construct the response with the `$responseClass::fromPsrResponse()` method as this is a common static method used by all responses, even if someone has made a custom response.

### Using your custom sender

Once you have created a custom sender, you must overwrite the protected `defaultSender` method on your connector. This method must return an instance of your sender. Saloon will handle everything else for you.

```php
<?php

class ForgeConnector extends Connector
{
    // {...}
    
    protected function defaultSender(): Sender
    {
        return new CustomSender;
    }
}
```
