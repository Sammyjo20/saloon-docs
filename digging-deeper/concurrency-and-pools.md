# 🏎 Concurrency & Pools

Saloon supports request concurrency and requests pools out of the box. This allows you to make multiple API calls to the same service while keeping the CURL connection open. Under the hood, it uses PHP's cURL multi-handler to keep the connection open, this results in huge speed benefits when making lots of API calls. Saloon's concurrency is powered by [Guzzle's implementation](https://docs.guzzlephp.org/en/stable/quickstart.html?highlight=pool#concurrent-requests) behind the scenes.

One of Laravel's core members, Nuno Maduro [wrote a great blog post](https://nunomaduro.com/speed\_up\_your\_php\_http\_guzzle\_requests\_with\_concurrency) about request concurrency and its performance with Guzzle directly. The same performance can be shared with Saloon's implementation as Saloon uses Guzzle behind the scenes.

{% hint style="info" %}
Concurrency is only supported with the `GuzzleSender` and `HttpSender` senders for Saloon. The default sender out of the box with Saloon is the GuzzleSender.
{% endhint %}

We conducted our own benchmark by making 1,000 API calls to our internal testing API, running on a virtual server with 2 vCPUs and 2GB of RAM. The results were impressive, as all 1,000 API calls were completed in just **2.5 seconds**, a significant improvement compared to 60 seconds when using synchronous requests.

```php
<?php

$connector = new TestConnector;

// ⬇️ Takes 60 seconds... 😪

$requests = function () {
    for ($i = 0; $i < 1000; $i++) {
        $connector->send(new GetUserRequest);
    }
};

// ⬇️ Takes 2.5 seconds! 🔥

$requests = function () {
    for ($i = 0; $i < 1000; $i++) {
        yield new GetUserRequest;
    }
};

$connector->pool($requests, concurrency: 10)->send()->wait();
```

### Getting Started

Saloon's pooling has been designed specifically to be easy to use. Simply instantiate your connector class and use the `pool` method to create a pool. This method doesn't require any arguments, but you may provide requests, concurrency and handlers right from this method.

```php
<?php

$forge = new ForgeConnector;

// Pool has various optional parameters if you want to set them here...

$pool = $forge->pool(
    requests: [],
    concurrency: 5,
    responseHandler: function () { ... },
    exceptionHandler: function () { ... },
);

// Initiate the transfer of requests

$promise = $pool->send();

// Force all the requests to be fulfilled

$promise->wait();
```

### Available Methods

* `setRequests(callable|iterable $requests)`
* `getRequests()`
* `withResponseHandler(callable $callable)`
* `withExceptionHandler(callable $callable)`
* `send()`

### Providing Requests

The pool class accepts many types of requests:

* Array of requests
* PHP generator providing requests
* Closure or Invokable class returning an array of requests
* Closure or Invokable class returning a generator

You may provide these requests into the pool either as the first argument of the `pool` method or using the `setRequests` method. The requests can be instances of `Saloon\Http\Request` or `Saloon\Http\PendingRequest`.

#### Array of requests

The simplest way to provide requests to the pool is an array.

```php
<?php

$forge = new ForgeConnector;

$forge->pool([
    new GetServersRequest,
    new GetSitesRequest,
    new GetUserRequest,
]);

// Or 

$pool = $forge->pool();

$pool->setRequests([
    new GetServersRequest,
    new GetSitesRequest,
    new GetUserRequest,
]);
```

#### Using a PHP generator

If you are going to send lots of requests you may wish to provide a generator into the pool. This allows you to keep memory consumption low and potentially send hundreds of requests.

```php
<?php

$generatorCallback = function (): Generator {
    for ($i = 0; $i < 3; $i++) {
        yield $i => new UserRequest;
    }
};

$forge = new ForgeConnector;
$pool = $forge->pool($generatorCallback);

// or

$pool->setRequests($generatorCallback);
```

#### Using a callable or invokable class

You may wish to use a callable method to return an array of requests or a generator, this is useful if you have any additional logic that needs to execute just before the request pool begins.

{% tabs %}
{% tab title="Callable" %}
```php
<?php

$callback = function () {
    return [
        new GetServersRequest,
        new GetSitesRequest,
        new GetUserRequest,
    ];
};

$forge = new Forge;
$pool = $forge->pool($callback);

// or

$pool->setRequests($callback);
```
{% endtab %}

{% tab title="Invokable Class" %}
```php
<?php

class PoolClass {
    public function __invoke() {
        return [
            new GetServersRequest,
            new GetSitesRequest,
            new GetUserRequest,
        ];
    }
}

$forge = new ForgeConnector;
$pool = $forge->pool(new PoolClass);
```
{% endtab %}
{% endtabs %}

### Concurrency

By default, Saloon will send up to 5 requests concurrently. You can customise the concurrency that is used by pools with the `setConcurrency` method. It accepts an integer or a callable like a method or an invokable class in case you want to write advanced logic to determine the concurrency.

```php
<?php

$forge = new ForgeConnector;
$pool = $forge->pool(...);

$pool->setConcurrency(10);

// or

$pool->setConcurrency(function () {
    return 10;
});
```

### Response Handlers

When you send requests with pooling, each request is asynchronous, so you cannot guarantee when they are returned. In order to handle the response, Saloon has provided you with two handlers to handle successful requests and failed requests.

#### Handling Successful Requests

Any responses that are successful will be handled by the response handler. You may add this with the constructor of your pool or add it once it is created. You can only have one handler. You will get a response instance.

```php
<?php

use Saloon\Http\Response;

$pool = $forge->pool(
    requests: [],
    concurrency: 5,
    responseHandler: function (Response $response) {
        // Handle Response
    },
);

// Or

$pool->withResponseHandler(function (Response $response) {
    // Handle Response
});
```

#### Handling Failed Requests

When requests fail, they will always be caught with the error handler, even if you don't throw on requests. When a failed request happens you can handle the exception with the `withExceptionHandler` method.

```php
<?php

use Saloon\Http\Response;
use Saloon\Exceptions\Request\RequestException;
use Saloon\Exceptions\Request\FatalRequestException;

$pool = $forge->pool(
    requests: [],
    concurrency: 5,
    exceptionHandler: function (FatalRequestException|RequestException $exception) {
        // Handle Exception
    },
);

// Or

$pool->withExceptionHandler(function (FatalRequestException|RequestException $exception) {
    // Handle Exception
});
```

### Named/Keyed Requests

Saloon also supports keyed responses to help you easily track exact requests that have been sent. This is especially useful if you are sending requests to different endpoints. You may used keying with arrays or even with a generator.

```php
<?php

$pool = $forge->pool([
    'servers' => new GetServersRequest,
    'sites' => new GetSitesRequest,
    'user' => new GetUserRequest,
]);

// You may access the key in the response and error handlers

$pool->withResponseHandler(function (Response $response, string $key) {
    match($key) {
        'servers' => $this->updateServersList($response),
        'sites' => $this->updateSitesList($response),
        'user' => $this->updateUser($response),
    }
});
```

### Sending

Once you have provided the pool with requests, you are ready to send them. Just use the `send` method on the pool. This will return an instance of `PromiseInterface` . Requests will be handled asynchronously but you can force them to complete with the `wait` method.

```php
<?php

$forge = new ForgeConnector;

$pool = $forge->pool([
    new GetServersRequest,
    new GetSitesRequest,
    new GetUserRequest,
]);

$pool->withResponseHandler(function (Response $response) {
    // Handle Response
});

$pool->withExceptionHandler(function (FatalRequestException|RequestException $exception) {
    // Handle Exception
});

// Initiate the transfer of requests

$promise = $pool->send();

// Force all the requests to be fulfilled

$promise->wait();
```
