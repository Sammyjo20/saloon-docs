# 🛩 Senders

Under the hood, Saloon uses a `Sender` class to send a fully built request to the web. Saloon uses the `GuzzleSender` as its default sender as Guzzle provides lots of useful functionality out of the box and is the most popular HTTP client for PHP. If you are interested to see how the sender looks, [click here to see the GuzzleSender code](https://github.com/saloonphp/saloon/blob/v3/src/Http/Senders/GuzzleSender.php).

### Creating your own sender

While Saloon's default sender is the `GuzzleSender`, you are free to create your own and change the default sender that your connector uses.&#x20;

#### Sender Contract

Saloon has a `Sender` interface which defines the structure that a sender must have. You must implement this contract for Saloon to accept the sender that you are creating.&#x20;

```php
<?php

use Saloon\Http\Response;
use Saloon\Http\PendingRequest;
use Saloon\Data\FactoryCollection;
use GuzzleHttp\Promise\PromiseInterface;

interface Sender
{
    /**
     * Get the factory collection
     */
    public function getFactoryCollection(): FactoryCollection;

    /**
     * Send the request synchronously
     */
    public function send(PendingRequest $pendingRequest): Response;

    /**
     * Send the request asynchronously
     */
    public function sendAsync(PendingRequest $pendingRequest): PromiseInterface;
}
```

The interface contains three methods.

* **getFactoryCollection** - This method expects a `FactoryCollection` DTO to be returned. This `FactoryCollection` DTO should contain all of the PSR-17 factories so that Saloon can build up requests, URIs and streams.
* **send** - This method is called when a request is being sent synchronously. You must return a Saloon `Response` class.
* **sendAsync** - This method is called when a request is being sent asynchronously. You must return a `PromiseInterface`.&#x20;

#### Suggested PSR-17 Factory Libraries

Saloon builds the PSR-7 request class internally based on the factories that you define in the `FactoryCollection` DTO. These PSR-17 factories are often provided by the HTTP client like Guzzle but sometimes they may not be provided. You can install any library you choose but there are two which we recommend:

* [**guzzlehttp/psr7**](https://github.com/guzzle/psr7) - Guzzle's implementation of PSR-7 and PSR-17
* [**nyholm/psr7**](https://github.com/Nyholm/psr7) - A popular, lightweight PSR-7 and 17 implementation

#### MultipartBodyFactory

Saloon is now also responsible for building the data stream from the various request body types that a request uses (JSON, String, Multipart). Saloon will use the `StreamFactory` provided in your `FactoryCollection` but there isn't an implementation for building `multipart/form-data` bodies. You will need to make your own `MultipartBodyFactory` implementation to build the multipart streams.

Guzzle has a great multipart builder, but another great alternative is the [multipart builder created by PHP-HTTP](https://github.com/php-http/multipart-stream-builder).

#### HTTP Client Instances

The sender instance is kept alive for the entire connector's lifetime so that the same sender is used to send all requests. This is required because Guzzle's HTTP client cannot be destructed if you are using request concurrency. It's recommended that your HTTP client is created when the sender is constructed and kept as a property on the sender.

#### Sending Requests

You should make sure to use the PSR-7 request provided by the `PendingRequest` class to send the request - as this class is the final class built by Saloon. You can create this by calling the `createPsrRequest` method.&#x20;

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\PendingRequest;

class CustomSender implements Sender
{
    public function send(PendingRequest $pendingRequest): Response
    {
<strong>        $psrRequest = $pendingRequest->createPsrRequest();
</strong>        
        // Send your request
    }
}
</code></pre>

{% hint style="warning" %}
The PSR-7 request is not cached on the `PendingRequest` so you should only call `createPsrRequest` once in the request lifecycle.
{% endhint %}

#### Responses

When creating your response, you should use the `$pendingRequest->getResponseClass()` method as this contains the response class that needs to be constructed. You should create the response with the `$responseClass::fromPsrResponse()` method as this is a common static method used by all responses, even if someone has made a custom response.

### Using your custom sender

Once you have created a custom sender, you must overwrite the protected `defaultSender` method on your connector. This method must return an instance of your sender. Now when you send a request using this connector it will use your custom sender.

```php
<?php

use Saloon\Contracts\Sender;

class ForgeConnector extends Connector
{
    // {...}
    
    protected function defaultSender(): Sender
    {
        return new CustomSender;
    }
}
```
