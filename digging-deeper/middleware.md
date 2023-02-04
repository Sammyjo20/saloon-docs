# 💂 Middleware

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
use Saloon\Contracts\PendingRequest;

class ForgeConnector extends Connector
{
    // { ... }
    
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
use Saloon\Contracts\PendingRequest;

class GetServersRequest extends Request
{
    // { ... }
    
    public function boot(PendingRequest $pendingRequest): void
    {
        $pendingRequest->headers()->add('X-Current-Time', new DateTime);
    }
}
```
{% endtab %}
{% endtabs %}

### The PendingRequest

The `PendingRequest` class is an intermediary class that Saloon uses to populate everything when you send a request. Every time you send a request, a new PendingRequest instance is created which prevents your connector or request from being mutated every time a request is sent. The PendingRequest class has many of the methods that you are used to seeing on the request/connector for managing headers, query parameters, config, and request body.

The PendingRequest class is used for boot methods, middleware and [plugins](traits.md).

### Request Middleware

You may also at any point tap into the request lifecycle by using request middleware. Request middleware is useful to change something on the PendingRequest instance before the request is sent like authenticating the&#x20;

Simply on your connector or request, you can call the `middleware()` method and use the `onRequest()` method. You should provide a callable, like a closure or invokable class. You get access to the `PendingRequest` instance.

{% hint style="info" %}
Return values are not required, but you may either return an instance of `PendingRequest` or a `MockResponse` class for an early fake response.
{% endhint %}

#### Anonymous Functions

You can use a regular closure/anonymous function to create a middleware on the fly.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Contracts\PendingRequest;

$forge = new ForgeConnector;

$forge->middleware()->onRequest(function (PendingRequest $pendingRequest) {
    $pendingRequest->headers()->add('Authorization', 'Bearer ' . $token);
});
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Contracts\PendingRequest;

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

use Saloon\Contracts\PendingRequest;
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

use Saloon\Contracts\PendingRequest;

$forge = new ForgeConnector;
$forge->middleware()->onRequest(new AuthenticateRequest);
```
{% endtab %}

{% tab title="Usage (Request)" %}
```php
<?php

use Saloon\Contracts\PendingRequest;

$request = new GetServersRequest;
$request->middleware()->onRequest(new AuthenticateRequest);
```
{% endtab %}
{% endtabs %}

### Early Fake Responses

You may also choose to tap into Saloon's MockResponse functionality by creating your own fake responses. Behind the scenes, Saloon's MockResponse extends the SimulatedResponsePayload class which can be returned within a request middleware. When you return a SimulatedResponsePayload or MockResponse, the rest of the request middleware will still be processed but the fake response will be stored on the PendingRequest.&#x20;

If this fake response is present before Saloon sends the request, it will use the `SimulatedSender` instead of the default sender you have provided. This is super handy if you want to build your own middleware that stops Saloon from sending real requests, like for caching.&#x20;

```php
<?php

use Saloon\Contracts\PendingRequest;

$request = new GetServersRequest;

$request->middleware()
    ->onRequest(function (PendingRequest $pendingRequest) {
        return new MockResponse(
            data: ['data' => 'Fake Data!'], 
            statusCode: 200, 
            headers: []
        );
    });
```

{% hint style="warning" %}
Even though you are returning a MockResponse, the next middleware will still receive the PendingRequest instance. Additionally, if another middleware also returns a fake response, the latest will be preferred.
{% endhint %}

### Response Middleware

Once you have sent your request, even if it's a mocked response, Saloon will send the response down the response middleware pipeline. You may add your own response middleware to change the response class or log responses.&#x20;

On your connector or request, you can call the `middleware()` method and use the `onResponse()` method. You should provide a callable, like a closure or invokable class. You get access to the `Response` instance.

{% hint style="info" %}
Return values are not required, but you may return an instance of `Saloon\Contracts\Response` to overwrite the response class in the middleware.
{% endhint %}

#### Anonymous Functions

You can use a regular closure/anonymous function to create a middleware on the fly.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Contracts\Response;

$forge = new ForgeConnector;

$forge->middleware()->onResponse(function (Response $response) {
    Logger::recordResponse($response);
});
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Contracts\Response;

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

use Saloon\Contracts\Response;
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

use Saloon\Contracts\PendingRequest;

$forge = new ForgeConnector;
$forge->middleware()->onResponse(new LogResponse);
```
{% endtab %}

{% tab title="Usage (Request)" %}
```php
<?php

use Saloon\Contracts\PendingRequest;

$request = new GetServersRequest;
$request->middleware()->onResponse(new LogResponse);
```
{% endtab %}
{% endtabs %}

### Using Constructors

While registering middleware on the fly is really useful, it often leads to repeated code. If you would like your connector to always have a specific request or response middleware you should use the `boot` method as described above, or use the constructor of your connector or request.

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

### Using Plugins

Plugins are another useful feature for Saloon that allows you to extend Saloon and tap into the middleware from traits. This is especially useful if you intend to use the trait on multiple requests or connectors.

[Read through the plugins page for more information.](traits.md)

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

use Saloon\Http\Config;

Config::middleware()->onResponse(new LogResponse, 'logResponse');
```

{% hint style="danger" %}
Be very careful with global middleware. Since it uses a static property behind the scenes, the value is kept between tests when running a full test suite. You can use the `Config::resetMiddleware` method to get around this issue.
{% endhint %}

### Middleware Execution Order

The following image illustrates the order that middleware is executed in.

<figure><img src="../.gitbook/assets/Middleware Order.png" alt=""><figcaption></figcaption></figure>

### Prepending Middleware

You may choose to "prepend" middleware which will put a given middleware at the top of the execution chain. For example if I added a middleware in the boot methods, as described above, it would usually run after plugin and request/connector middleware - but if I used the `prepend` method, it will run at the very beginning (assuming nothing else has been prepended after your middleware)

```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\PendingRequest;

class GetServersRequest extends Request
{
    // { ... }
    
    public function boot(PendingRequest $pendingRequest): void
    {
        $request->middleware()->onResponse(new LogResponse, prepend: true);
    }
}
```

### Middleware Caviets

Here are some known caveats that you should know about when using Saloon's middleware.

* **You cannot add request middleware from inside of another request middleware,** but you can add response middleware inside of the onRequest() middleware method.
* **You cannot add response middleware from inside of another response middleware.**
* You may return a simulated response payload or fake response in request middleware but you will always get a PendingRequest back

### Guzzle Handlers / Middleware

With previous versions of Saloon, you could add Guzzle middleware or "handlers" directly to the connector or request. Version two is now sender agnostic, so the `addHandler` method has been removed but you may still add Guzzle middleware if you are using the `GuzzleSender` (the default sender with Saloon)

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
