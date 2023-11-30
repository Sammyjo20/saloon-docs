# 🌳 Upgrading from v2

Saloon version three comes with many improvements over version two, but most of the improvements were internal changes, so the upgrade to v3 from v2 shouldn't be too cumbersome. If you haven't already, it's recommended to read through ["What's new in v3"](whats-new-in-v3.md) to get an idea of what has changed before making the upgrade.

### Upgrade Saloon's Packages

First of all, you should bump Saloon to the next major version in your `composer.json` file. You should also update any of the following plugins if you have them installed.&#x20;

* `saloonphp/saloon` to `^3.0`
* `saloonphp/laravel-plugin` to `^3.0`
* `saloonphp/cache-plugin` to `^3.0`&#x20;
* `saloonphp/laravel-http-sender` to `^2.0`
* `saloonphp/rate-limit-plugin` to `^2.0`
* `saloonphp/pagination-plugin` to `^2.0`

After that, make sure to run the following command to update your `composer.lock` file and your `vendor` directory.&#x20;

```bash
composer update "saloonphp/*"
```

{% hint style="info" %}
The above command will just update the `saloonphp` libraries. If you have other libraries using Saloon you should make sure that they are updated as well.&#x20;
{% endhint %}

### <mark style="color:red;">High</mark> Impact Changes

High-impact changes will most likely be things that everyone needs to update.

#### Minimum TLS 1.2 version enforced

From version three, Saloon uses Guzzle `^7.6` which introduced a new option to configure a minimum TLS version. With Saloon v3, Saloon has set this default to **TLS 1.2.** If any of your API integrations require a lower level of TLS security, you can change the Saloon config while your application is loading.

```php
<?php

use Saloon\Config;

Config::$defaultTlsMethod = STREAM_CRYPTO_METHOD_TLSv1_1_CLIENT;
```

You can [visit this page on the PHP documentation](https://www.php.net/manual/en/function.stream-socket-enable-crypto.php) to see the different TLS options available.&#x20;

{% hint style="info" %}
**TLS 1.1** has been deprecated since 2021 so it is unlikely that APIs are still using it, but older systems may have not upgraded yet.
{% endhint %}

#### New Paginators

With version three, paginators are no longer included in the core Saloon library and have been moved to a plugin. Additionally, the paginators have been completely rebuilt. Review if you have used any of the pagination functionality and follow the new [pagination guide](../digging-deeper/pagination/) to rebuild them in the new way.

#### Config and MockConfig classes have moved

Previously, Saloon's `Config` and `MockConfig` classes lived in a `Saloon\Helpers` directory. These classes have now been moved to the root `Saloon` namespace. You should update your `use` statements to the new namespace.

| Find                            | Replace                 |
| ------------------------------- | ----------------------- |
| `use Saloon\Helpers\Config`     | `use Saloon\Config`     |
| `use Saloon\Helpers\MockConfig` | `use Saloon\MockConfig` |

#### Core Interfaces Removed

Another decision that was made was to backtrack on the interfaces that were added in Saloon v2. This may not affect your integration, but a lot of the code examples and recommendations in v2 suggested using the contracts instead of the abstract classes, so you may have a few places in your application where you have imported the interface and not the class. You should find and replace the following main interfaces.

<table><thead><tr><th width="394">Find</th><th>Replace</th></tr></thead><tbody><tr><td><code>use Saloon\Contracts\Connector</code></td><td><code>use Saloon\Http\Connector</code></td></tr><tr><td><code>use Saloon\Contracts\Request</code></td><td><code>use Saloon\Http\Request</code></td></tr><tr><td><code>use Saloon\Contracts\PendingRequest</code></td><td><code>use Saloon\Http\PendingRequest</code></td></tr><tr><td><code>use Saloon\Contracts\Response</code></td><td><code>use Saloon\Http\Response</code></td></tr></tbody></table>

{% hint style="info" %}
There have been many other interfaces removed in v3 however it's unlikely that your application would have used them. They are listed below.
{% endhint %}

#### HasBody Trait Renamed to HasStringBody

During some of the code review to improve readability, the `HasBody` body trait has been renamed to `HasStringBody` to be more explicit. If any of your requests send a **plain string body**, you should find and replace this trait.

<table><thead><tr><th width="349">Find</th><th>Replace</th></tr></thead><tbody><tr><td><code>use Saloon\Traits\Body\HasBody</code></td><td><code>use Saloon\Traits\Body\HasStringBody</code></td></tr><tr><td><code>use HasBody</code></td><td><code>use HasStringBody</code></td></tr></tbody></table>

### <mark style="color:orange;">Medium</mark> Impact Changes

Medium-impact changes are changes that come up for some people but not others.

#### SendAndRetry method argument name changes

The second argument of the `$connector->sendAndRetry()` method has been renamed from `maxAttempts` to `tries`. If you are using named arguments then you will need to replace this name.

<pre class="language-php"><code class="lang-php">$connector->sendAndRetry(
    request: $request,
<strong>    tries: 5,
</strong>);
</code></pre>

#### SendAndRetry handler now expects a Request class and not PendingRequest

If you previously used the `handleRetry` argument when using `$connector->sendAndRetry()` - Saloon will now send an instance of `Request` instead of `PendingRequest` through the closure.

<pre class="language-php"><code class="lang-php">$connector->sendAndRetry(
    request: $request,
    tries: 5,
    interval: 0,
<strong>    handleRetry: static function (Throwable $exception, Request $request) {
</strong>        // $request->headers()->add();
    },
);
</code></pre>

#### Config Method Changes

With version three, some of the methods in the `Saloon\Config` class have been renamed. These changes have been listed below and you should make sure to rename them if you have used them in your application.

| Find                          | Replace                           |
| ----------------------------- | --------------------------------- |
| `Config::middleware()`        | `Config::globalMiddleware()`      |
| `Config::resetMiddleware()`   | `Config::clearGlobalMiddleware()` |
| `Config::resolveSenderWith()` | `Config::setSenderResolver()`     |
| `Config::setDefaultSender()`  | `Config::$defaultSender =`        |

### <mark style="color:blue;">Low</mark> Impact Changes

Low-impact changes are changes that are unlikely to come up unless you have overwritten some of Saloon's functionality or used its interfaces. It's still worth looking through to make sure you haven't missed anything.

#### Body Method Renamed On MockResponse

Previously, Saloon had a method `getBody()` on a `MockResponse` class. This has now been renamed to `body()` to be more consistent with the rest of Saloon. Additionally, the old `getBodyAsString()` method has been removed.

#### Middleware Arguments Changed

Saloon v3 introduced the ability to reorder middleware. With this, the old `prepend` argument on middleware methods has been replaced with a `pipeOrder` argument. Additionally, the second argument for the middleware is now `name` .&#x20;

#### Methods Removed From PendingRequest Class

The following methods have now been removed from the `PendingRequest` class. You can use the suggested replacement if you are using them in your application.

<table><thead><tr><th width="325">Old</th><th>Suggested Replacement</th></tr></thead><tbody><tr><td><code>$pendingRequest->send()</code></td><td><code>$pendingRequest->getConnector()->send()</code></td></tr><tr><td><code>$pendingRequest->sendAsync()</code></td><td><code>$pendingRequest->getConnector()->sendAsync()</code></td></tr><tr><td><code>$pendingRequest->getSender()</code></td><td><code>$pendingRequest->getConnector()->sender()</code></td></tr></tbody></table>

#### Promise Execution Changes

Previously, Saloon's `PendingRequest` and middleware would be invoked before an asynchronous request was sent (using `$connector->sendAsync()`) - now Saloon has fixed this issue and the `PendingRequest` and middleware will only be invoked while the `Promise` is being resolved - at the same time as sending the request. If your application depended on the old way the middleware was invoked you should make sure your application isn't affected by this change.

#### SimulatedResponsePayload Class Renamed

Previously, the `MockResponse` would extend a class called `SimulatedResponsePayload`. This class has now been renamed to `FakeResponse`. Additionally, methods on the `PendingRequest` like `$pendingRequest->setSimulatedResponsePayload()` has been renamed to `$pendingRequest->setFakeResponse()`. If you were extending the old class or using the methods to set the fake response in middleware, you should rename these methods.

#### Sender Contract Updated

The `Sender` interface has been completely rewritten in Saloon v3. Instead of just the `sendRequest` method. You now have to define three methods. If you have made a custom sender in Saloon, you should make sure this is updated.

```php
<?php

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

#### Date Helper Removed

Saloon v2 shipped with a simple `Date` helper class that wrapped around PHP's `DateTime` classes. This has now been removed in Saloon v3.

#### Other Helpers Renamed And Made Final

Saloon v2 shipped with a few helpers to reduce dependencies. The `Str` and `Arr` helper methods have been renamed to `StringHelpers` and `ArrayHelpers` respectively to prevent accidental imports when using Laravel. Saloon's internal helper classes have also been declared `final` as these shouldn't be extended or used by your application.

#### Other Interfaces (Contracts) Removed

As mentioned above, Saloon v3 removes a lot of interfaces that weren't adding value to the library. Alongside the ones mentioned above, these other interfaces have been removed.

<details>

<summary>Show Removed Interfaces</summary>

* src/Contracts/Arrayable.php
* src/Contracts/Authenticatable.php
* src/Contracts/CanThrowRequestExceptions.php
* src/Contracts/Connector.php
* src/Contracts/DebuggingDriver.php
* src/Contracts/Dispatcher.php
* src/Contracts/HasConfig.php
* src/Contracts/HasDebugging.php
* src/Contracts/HasDelay.php
* src/Contracts/HasHeaders.php
* src/Contracts/HasMiddlewarePipeline.php
* src/Contracts/HasMockClient.php
* src/Contracts/HasPagination.php
* src/Contracts/HasQueryParams.php
* src/Contracts/IntegerStore.php
* src/Contracts/MiddlewarePipeline.php
* src/Contracts/MockClient.php
* src/Contracts/Paginator.php
* src/Contracts/PendingRequest.php
* src/Contracts/Pipeline.php
* src/Contracts/Pool.php
* src/Contracts/Request.php
* src/Contracts/Response.php
* src/Contracts/SimulatedResponsePayload.php

</details>

#### The Base BodyRepository Class Is No Longer Stringable

If you have made your own `BodyRepository` class from the base class and relied on the `__toString` method this has now been removed. Additionally, the interface has been updated and now requires a `toStream()` method. Inside here, you must define how to convert your body type into a Stream using the provided `StreamFactory`.

#### Timeout Enum Removed

The old `Timeout` enum has now been removed. If your application used to use this method, you should migrate to use `Config::$defaultConnectionTimeout` and `Config::$defaultRequestTimeout`.

#### ToArray Method Removed From MultipartValue

Previously, the `MultipartValue` object had a `toArray` method. Now that all body repositories are converted into Streams, this method is no longer required.

#### SimulatedSender Removed

The `SimulatedSender` class which would be used if you were mocking or had a cached response has now been removed.

### Did we miss anything?&#x20;

If we forgot to add something to this upgrade please let us know by [opening an issue](https://github.com/saloonphp/saloon/issues).&#x20;
