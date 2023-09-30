# 🔁 Caching Responses

There are scenarios where you may want to cache a response from an API, like retrieving a static list or retrieving data that you know won't change for a specified amount of time. Caching can be incredibly powerful and can speed up an application by relying less on a third-party integration. Saloon has a [first-party plugin](https://github.com/Sammyjo20/saloon-cache-plugin) that you can install to enable caching support.

### Installation

To install the caching functionality into Saloon, install the plugin using Composer.

```bash
composer require saloonphp/cache-plugin "^2.0"
```

### Configuration

Next, add the `Cacheable` interface and `HasCaching` trait to your request or connector. You must define the two methods.

* resolveCacheDriver
* cacheExpiryInSeconds

```php
<?php

use Saloon\CachePlugin\Contracts\Driver;
use Saloon\CachePlugin\Traits\HasCaching;
use Saloon\CachePlugin\Contracts\Cacheable;

class GetServersRequest extends Request implements Cacheable
{
    use HasCaching;

    // ...
    
    public function resolveCacheDriver(): Driver
    {
        //
    }
    
    public function cacheExpiryInSeconds(): int
    {
        //
    }
}
```

{% hint style="warning" %}
When you add the `HasCaching` trait onto your connector, every request through the connector will be cached.
{% endhint %}

### Cache Drivers

Saloon's caching plugin works with different caching drivers that you can use on your request or connector. The plugin comes with the following cache drivers:

* PsrCacheDriver (Supports PSR-16 Cache Implementations)
* FlysystemDriver (Requires `league/flysystem` version 3)
* LaravelCacheDriver (Supports any of Laravel's cache disks, requires Laravel)

{% tabs %}
{% tab title="PsrCacheDriver" %}
```php
use Saloon\CachePlugin\Drivers\PsrCacheDriver;

public function resolveCacheDriver(): Driver
{
    // This example uses the PhpArrayAdapter from 
    // the symfony/cache library.
    
    return new PsrCacheDriver(new PhpArrayAdapter);
}
```
{% endtab %}

{% tab title="FlysystemDriver" %}
```php
use Saloon\CachePlugin\Drivers\FlysystemDriver;

public function resolveCacheDriver(): Driver
{
    // This example uses the "AwsS3V3Adapter" driver
    // that is provided by Flysystem.

    return new FlysystemDriver(
        new Filesystem(new AwsS3V3Adapter($s3Client, 'bucket-name'))
    );
}
```
{% endtab %}

{% tab title="LaravelCacheDriver" %}
```php
use Illuminate\Support\Facades\Cache;
use Saloon\CachePlugin\Drivers\LaravelCacheDriver;

public function resolveCacheDriver(): Driver
{
    // This example uses Redis cache store that
    // Laravel provides.

    return new LaravelCacheDriver(Cache::store('redis'));
}
```
{% endtab %}
{% endtabs %}

### Expiry

When Saloon caches a request, it will last a specified amount of time in seconds. You should specify this time in the `cacheExpiryInSeconds` method.

```php
public function cacheExpiryInSeconds(): int
{
    return 3600; // One Hour
}
```

### Caching Requests

Once you have configured the cache driver and expiry, Saloon will automatically cache requests and store them in your requested cache-store. The next time you send a request, the cached response will be automatically swapped out and no real request will be sent.

#### Checking if a response is cached

You may use the `isCached()` method to determine if a response is cached.

```php
<?php

$forge = new ForgeConnector;

$response = $forge->send(new GetServersRequest);
$response->isCached(); // false

// Next time it is sent...

$response = $forge->send(new GetServersRequest);
$response->isCached(); // true
```

#### When will Saloon cache a request?

Saloon will only cache a **successful** request when the method is either GET or OPTIONS. You can customise this by extending the `getCacheableMethods` method where you added the `HasCaching` trait.

```php
<?php

use Saloon\Enums\Method;

class GetServersRequest extends Request implements Cacheable
{
    use HasCaching;
    
    protected function getCacheableMethods(): array
    {
        return [Method::GET, Method::OPTIONS, Method::POST];
    }
}
```

You may customise when Saloon considers a request as successful. To read more about this, [click here.](../the-basics/handling-failures.md#customising-when-saloon-thinks-a-request-has-failed)

### Customising the cache key

By default, the cache key is created from the full request URL, the headers that are sent and the query parameters that are used. You may choose to define your own custom cache key. Just extend the protected `cacheKey` method where you added the `HasCaching` trait. You will get access to the `PendingRequest` instance that contains all the request properties.

```php
<?php

use Saloon\Enums\Method;

class GetServersRequest extends Request implements Cacheable
{
    use HasCaching;
    
    protected function cacheKey(PendingRequest $pendingRequest): ?string
    {
        return 'custom-cache-key';
    }
}
```

### Invalidating the current cache

You may want to make a request and purge the existing cache before making the request. You can use the `invalidateCache` method on the request before sending the request and Saloon will delete any existing cache for that request.

```php
<?php

$forge = new ForgeConnector;

$request = new GetServersRequest;
$request->invalidateCache();

$response = $forge->send($request);
```

### Temporarily Disabling Caching

Sometimes you may wish to disable the caching on a per-request basis for debugging or to bypass caching. You can do this by using the `disableCaching` method on the request.

```php
<?php

$forge = new ForgeConnector;

$request = new GetServersRequest;
$request->disableCaching();

$response = $forge->send($request);
```

### Source Code

To view the source code of this plugin, [click here](https://github.com/Sammyjo20/saloon-cache-plugin).
