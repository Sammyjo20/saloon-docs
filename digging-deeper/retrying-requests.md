# 🎯 Retrying Requests

Sometimes you may deal with APIs that fail frequently or require you to retry multiple times before a request is successful. Saloon has a useful built-in feature that allows you to send a request and retry multiple times.&#x20;

This method has been heavily inspired by Laravel's [excellent retrying functionality](https://laravel.com/docs/9.x/http-client#retries) for their built-in HTTP client.

### Getting Started

Saloon has a built-in method on the connector called `sendAndRetry` . This method accepts the maximum number of attempts Saloon should make and an optional interval between requests.

```php
<?php

$forge = new ForgeConnector;

$response = $forge->sendAndRetry(new GetServersRequest, 3, 100);
```

If a request fails, it will be attempted again - if it reaches the maximum number of errors, a `RequestException` will be thrown with the last exception that happened. If a connection error happens, it will throw a `FatalRequestException`. If a request is successful at any point, it will return a `Response` instance.

{% hint style="info" %}
Currently the `sendAndRetry` the method only works with synchronous requests because it requires a response to come back to determine if it was successful.&#x20;
{% endhint %}

### Customising when a retry is attempted

By default, Saloon uses the `throw` method on the response. If it throws an exception, it will be retried. Read more about how this method works [here](../the-basics/handling-failures.md). You may choose to use the fourth argument `handleRetry` to provide a closure that should return a boolean if true, Saloon will make the next attempt. For example, you may only want to retry if a `FatalRequestException` happens, which means it likely couldn't connect to the service.

```php
<?php

$forge->sendAndRetry(new GetServersRequest, 3, 100, function ($exception) {
    return $exception instanceof FatalRequestException;
});
```

If a request fails, you may also choose to use this method to change the next request that is sent. For example, it may have failed because of an expired authentication token. The second argument of the `handleRetry` method provides you with the next `PendingRequest` instance.

```php
<?php

$forge->sendAndRetry(new GetServersRequest, 3, 100, function ($exception, $pendingRequest) {
    if (! $exception instanceof RequestException || $exception->getResponse()->status() !== 401) {
        return false;
    }
    
    $pendingRequest->authenticate($this->refreshAccessToken());
    
    return true;
});
```

{% hint style="danger" %}
When modifying the PendingRequest,  Request middleware cannot be added to a because the PendingRequest has already run the middleware pipeline. Additionally, you should take care when debugging because the debugger will log the PendingRequest before it is modified by the sendAndRetry method.
{% endhint %}

### Disabling throwing exceptions

By default, Saloon will throw an exception if all the attempts are made and every attempt was unsuccessful. You may choose to disable this functionality and always return a failed response by using the `throw` argument.

```php
<?php

$response = $forge->sendAndRetry(new GetServersRequest, 3, 100, throw: false);
```

{% hint style="info" %}
Note that if a connection failure happens, Saloon will still throw a `FatalRequestException` as there will be no response to return.
{% endhint %}
