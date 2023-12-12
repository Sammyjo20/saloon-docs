# 🪄 What's new in v3

We are excited to announce the release of Saloon version three, which is designed to provide an even better developer experience than before. This version comes with numerous internal changes that enhance maintainability, performance, and code readability. We have taken into account the feedback we received from users of Saloon v2 and made intentional modifications to improve your experience.

To begin it's worth mentioning that Saloon has become a lot more lightweight in version three!

> **333 changed files** with <mark style="color:green;">**4,063 additions**</mark> and <mark style="color:red;">**7,576 deletions**</mark>

### Changes in version three

#### Improved Pagination

One of the biggest changes is how pagination in Saloon works. After seeing a few people use it - we felt that the developer experience could be better, so we revisited the whole feature. With Saloon v3, it no longer comes bundled into Saloon which reduces code but the pagination plugin provides a beautiful, expressive way to build and customise paginators.

One of the motivations for rebuilding the pagination was customisation. Previously, you would have to extend the paginator class you are already using and guess which methods needed to be changed. The new pagination has been massively simplified and each of the paginators use the same methods to apply their pagination. This makes them much easier to build and maintain. Here is how the pagination definition looks in version three:

```php
<?php

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
    public function paginate(Request $request): PagedPaginator
    {
        // You define your own pagination class that extends one of Saloon's
        // base pagination classes. This makes overwriting super easy.
    
        return new class(connector: $this, request: $request) extends PagedPaginator
        {
            // Determine if we are on the last page...
        
            protected function isLastPage(Response $response): bool
            {
                return is_null($response->json('next_page_url'));
            }
            
            // Get page items...
            
            protected function getPageItems(Response $response, Request $request): array
            {
                return $response->json('items');
            }
        };
    }
}
```

#### PSR Support

Another important change to Saloon v3 was to lean more into the building and use of PSR-7 requests. Previously, Saloon would use Guzzle's configuration options to build up query parameters, request body and the URI. Now, Saloon builds all PSR requests in-house and passes that request to the sender. This provides a few key benefits over letting Guzzle handle it:

* **Better developer experience** - with less hidden "magic" going on behind the scenes, you'll be able to debug and dump the PSR request Saloon creates to see how your request will be sent and what you're sending.
* **Decoupled further from Guzzle** - Saloon now only uses Guzzle to send your requests which means Saloon is much less dependent on Guzzle.
* **Lower-level customization** - You will be able to modify the PSR-7 request before it is sent, which gives you much better control than before.
* **New PSR methods -** You will now be able to create a PSR-7 request from a `PendingRequest` class.

{% hint style="info" %}
This is an internal change only, and you won't need to change how your implementation works. You'll still use connectors and requests.
{% endhint %}

#### Global Retry System

Another really exciting feature in version three is the ability to configure retrying functionality at a connector level for all of your requests. Previously, the only way to configure the retry functionality was to do it in-line like this:

```php
$response = $connector->sendAndRetry($request, 5);
```

This worked well, but it doesn't really follow one of Saloon's core values - to have beautiful, reusable code. Now you can define retrying at a class level. You just have to add the `$tries` property to either your connector or request to get it to work. After this, if you use `$connector->send()` and the request fails, it will be retried automatically. This is especially useful for SDK development or building against unstable APIs that need retrying at a connector level.

```php
class ForgeConnector extends Connector
{
    /**
     * The number of times a request should be retried if a failure response is returned.
     *
     * Set to null to disable the retry functionality.
     */
    public ?int $tries = 5;
}
```

#### New Response Body Methods

Saloon version three has a few new methods for the `Response` class to make it even easier to use in your application.

* **saveBodyToFile($pathOrResource)** - This method allows you to save the raw body straight to a file or into an open file resource while keeping memory usage low.
* **getRawStream()** - This method is similar to the existing `stream()` method but will return a file `resource` instead of a `StreamInterface`. This method is really useful when passing the stream into other libraries like League's Flysystem library.
* **getConnector()** - This method is a nice shortcut to get the connector from the response.
* **getPsrRequest()** - This method gives you the PSR request that was built when sending the request - which is the lowest level request class before going through the HTTP client.

#### Removed Interfaces For Maintainability

With Saloon v2 - almost every class had an interface that acted as a blueprint for how the class should be structured. This was a decision made in version two to help make Saloon as flexible as possible. With version three, the decision was made to go back on this flexibility. While the added flexibility of giving the developer the freedom to make their own implementations - it added unnecessary complication and blocked Saloon from nice features and methods that could have been added before version three.

We believe that most Saloon users would just use Saloon's implementation and adequate customization can be achieved with inheritance.&#x20;

Many interfaces (contracts) were removed with version three but some of the major ones include:

* **Saloon\Contracts\Connector**
* **Saloon\Contracts\Request**
* **Saloon\Contracts\PendingRequest**
* **Saloon\Contracts\Response**

The abstract classes still exist, but without the interfaces - the maintainability of Saloon v3 has improved massively. This means that the current version can be maintained for longer and new features can be released faster and without major version changes.

#### Minimum TLS 1.2 Security

Saloon version three requires Guzzle `^7.6` which introduced support for a minimum TLS version. Since TLS 1.1 has been deprecated for almost three years, Saloon has made the minimum version **TLS 1.2**. This is to promote more secure API integrations and follow best practices however it can be changed using configuration if required.

#### Better Middleware Order

The previous version introduced support for middleware in Saloon and allowed users to add their own middleware. This also introduced some challenges with how some of Saloon's other features worked like mocking and authenticators. Often when the user adds their own middleware, they may not see headers from authenticators or mock clients being applied to the `PendingRequest` class. Saloon v3's middleware order has been improved for the best feature compatibility.

If you are curious, this is how the new middleware order looks:

1. Plugins are "booted"
2. Request and connector properties (Headers, Query Parameters, Config, Middleware) are merged
3. Request and connector body is merged
4. Request and connector delay is merged
5. Authenticators are invoked and their logic applied to the `PendingRequest`
6. Request and connector is "booted"
7. Headers are validated
8. Delay is invoked (if applied)
9. Middleware is run in the following order:
   1. Global middleware
   2. Mock client finds a fake response (if present)
   3. Plugin middleware
   4. **User-added middleware**
   5. Debugging middleware is run (for the final object)

#### Simplified Debugging

Saloon v2 shipped with some useful debugging functionality. It came with numerous drivers for different outputs and worked nicely. With version three, it was decided that debugging should be simpler. From version three, if you need to debug your request or response, you can now use the `debug` method and pass in a closure to see either the `PendingRequest` , PSR request or `Response` class while it is being sent.&#x20;

Additionally, changes were made to the middleware pipeline to allow the debugging middleware to always run at the end of the pipeline, ensuring you'll always see the final result.

```php
$connector->debugRequest(static function (PendingRequest $pendingRequest, Requestinterface $psrRequest) {
    dd($pendingRequest->headers());
});

$connector->debugResponse(static function (Response $response) {
    dd($response->body());
});
```

#### Better Asynchronous Promise Handling

With Saloon v3, promises are handled better. Previously, the `PendingRequest` class would be built up, and middleware run and then the `Promise` was created. With version three, a promise is created straight away and the `PendingRequest` and middleware is only invoked when the promise is being sent. This is good for plugins that need to run when a request is being sent like the rate limit plugin.

#### SDK Helpers

Last, but not least - Saloon v3 has shipped with a `BaseResource` class which is a class that was recommended in the [Building SDKs](../the-basics/building-sdks.md#request-resources) chapter of the documentation. Since this class is only small and provides value to the SDK developer, this class has now been bundled.
