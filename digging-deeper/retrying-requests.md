# 🎯 Retrying Requests

Sometimes you may use APIs that are unreliable or require you to retry multiple times before a request is successful. Saloon has a global retry system that can be configured to automatically retry failed requests.

### Configuring our connector

Let's say that we have an API which isn't reliable. We can define a public `$tries` property on our connector which defines how many times a request will be sent if it fails.

<pre class="language-php"><code class="lang-php">class ForgeConnector extends Connector
{
<strong>    public ?int $tries = 3;
</strong>}
</code></pre>

Now when we send any request through this connector, if it fails it will be retried.

```php
$forgeConnector = new ForgeConnector;

$response = $forgeConnector->send(new UnreliableRequest);
```

If we get to attempt three and it still fails, Saloon will throw a `FatalRequestException` (for connection errors) or a `RequestException` for typical HTTP errors.

{% hint style="warning" %}
The retry functionality will only work when sending requests synchronously. The retry logic will not work when using `sendAsync` or pools because Saloon needs to wait for a response to come back to determine if it needs to be retried.
{% endhint %}

#### What does Saloon consider a failed request?

By default, if the API couldn't be connected to or if the response status code is either a 4xx or 5xx, then Saloon will consider it a failed request. You can change this by extending the `hasRequestFailed` method on your connector. [Click here to learn more.](../the-basics/handling-failures.md#customising-when-saloon-thinks-a-request-has-failed)

### Intervals

You can also configure an interval in **milliseconds** that Saloon should wait between retries. You can configure an interval with the `$retryInterval` property.

<pre class="language-php"><code class="lang-php">class ForgeConnector extends Connector
{
    public ?int $tries = 3;
    
<strong>    public ?int $retryInterval = 1000;
</strong>}
</code></pre>

#### Exponential Backoff

You can also use exponential backoff when retrying which will double the retry interval after each unsuccessful attempt. This is useful as it can be less strenuous on the API if it is experiencing issues.

<pre class="language-php"><code class="lang-php">class ForgeConnector extends Connector
{
    public ?int $tries = 3;
    
    public ?int $retryInterval = 500;
    
<strong>    public ?bool $useExponentialBackoff = true;
</strong>}
</code></pre>

### Exceptions

By default, when all attempts have been exceeded, Saloon will either throw a `FatalRequestException` or a `RequestException`. You can choose to disable this behaviour and return the last response that Saloon received instead.

If the exception was `FatalRequestException` then the exception will still be thrown as we wouldn't have a response to return.&#x20;

<pre class="language-php"><code class="lang-php">class ForgeConnector extends Connector
{
    public ?int $tries = 3;
    
<strong>    public ?bool $throwOnMaxTries = false;
</strong>}
</code></pre>

### Customising the request before the next retry

In some situations, you might want to change the next request to improve the chances of the next retry being successful. You can overwrite the `handleRetry` method on the connector or request and use it to customise the `Request` class. You must also return a boolean in this method to tell Saloon if the retry should commence.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Exceptions\RequestException;
use Saloon\Http\Auth\TokenAuthenticator;
use Saloon\Exceptions\FatalRequestException;

class ForgeConnector extends Connector
{
    public ?int $tries = 3;
    
    public function handleRetry(FatalRequestException|RequestException $exception, Request $request): bool
    {
        if ($exception instanceof RequestException &#x26;&#x26; $exception->getResponse()->status() === 401) {
<strong>            $request->authenticate(new TokenAuthenticator($this->getNewToken()));
</strong>        }
        
        return true;
    }
}
</code></pre>

### Using the sendAndRetry method

{% hint style="warning" %}
This method has been deprecated since Saloon v3.6.4
{% endhint %}

Saloon also has a `sendAndRetry` method which can be used on the fly on any connector or request without needing to configure anything. This method has the same functionality in the format of a method.

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
    useExponentialBackoff: true,
);
```
