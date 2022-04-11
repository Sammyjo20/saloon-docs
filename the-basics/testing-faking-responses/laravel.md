# Laravel

When testing in Laravel, use the Saloon facade and call `Saloon::fake()` method before testing your application logic that makes a Saloon request. Saloon will automatically detect when an API request is about to be made and will respond with the fake response - stopping the real API request from being sent. This powerful feature allows you to test **any** feature in your application that uses Saloon requests.

### The MockResponse class

The **MockResponse** class is used to create fake responses Saloon understands. It can accept data, a status, headers and config.

```php
use Sammyjo20\Saloon\Http\MockResponse;

new MockResponse(['name' => 'Sam'], 200, $headers, $config),
```

You can also create a **MockResponse** class from an existing request.

```php
use Sammyjo20\Saloon\Http\MockResponse;

MockResponse::fromRequest(new GetForgeServerRequest(12345), 200);
```

### Basic Usage (Sequence Mocking)

Basic sequence testing allows you to define a number of fake responses. When your application uses Saloon, it will pull out the next response in the sequence, removing it too.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;
use Sammyjo20\Saloon\Http\MockResponse;

Saloon::fake([
    new MockResponse(['name' => 'Sam'], 200),
    new MockResponse(['name' => 'Alex'], 200),
    new MockResponse(['error' => 'Server Unavailable'], 500),
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
(new GetForgeServerRequest)->send() // Will return with `['error' => 'Server Unavailable']` and status `500`
```

### Connector Mocking

You may also explicitly define mock responses for a particular connector that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;
use Sammyjo20\Saloon\Http\MockResponse;

Saloon::fake([
    ForgeConnector::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceConnector::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
```

### Request Mocking

You may also explicitly define mock responses for a particular request that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;
use Sammyjo20\Saloon\Http\MockResponse;

Saloon::fake([
    GetForgeServerRequest::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceRequest::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
```

### URL Mocking

You can also define fake responses for particular URL patterns. Whenever a request is made for a particular pattern, Saloon will respond to that request.

```php
use Sammyjo20\SaloonLaravel\Facades\Saloon;
use Sammyjo20\Saloon\Http\MockResponse;

Saloon::fake([
    'forge.laravel.com/api/*' => new MockResponse(['name' => 'Sam'], 200),
    'samcarre.dev/*' => new MockResponse(['name' => 'Alex'], 200),
    'samcarre.dev/exact' => new MockResponse(['name' => 'Taylor'], 200), // Exact requests
    '*' => new MockResponse(['name' => 'Wildcard'], 200), // Any other requests
]);

(new GetForgeServerRequest)->send() // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send() // Will return with `['name' => 'Alex']` and status `200`
(new ExactRequest)->send() // Will return with `['name' => 'Taylor']` and status `200`
(new WildcardServiceRequest)->send() // Will return with `['name' => 'Wildcard']` and status `200`
```

### Adding Expectations

When using faking responses, it's important to be able to check that a specific make request was sent and with the correct data, headers, and config. Saloon provides you with various ways to add expectations to your tests.&#x20;

#### Available Expectations&#x20;

* AssertSent
* AssertNotSent
* AssertSentJson
* AssertNothingSent
* AssertSentCount

To use one of the expectations, you can simply call the static method on the **Saloon Facade.**

```php
<?php

// ...

$response = (new GetForgeServerRequest(123456))->send();

Saloon::assertSent(GetForgeServerRequest::class);
```

The **AssertSent / AssertNotSent** are the two most powerful expectation methods. They can accept a Saloon request, a URL pattern or even a closure where you define if a request/response is what you expect.

```php
<?php

// ... 

$response = (new GetForgeServerRequest(123456))->send();

Saloon::assertSent(GetForgeServerRequest::class);

Saloon::assertSent('/servers/*');

Saloon::assertSent(function (SaloonRequest $request, SaloonResponse $response) {
    return $request instanceof GetForgeServerRequest::class 
    && $request->serverId === 123456;
});
```
