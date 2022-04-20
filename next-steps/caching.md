# Caching

Sometimes you may wish to cache the responses that come back from the API integrations you are integrating. Saloon has a first-party package plugin that you can install to enable this functionality.

{% hint style="warning" %}
Currently the caching plugin only supports explicit caching, where you define when a request should be cached. There will be an update in the future that introduces caching based on the Cache-Control headers.
{% endhint %}

### Installation

Firstly, install the Saloon Cache Plugin from composer into your project.

```bash
composer require sammyjo20/saloon-cache-plugin
```

### Configuration

After you have installed the plugin, add the **AlwaysCacheResponses** trait to your request or connector. After you have added the trait, you will need to define the cache driver and the cache TTL.

```php
<?php

use Sammyjo20\SaloonCachePlugin\Traits\AlwaysCacheResponses;

class GetForgeServerRequest extends SaloonRequest
{
    use AlwaysCacheResponses;

    // ...
    
    public function cacheDriver(): DriverInterface
    {
        //
    }
    
    public function cacheTTLInSeconds(): int
    {
        return 7200;
    }
}
```

{% hint style="danger" %}
If you add the caching plugin to your connector, it will cache every single request that the connector uses.
{% endhint %}

### Cache Drivers

There are three available cache drivers that you can use. The cache driver defines how the cache file will be stored and retrieved by the cache plugin.

#### FlysystemDriver

Allows you to define a Flysystem storage driver that will be used to store the cache files. This allows you to store the cache files in many places like Amazon S3. [Learn more about Flysystem](https://flysystem.thephpleague.com/docs/)

```php
<?php

use League\Flysystem\Filesystem;
use Sammyjo20\SaloonCachePlugin\Drivers\FlysystemDriver;

public function cacheDriver(): DriverInterface
{
    return new FlysystemDriver(new Filesystem(...));
}
```

#### LaravelCacheDriver

Allows you to define a Laravel Cache store like database/redis. This should only be used if you are using Saloon in Laravel.

```php
<?php

use Illuminate\Support\Facades\Cache;
use Sammyjo20\SaloonCachePlugin\Drivers\LaravelCacheDriver;

public function cacheDriver(): DriverInterface
{
    return new LaravelCacheDriver(Cache::store('database'));
}
```

#### SimpleCacheDriver (PSR-16 Compatible)

Allows you to use any PSR-16 compatible cache.

```php
<?php

use Sammyjo20\SaloonCachePlugin\Drivers\SimpleCacheDriver;

public function cacheDriver(): DriverInterface
{
    return new SimpleCacheDriver(new ArrayCache(...));
}
```

### Response

Saloon will respond with a Saloon response (or your custom response if defined) when retrieving a cached response. To check if a response has been cached, you can use the `isCached()` method.

```php
<?php

$request = new GetForgeServerRequest(serverId: '123456');
$response = $request->send();

$response->isCached(); // False on the first request.

$requestTwo = new GetForgeServerRequest(serverId: '123456');
$responseTwo = $requestTwo->send();

$responseTwo->isCached(); // True on the second request
```

### Custom Cache Keys

By default, the cache key will be built up from the full URL of the request, the class name and the headers of the request. The plugin will create a SHA-256 hash based on these three items. If you would like to have your own custom cache key, like using the UUID of a user, then you can extend the `cacheKey` method.

```php
protected function cacheKey(SaloonRequest $request, array $headers): string
{
    return 'my-custom-key';
}
```

### Disable Caching

Sometimes you may wish to disable the caching on a per-request basis for debugging or to bypass caching. You can do this by using the `disableCaching` method on the request.

```php
<?php

$request = new GetForgeServerRequest(serverId: '123456');
$request->disableCaching();

// Send request, will always skip caching.
```

### Invalidating Cache

You may want to make a request and purge the existing cache before making the request. You can use the `invalidateCache` method on the request before sending the request and Saloon will delete any existing cache for that request.

```php
<?php

$request = new GetForgeServerRequest(serverId: '123456');
$request->invalidateCache();

// Send request, will delete any existing cache.
```

### Source Code

[Interested in the source code of this plugin? Click here.](https://github.com/Sammyjo20/saloon-cache-plugin)
