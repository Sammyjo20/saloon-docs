# Non-Laravel / PHP

Saloon provides a way to test your PHP applications and SDKs really easily. You will need to create an instance of `MockClient` and then pass the `MockClient` as an argument to the `send` method on your request. There are multiple ways you can write tests.

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
use Sammyjo20\Saloon\Clients\MockClient;
use Sammyjo20\Saloon\Http\MockResponse;

$mockClient = new MockClient([
    new MockResponse(['name' => 'Sam'], 200),
    new MockResponse(['name' => 'Alex'], 200),
    new MockResponse(['error' => 'Server Unavailable'], 500),
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
(new GetForgeServerRequest)->send($mockClient) // Will return with `['error' => 'Server Unavailable']` and status `500`
```

### Connector Mocking

You may also explicitly define mock responses for a particular connector that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\Saloon\Clients\MockClient;
use Sammyjo20\Saloon\Http\MockResponse;

$mockClient = new MockClient([
    ForgeConnector::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceConnector::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
```

### Request Mocking

You may also explicitly define mock responses for a particular request that is used. Unlike sequence tests, these are kept even after the response has been sent.

```php
use Sammyjo20\Saloon\Clients\MockClient;
use Sammyjo20\Saloon\Http\MockResponse;

$mockClient = new MockClient([
    GetForgeServerRequest::class => new MockResponse(['name' => 'Sam'], 200),
    OtherServiceRequest::class => new MockResponse(['name' => 'Alex'], 200),
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
```

### URL Mocking

You can also define fake responses for particular URL patterns. Whenever a request is made for a particular pattern, Saloon will respond to that request.

```php
use Sammyjo20\Saloon\Clients\MockClient;
use Sammyjo20\Saloon\Http\MockResponse;

$mockClient = new MockClient([
    'forge.laravel.com/api/*' => new MockResponse(['name' => 'Sam'], 200),
    'samcarre.dev/*' => new MockResponse(['name' => 'Alex'], 200),
    'samcarre.dev/exact' => new MockResponse(['name' => 'Taylor'], 200), // Exact requests
    '*' => new MockResponse(['name' => 'Wildcard'], 200), // Any other requests
]);

(new GetForgeServerRequest)->send($mockClient) // Will return with `['name' => 'Sam']` and status `200`
(new OtherServiceRequest)->send($mockClient) // Will return with `['name' => 'Alex']` and status `200`
(new ExactRequest)->send($mockClient) // Will return with `['name' => 'Taylor']` and status `200`
(new WildcardServiceRequest)->send($mockClient) // Will return with `['name' => 'Wildcard']` and status `200`
```

### Adding Expectations

When using faking responses, it's important to be able to check that a specific make request was sent and with the correct data, headers, and config. Saloon provides you with various ways to add expectations to your tests.&#x20;

#### Available Expectations&#x20;

* AssertSent
* AssertNotSent
* AssertSentJson
* AssertNothingSent
* AssertSentCount

To use one of the expectations, you can simply call the method on your **Mock Client**.

```php
<?php

// ...

$response = (new GetForgeServerRequest(123456))->send($mockClient);

$mockClient->assertSent(GetForgeServerRequest::class);
```

The **AssertSent / AssertNotSent** are the two most powerful expectation methods. They can accept a Saloon request, a URL pattern or even a closure where you define if a request/response is what you expect.

```php
<?php

// ... 

$response = (new GetForgeServerRequest(123456))->send($mockClient);

$mockClient->assertSent(GetForgeServerRequest::class);

$mockClient->assertSent('/servers/*');

$mockClient->assertSent(function (SaloonRequest $request, SaloonResponse $response) {
    return $request instanceof GetForgeServerRequest::class 
    && $request->serverId === 123456;
});
```
