# 🎯 Retrying Requests

Sometimes you may deal with APIs that fail frequently or require you to retry multiple times before a request is successful. Saloon has a global retry system that allows you to send a request and retry multiple times.&#x20;

There are a few ways in which you can configure the retry functionality. You can either configure it on a connector or request class, or you can use the `sendAndRetry` method as and when you need to. It's recommended to configure it on a class level as Saloon is all about keeping everything in one place.

### Configuring our connector

Let's say that we have an API that no matter what requests we make, isn't reliable. We'll configure some retry functionality to work across every request. You can also configure the same properties on a request basis if it's just one particular request that isn't reliable.&#x20;

The only property we need to enable the retry functionality is the `$retry` property. Here you can define an integer denoting the number of times a request should be retried. We'll configure our connector to retry three times before failing.

<pre class="language-php"><code class="lang-php">&#x3C;?php

class ForgeConnector extends Connector
{
<strong>    public ?int $tries = 3;
</strong>}
</code></pre>

Now when we send any request through this connector, if it fails it will be retried. It's as simple as that! Continue reading to see some more advanced features of Saloon's global retry system.

```php
<?php

$forgeConnector = new ForgeConnector;

$response = $forgeConnector->send(new UnreliableRequest);
```

If we get to attempt three and it still fails, Saloon will throw a `FatalRequestException` (for connection errors) or a `RequestException` for typical HTTP errors.

{% hint style="warning" %}
The retry functionality will only work when sending requests synchronously. The retry logic will not work when using `sendAsync` or pools because Saloon needs to wait for a response to come back to determine if it needs to be retried.
{% endhint %}

#### What does Saloon consider a failed request?

So far we've mentioned that "when a request fails" it will be retried - but how does Saloon consider a request to be failed? By default, if the status code is either a 4xx or 5xx, then Saloon will consider it failed, however, you can configure this. Have a read through the [Handling Failures](../the-basics/handling-failures.md#customising-when-saloon-thinks-a-request-has-failed) section of the documentation for more.

### Configuring Intervals

You can also configure an optional interval in milliseconds that Saloon should wait before requests. This is useful to avoid being rate-limited or overloading the server while it is already down. You can configure an interval with the `$retryInterval` property. For example, let's say I wanted to wait 500ms between each request.

<pre class="language-php"><code class="lang-php">&#x3C;?php

class ForgeConnector extends Connector
{
    public ?int $tries = 3;
    
<strong>    public ?int $retryInterval = 500;
</strong>}
</code></pre>

### Customizing Exception Behaviour

By default, when all attempts have been exceeded, Saloon will either throw a `FatalRequestException` or a `RequestException`. You can choose to disable this behaviour and instead return the last response that Saloon received. If the exception was `FatalRequestException` then the exception will still be thrown as we wouldn't have a response to return.&#x20;

<pre class="language-php"><code class="lang-php">&#x3C;?php

class ForgeConnector extends Connector
{
    public ?int $tries = 3;
    
<strong>    public ?bool $throwOnMaxTries = false;
</strong>}
</code></pre>

### Customising when the request before the next retry

There are situations when you might want to change the next request to improve the chances of the next retry being successful. You can overwrite the `handleRetry` method on the connector or request and use it to customise the `Request` class. You must also return a boolean in this method to tell Saloon if the retry should commence.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Exceptions\FatalRequestException;
use Saloon\Exceptions\RequestException;

class ForgeConnector extends Connector
{
    public ?int $tries = 3;
    
    public function handleRetry(FatalRequestException|RequestException $exception, Request $request): bool
    {
        if ($exception instanceof RequestException &#x26;&#x26; $exception->getResponse()->status() === 401) {
<strong>            $request->withTokenAuth($this->getNewToken());
</strong>        }
        
        return true;
    }
}
</code></pre>

### Using the sendAndRetry method

Saloon also has a `sendAndRetry` method which can be used on the fly on any connector or request without needing to configure anything. This method has exactly the same functionality in the format of a method.

```php
<?php

$forgeConnector = new ForgeConnector;

$response = $forgeConnector->sendAndRetry(
    request: new UnreliableRequest,
    tries: 3,
    interval: 500,
    handleRetry: function (FatalRequestException|RequestException $exception, Request $request) {
        //
    },
    throw: true,
);
```

{% hint style="info" %}
While the above method is a little more convenient, it's recommended to configure your retry functionality inside of the request or connector.
{% endhint %}
