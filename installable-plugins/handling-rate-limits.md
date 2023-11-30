# ⛔ Handling Rate Limits

Handling rate limits with API integrations can be hard. Saloon has a first-party plugin that provides you with the tools you need to prevent rate limits and handle what happens if a rate limit is exceeded.

With this plugin, you are able to define various limits on a per-connector or request basis. You can also control if Saloon should throw an exception or sleep if a limit is reached. Saloon will keep track of how many requests are made and when a rate limit is hit, Saloon will prevent further requests on the connector/request until the rate limit has been lifted.

Saloon will even listen out for "429 Too Many Requests" responses and will automatically throw exceptions before your future requests are sent and the limit will be lifted based on the `Retry-After` header.

This plugin also comes with a Laravel Job middleware which you can use inside of your jobs to automatically release them back onto the queue if a rate limit has been reached.

```php
<?php

use Saloon\Http\Connector;
use Saloon\RateLimitPlugin\Limit;
use Saloon\Traits\Plugins\AcceptsJson;
use Saloon\RateLimitPlugin\Stores\RedisStore;
use Saloon\RateLimitPlugin\Traits\HasRateLimits;
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;

class SpotifyConnector extends Connector
{
    use AcceptsJson;
    use HasRateLimits;

    public function resolveBaseUrl(): string
    {
        return 'https://api.spotify.com/v1';
    }

    protected function resolveLimits(): array
    {
        return [
            Limit::allow(100)->everyMinute(),
            Limit::allow(1000)->everyDay(),
            Limit::allow(5000)->everyMonth(),
        ];
    }

    protected function resolveRateLimitStore(): RateLimitStore
    {
        $redis = new Redis;
        $redis->connect('127.0.0.1');
    
        return new RedisStore($redis);
    }
}
   
```

### Installation

You can install this plugin via Composer.

```
composer require saloonphp/rate-limit-plugin "^2.0"
```

### Getting Started

To use the plugin, add the `HasRateLimit` trait to your connector or request. If you have a connector, you should put it on the connector, but it can be put on an individual request if a specific endpoint has a different rate limit or if you are using solo requests. If you are using the trait on both your connector and request, the rate limits are combined together.

```php
use Saloon\Http\Connector;
use Saloon\RateLimitPlugin\Traits\HasRateLimits;

class SpotifyConnector extends Connector
{
    use HasRateLimits;
}
```

Next, you will be required to implement two methods: `resolveLimits` and `resolveRateLimitStore`. These methods allow you to define the limits that Saloon will keep track of and the store where the limit "hits" will be kept.

```php
use Saloon\Http\Connector;
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;
use Saloon\RateLimitPlugin\Stores\MemoryStore;
use Saloon\RateLimitPlugin\Traits\HasRateLimits;

class SpotifyConnector extends Connector
{
    use HasRateLimits;
    
    protected function resolveLimits(): array
    {
        return [];
    }

    protected function resolveRateLimitStore(): RateLimitStore
    {
        return new MemoryStore;
    }
}
```

### Stores

Here are the various stores that the rate-limiting plugin supports. You may also [create your own stores](https://github.com/Sammyjo20/saloon-docs/blob/v2/digging-deeper/broken-reference/README.md) if this library does not come with the one you need. Stores are used to keep track of how many requests have been sent through a given connector/request.

#### Available Stores

* In-Memory (Array)
* File
* Redis
* Predis
* PSR Cache Store
* Laravel Cache Store

#### Memory Store

The simplest store. This store is persisted on the current instance of the connector/request and all information is lost when the connector is destructed.

```php
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;
use Saloon\RateLimitPlugin\Stores\MemoryStore;

protected function resolveRateLimitStore(): RateLimitStore
{
    return new MemoryStore;
}
```

#### File Store

This store will use the local filesystem to store the limits. The only requirement for this store is for you to define the absolute path to a directory where you would like the limits to be stored.

```php
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;
use Saloon\RateLimitPlugin\Stores\FileStore;

protected function resolveRateLimitStore(): RateLimitStore
{
    return new FileStore('some/application/directory');
}
```

#### Redis Store

This store will use PHP's `Redis` extension to store the limits on a Redis database. You should pass the Redis configuration into this store.

```php
use Redis;
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;
use Saloon\RateLimitPlugin\Stores\RedisStore;

protected function resolveRateLimitStore(): RateLimitStore
{
    $client = new Redis;
    $client->connect('127.0.0.1');

    return new RedisStore($client);
}
```

#### Predis Store

Similar to the `RedisStore`, the `PredisStore` allows you to connect to Redis through the `predis/predis` PHP library.

```php
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;
use Saloon\RateLimitPlugin\Stores\PredisStore;

protected function resolveRateLimitStore(): RateLimitStore
{
    $client = new Predis\Client([
        'scheme' => 'tcp',
        'host'   => '10.0.0.1',
        'port'   => 6379,
    ]);

    return new PredisStore($client);
}
```

#### PSR Cache Store

This store supports any PSR-16 cache store provided by the `psr/simple-cache` library.

```php
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;
use Saloon\RateLimitPlugin\Stores\PsrStore;

protected function resolveRateLimitStore(): RateLimitStore
{
    return new PsrStore(new SomePsr16Store);
}
```

#### Laravel Cache Store

This store can only be used in a Laravel environment but allows you to use any of Laravel's cache disks.

```php
use Illuminate\Support\Facades\Cache;
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;
use Saloon\RateLimitPlugin\Stores\LaravelCacheStore;

protected function resolveRateLimitStore(): RateLimitStore
{
    return new LaravelCacheStore(Cache::store('redis'));
}
```

### Limits

While this plugin can detect if a 429 status occurs from a response, it's better to prevent your application from hitting rate limits than let them happen. This plugin provides an expressive `Limit` class which can be used to define different limits. You can define as many limits as you like, with various intervals.

#### Configuring Limits

Here is a simple example of a limit for an API which only allows 60 requests per minute, but has a daily limit of 1,000 API calls. There are many different limit intervals, as well as different ways you can instruct Saloon to handle the limit. There is no restriction on the number of limits you can have.

```php
use Saloon\RateLimitPlugin\Limit;

protected function resolveLimits(): array
{
    return [
        Limit::allow(60)->everyMinute(),
        Limit::allow(1000)->everyDay(),
    ];
}
```

#### Limit Intervals

There are various limit intervals which you can use on your limiter, ranging for seconds to up to the end of the month.

```php
use Saloon\RateLimitPlugin\Limit;

Limit::allow(60)->everySeconds(seconds: 5);
Limit::allow(60)->everyMinute();
Limit::allow(60)->everyFiveMinutes();
Limit::allow(60)->everyThirtyMinutes();
Limit::allow(60)->everyHour();
Limit::allow(60)->everySixHours();
Limit::allow(60)->everyTwelveHours();
Limit::allow(60)->everyDay();
Limit::allow(60)->everyDayUntil('8pm');
Limit::allow(60)->untilMidnightTonight();
Limit::allow(60)->untilEndOfMonth();
```

#### Custom Names

Sometimes you may need to add a name to your limiter. A good example of this is if you have separate API keys per user and therefore require a different API rate limit per user. You can add the `name()` method to your limit to specify a custom name for your limiter. Each limiter's name must be unique.

```php
protected function resolveLimits(): array
{
    return [
        Limit::allow(60)->everyMinute()->name('spotify-limit-user-' . $this->userId),
    ];
}
```

#### Custom Prefixes

Saloon will append the class name of the connector or a request as the prefix to the limiter name. For example SpotifyConnector:30\_every\_60. You may customise the prefix by extending the `getLimiterPrefix` method on your connector or request.

```php
protected function getLimiterPrefix(): ?string
{
    return 'spotify-user-' . $this->userId;
}
```

#### Custom Thresholds

You may want to specify the percentage threshold that Saloon should accept as the number of "hits" on a given limit. This is useful if you want to stay just under the real API limit while still defining the limit in the connector/request.

The threshold must be a number between 0 and 1 (e.g 0.8 = 80%)

```php
use Saloon\RateLimitPlugin\Limit;

protected function resolveLimits(): array
{
    return [
        Limit::allow(60, threshold: 0.8)->everyMinute(), // Will fail when at 80% capacity
    ];
}
```

#### Sleep

If would rather Saloon didn't throw an exception, you can use the `sleep` method when defining a limit. When using the sleep method, an exception won't be thrown. Instead, Saloon will wait the remaining number of seconds before a request is attempted again.

```php
use Saloon\RateLimitPlugin\Limit;

protected function resolveLimits(): array
{
    return [
        Limit::allow(60)->sleep(),
    ];
}
```

#### "429: Too Many Attempts" Detection

While it's recommended that you should define your limits above, Saloon will try to catch 429 "Too Many Attempts" errors from an API and will automatically mark a limit as "exceeded" if it sees this status. By default, Saloon will attempt to parse the `Retry-After` header to work out when a limit has been exceeded. If Saloon cannot calculate this, the limit will be released after 60 seconds.

You can customise this behaviour by overwriting the `handleTooManyAttempts` method.

```php
protected function handleTooManyAttempts(Response $response, Limit $limit): void
{
    if ($response->status() !== 429) {
        return;
    }

    $limit->exceeded(
        releaseInSeconds: RetryAfterHelper::parse($response->header('Retry-After')),
    );
}
```

Alternatively, you may choose to disable this functionality. You can do this by setting the `detectTooManyAttempts` property to `false` in your connector/request's constructor.

<pre class="language-php"><code class="lang-php">use Saloon\Http\Connector;
use Saloon\RateLimitPlugin\Stores\MemoryStore;
use Saloon\RateLimitPlugin\Traits\HasRateLimits;

class SpotifyConnector extends Connector
{
    use HasRateLimits;
    
    public function __construct()
    {
<strong>        $this->detectTooManyAttempts = false;
</strong>    }
}
</code></pre>

### Handling Rate Limits Being Exceeded

When a rate limit has been reached, Saloon will throw a `RateLimitReachedException`. This exception contains a `getLimit` method which may be used to see the limit that has thrown the exception and see the number of seconds to wait before a request can be sent again. This can be done with a simple try-catch approach or if you are using the provided Laravel job middleware, then you can instruct your jobs to wait until the limit has been lifted.

#### Try/Catch

As mentioned above, Saloon will throw an exception if a rate limit is reached or if the API returns a 429: Too Many Requests response. You could use a try/catch block to catch the exception and do something with the limit. For example, you may return an error to your users to let them know how long they need to wait - or you might retry the request later. If you are using Saloon in a context of a queued process, then you may want to retry the queued job in the future, from the remaining seconds.

```php
use Saloon\RateLimitPlugin\Exceptions\RateLimitReachedException;

$spotify = new SpotifyConnector;

try {
   $response = $spotify->send(new GetPlaylistRequest);
} catch (RateLimitReachedException $exception) {
    $seconds = $exception->getLimit()->getRemainingSeconds();
    
    // Return our users back to our application with a custom response that could be 
    // shown on the front end.

    return response("Too many requests to Spotify's API. Please try again in ${$seconds} seconds.");
}
```

#### Sleep

If would rather Saloon didn't throw an exception, you can use the `sleep` method when defining a limit. When using the sleep method, an exception won't be thrown. Instead, Saloon will wait the remaining number of seconds before a request is attempted again.

```php
use Saloon\RateLimitPlugin\Limit;

protected function resolveLimits(): array
{
    return [
        Limit::allow(60)->sleep(),
    ];
}
```

#### Laravel Job Middleware

If you are using Laravel, then this library comes with a [job middleware](https://laravel.com/docs/queues#job-middleware) that you can use. This job middleware will catch the `RateLimitReachedException` and automatically release your job back onto the queue with the remaining seconds added. Add this to the `middleware` method on your Laravel Job.

```php
use Saloon\RateLimitPlugin\Helpers\ApiRateLimited;
 
public function middleware(): array
{
    return [new ApiRateLimited];
}
```

{% hint style="info" %}
You may also wish to increase your job's tries when using this middleware in case the job needs to be retried multiple times.
{% endhint %}

### Creating your own store

You may create your own rate limit store by implementing the `RateLimitStore` interface.

```php
use Saloon\RateLimitPlugin\Contracts\RateLimitStore;

class CustomStore implements RateLimitStore
{
    /**
     * Get a rate limit from the store
     */
    public function get(string $key): ?string
    {
        //
    }

    /**
     * Set the rate limit in the store
     */
    public function set(string $key, string $value, int $ttl): bool
    {
        //
    }
}
```

### Disabling Rate Limiting

Sometimes you might want to disable rate limiting by default or even disable it on a per-connector basis. You can either use a property to disable the rate-limiting functionality by default, or you can use the `useRateLimitPlugin()` method to disable it on a per-instance basis.

<pre class="language-php"><code class="lang-php">use Saloon\Http\Connector;
use Saloon\RateLimitPlugin\Traits\HasRateLimits;

class SpotifyConnector extends Connector
{
    use HasRateLimits;
    
    public function __construct()
    {
<strong>        $this->rateLimitingEnabled = false;
</strong>    }
}

// Or...

$connector = new SpotifyConnector;
$connector->useRateLimitPlugin(false);
</code></pre>
