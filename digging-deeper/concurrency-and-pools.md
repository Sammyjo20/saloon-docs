# 🏎 Concurrency & Pools

{% hint style="warning" %}
This documentation is still a work in progress while Saloon v2 is in beta.
{% endhint %}

Saloon supports request concurrency and requests pools out of the box. This allows you to make multiple API calls to the same service while keeping the CURL connection open. Under the hood, it uses curl's multi-handler to keep the connection open, this results in huge speed benefits when making lots of API calls. Saloon's concurrency is powered by [Guzzle's implementation](https://docs.guzzlephp.org/en/stable/quickstart.html?highlight=pool#concurrent-requests) behind the scenes.

One of Laravel's core members, Nuno Maduro [wrote a great blog post](https://nunomaduro.com/speed\_up\_your\_php\_http\_guzzle\_requests\_with\_concurrency) about request concurrency and its performance with Guzzle directly. The same performance can be shared with Saloon's implementation as Saloon uses Guzzle behind the scenes.

{% hint style="info" %}
Concurrency is only supported with the `GuzzleSender` and `HttpSender` senders for Saloon. The default sender out of the box with Saloon is the GuzzleSender.
{% endhint %}

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

$promise = $pool->send();

$promise->wait();
```

### Available Methods

* setRequests(callable|iterable $requests)
* getRequests()
* withResponseHandler(callable $callable)
* withExceptionHandler(callable $callable)
* send()

### Providing Requests

The pool class accepts many types of requests:

* Array of requests
* PHP generator providing requests
* Closure or Invokable class returning an array of requests
* Closure or Invokable class returning a generator

You may provide these requests into the pool either as the first argument of the `pool` method or using the `setRequests` method. The requests can be instances of `Saloon\Contacts\Request` or `Saloon\Contracts\PendingRequest`.

#### Array of requests&#x20;

The simplest way to provide requests to the pool is an array.&#x20;

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

If you are going to send lots of requests you may wish to provide a generator into the pool. This allows you to keep memory consumption low and potentially send hundreds of requests.&#x20;

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

$forge = new ForgeConnector;
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

By default, Saloon will send up to 5 requests concurrently. You can customise the concurrency that is used by pools with the `setConcurrency` method. It accepts an integer or a callable like a method or an invokable class in case you want to write advanced logic to determine the concurrency.&#x20;

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

### Sending

Once you have provided the pool with requests, you are ready to send the requests!

### Response Handlers

### Named/Keyed Requests

You may wish to key the requests so you know which ones come back first.
