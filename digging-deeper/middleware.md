# 💫 Middleware

Saloon has a powerful middleware system that allows you to tap into the request and response lifecycle and make any changes you need before the request is sent or the response is given back to the user. This is useful if you want to build your own advanced Saloon integrations or write more advanced logic like generating a unique reference for every request.

### The Boot Method

Before we get into Saloon's middleware, there is a useful built-in method on every connector and request that you can utilize. This is the `boot` method. It gets executed every time you send a request. You will get access to the underlying `PendingRequest` instance that the sender will provide to the HTTP client. The boot method is a great way to quickly tap into a pending request and change something like add a header, modify the request body or even trigger events.

You may extend the public boot method on either your connector or your request. Remember if you use the connector's boot method, every request with that connector will use that boot method.

You may register middleware inside of the boot method too, which will be used in the rest of the `PendingRequest` lifecycle.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;
use Saloon\Http\PendingRequest;

class ForgeConnector extends Connector
{
    // ...
    
    public function boot(PendingRequest $pendingRequest): void
    {
        $pendingRequest->headers()->add('X-Current-Time', new DateTime);
    }
}
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Http\PendingRequest;

class GetServersRequest extends Request
{
    // ...
    
    public function boot(PendingRequest $pendingRequest): void
    {
        $pendingRequest->headers()->add('X-Current-Time', new DateTime);
    }
}
```
{% endtab %}
{% endtabs %}

### The PendingRequest

The `PendingRequest` class is an intermediary class that Saloon uses to populate everything when you send a request. Every time you send a request, a new `PendingRequest` instance is created which prevents your connector or request from being mutated every time a request is sent. The `PendingRequest` class has many of the methods that you are used to seeing on the request/connector for managing headers, query parameters, config, and request body.

The `PendingRequest` class is used for boot methods, middleware and [plugins](../plugins/traits.md).

### Request Middleware

You may also at any point tap into the request lifecycle by using request middleware. Request middleware is useful for changing something on the `PendingRequest` instance before the request is sent like making another request to get an authentication token for the original request.

On your connector or request, call the `middleware()` method and use the `onRequest()` method. You should provide a callable, like a closure or invokable class. This callable will be given access to the `PendingRequest` instance which can be mutated.

{% hint style="info" %}
Return values are not required, but you may either return an instance of `PendingRequest` or a `FakeResponse` class for an early fake response.
{% endhint %}

#### Anonymous Functions

You can use a regular closure/anonymous function to create a middleware on the fly.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\PendingRequest;

$forge = new ForgeConnector;

$forge->middleware()->onRequest(function (PendingRequest $pendingRequest) {
    $pendingRequest->headers()->add('Authorization', 'Bearer ' . $token);
});
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\PendingRequest;

$request = new GetServersRequest;

$request->middleware()->onRequest(function (PendingRequest $pendingRequest) {
    $pendingRequest->headers()->add('Authorization', 'Bearer ' . $token);
});
```
{% endtab %}
{% endtabs %}

#### Invokable Classes

You may also use invokable classes to keep your middleware classes tidy. If you are using invokable middleware classes, you should implement the `RequestMiddleware` interface.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Http\PendingRequest;
use Saloon\Contracts\RequestMiddleware;

class AuthenticateRequest implements RequestMiddleware
{
    public function __invoke(PendingRequest $pendingRequest): void
    {
        $pendingRequest->headers()->add('Authorization', 'Bearer ' . $token);
    }
}
```
{% endtab %}

{% tab title="Usage (Connector)" %}
```php
<?php

use Saloon\Http\PendingRequest;

$forge = new ForgeConnector;
$forge->middleware()->onRequest(new AuthenticateRequest);
```
{% endtab %}

{% tab title="Usage (Request)" %}
```php
<?php

use Saloon\Http\PendingRequest;

$request = new GetServersRequest;
$request->middleware()->onRequest(new AuthenticateRequest);
```
{% endtab %}
{% endtabs %}

### Early Fake Responses

You may also choose to tap into Saloon's mock functionality by creating your own fake responses. When you return a `FakeResponse`, the rest of the request middleware will still be processed but the fake response will be stored on the `PendingRequest`.

If this fake response is present before Saloon sends the request, it won't send the request to the sender, instead, Saloon will use the `FakeResponse`. This is super handy if you want to build your own middleware that stops Saloon from sending real requests, like for caching.

```php
<?php

use Saloon\Http\PendingRequest;
use Saloon\Http\Faking\FakeResponse;

$request = new GetServersRequest;

$request->middleware()
    ->onRequest(function (PendingRequest $pendingRequest) {
        return new FakeResponse(
            body: ['data' => 'Fake Data!'], 
            status: 200, 
            headers: []
        );
    });
```

{% hint style="warning" %}
Even though you are returning a `FakeResponse` class, the next middleware will still receive the PendingRequest instance. Additionally, if another middleware also returns a fake response, the latest one will be used.
{% endhint %}

### Response Middleware

Once you have sent your request, even if it's a fake response, Saloon will send the response down the response middleware pipeline. You may add your own response middleware to change the response class or do something like log responses.

On your connector or request, you can call the `middleware()` method and use the `onResponse()` method. You should provide a callable, like a closure or invokable class. This callable will be given access to the `Response` instance.

{% hint style="info" %}
Return values are not required, but you may return an instance of `Saloon\Http\Response` to overwrite the response class in the middleware.
{% endhint %}

#### Anonymous Functions

You can use a regular closure/anonymous function to create a middleware on the fly.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Response;

$forge = new ForgeConnector;

$forge->middleware()->onResponse(function (Response $response) {
    Logger::recordResponse($response);
});
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Response;

$request = new GetServersRequest;

$request->middleware()->onResponse(function (Response $response) {
    Logger::recordResponse($response);
});
```
{% endtab %}
{% endtabs %}

#### Invokable classes

Similar to request middleware, you can also create invokable middleware classes for response middleware. You should use the `ResponseMiddleware` contract to ensure your invokable class has the correct signature.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Http\Response;
use Saloon\Contracts\ResponseMiddleware;

class LogResponse implements ResponseMiddleware
{
    public function __invoke(Response $response): void
    {
        Logger::recordResponse($response);
    }
}
```
{% endtab %}

{% tab title="Usage (Connector)" %}
```php
<?php

use Saloon\Http\PendingRequest;

$forge = new ForgeConnector;
$forge->middleware()->onResponse(new LogResponse);
```
{% endtab %}

{% tab title="Usage (Request)" %}
```php
<?php

use Saloon\Http\PendingRequest;

$request = new GetServersRequest;
$request->middleware()->onResponse(new LogResponse);
```
{% endtab %}
{% endtabs %}

### Using Constructors

While registering middleware on the fly is really useful, it often leads to repeated code. If you would like your connector to always have a specific request or response middleware you should use the `boot` method described above, or use the constructor of your connector or request.

```php
<?php

class ForgeConnector extends Connector
{
    // {...}
    
    public function __construct()
    {
        $this->middleware()->onRequest(new AuthenticateRequest);
        $this->middleware()->onResponse(new LogResponse);
    }
}
```

{% hint style="danger" %}
Be cautious using anonymous non-static closures inside the constructor/boot method. This may cause issues like Saloon not being able to close connections properly. [Click here to read more.](../conclusion/known-issues.md#usage-of-anonymous-functions-with-long-running-processes-like-laravel-queues)
{% endhint %}

### Using Plugins

Plugins are another useful feature for Saloon that allows you to extend Saloon and tap into the middleware from traits. This is especially useful if you intend to use the trait on multiple requests or connectors.

[Read through the plugins page for more information.](../plugins/traits.md)

### Middleware Naming

You can choose to name your middleware. Each name must be unique to the given pipeline. For example, you cannot have two middleware with the same name on the request pipeline, but you could have the same name used once for the request pipeline and once for the response pipeline. Use the second argument to specify a name.

```php
<?php

$request = new GetServersRequest;

$request->middleware()->onResponse(new LogResponse, 'logResponse');
```

### Global Middleware

Saloon also supports adding global middleware. You most likely won't need this level of granularity but if you do, you may use the `Config` class. When using global middleware, you should make sure to name your middleware so it isn't accidentally registered twice.

```php
<?php

use Saloon\Config;

Config::globalMiddleware()->onResponse(new LogResponse, 'logResponse');
```

{% hint style="danger" %}
Be cautious with global middleware. Since it uses a static property behind the scenes, the value is kept between tests when running a full test suite. You can use the `Config::clearGlobalMiddleware()` method to get around this issue.
{% endhint %}

### Middleware Execution Order

The following is the order in which middleware is executed.

1. Global middleware (Like the Laravel plugin)
2. Mock client finds a fake response (if present)
3. Plugin middleware
4. User-added middleware
5. Debugging middleware is run (for the final object)

### Choosing When Your Middleware Is Executed

You may choose to change when your middleware is executed. For example, you might want to run something at the very end of all the other middleware that has been run. This is especially useful if you are building a plugin for people to install in Saloon and require the final request object before it is sent. You can use the third argument to specify an order. This expects a `PipeOrder` enum and allows you to choose either `FIRST` or `LAST`.&#x20;

When the middleware is executed, any middleware marked as "first" will run first, and any that have been marked as "last" will be executed last.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Http\PendingRequest;
use Saloon\Enums\PipeOrder;

class GetServersRequest extends Request
{
    // { ... }
    
    public function boot(PendingRequest $pendingRequest): void
    {
<strong>        $request->middleware()->onRequest(new RecordResponse, order: PipeOrder::LAST);
</strong>    }
}
</code></pre>

### Middleware Caveats

Here are some known caveats that you should know about when using Saloon's middleware.

* You cannot add request middleware from inside of another request middleware**,** but you can add response middleware inside of the onRequest() middleware method.
* You cannot add response middleware from inside of another response middleware.
* You may return a fake response in request middleware but you will always get a PendingRequest back

### Guzzle Handlers / Middleware

With previous versions of Saloon, you could add Guzzle middleware or "handlers" directly to the connector or request. From version two, Saloon is now sender-agnostic, so the `addHandler` method has been removed but you may still add Guzzle middleware if you are using the `GuzzleSender` (the default sender with Saloon)

### Adding Guzzle Middleware

You can add middleware to the Guzzle client by using the `sender` method on the connector. You must only add Guzzle middleware directly on your connector with your constructor method. This is because Saloon only instantiates the sender once, so in order to prevent middleware from being registered multiple times, it should be placed in the constructor.

```php
<?php

class ForgeConnector extends Connector
{
    // {...}
    
    public function __construct()
    {
        $this->sender()->addMiddleware(function (callable $handler) {
            return function (RequestInterface $request, array $options) use ($handler) {
                $request->withHeader('X-Custom-Header', 'Hello');
                
                return $handler($request, $options);             
            };
        })
    }
}
```

{% hint style="info" %}
To read more about Guzzle's middleware and handlers [click here](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html).
{% endhint %}

### Accessing The Underlying Guzzle Instance

You may need to modify the Guzzle client or the handler stack. If you need to do this, you can use the `getGuzzleClient` or `getHandlerStack` methods.

```php
<?php

class ForgeConnector extends Connector
{
    // {...}
    
    public function __construct()
    {
        $guzzleClient = $this->sender()->getGuzzleClient();
        
        $handlerStack = $this->sender()->getHandlerStack();
    }
}
```
