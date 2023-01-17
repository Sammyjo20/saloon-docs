# 🚧 Manual Fake Responses

Saloon makes it easy to fake API integrations in your tests. In your tests, you will need to create an instance of `MockClient` . The `MockClient` accepts an array of `MockResponses` which when used on a request, will respond with a fake response without actually sending a real request to the web. This helps speed up tests massively and can help you test your application for different API response scenarios, like a 404 error or 500 error.

### Registering a MockClient

Request mocking starts with a `MockClient`. This class can be applied directly to a connector instance to be used across all requests, or it can be applied on a per-request basis.

{% tabs %}
{% tab title="All Requests (Connector)" %}
```php
<?php

use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    MockResponse::make(['name' => 'Sam'], 200),
    MockResponse::make(['name' => 'Alex'], 200),
    MockResponse::make(['error' => 'Server Unavailable'], 500),
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

// All requests sent with the $forge instance will use the Mock Client
```
{% endtab %}

{% tab title="Individual Request" %}
```php
<?php

use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    MockResponse::make(['name' => 'Sam'], 200),
    MockResponse::make(['name' => 'Alex'], 200),
    MockResponse::make(['error' => 'Server Unavailable'], 500),
]);

$forge = new ForgeConnector;
$request = new GetAllServersRequest;

// Send a request with a MockClient

$forge->send($request, $mockClient);
```
{% endtab %}
{% endtabs %}

### Using Laravel?

If you are using Laravel, as well as the Saloon Laravel helper library, you don't have to use `withMockClient` on every instance of your connector. You may use the `Saloon` facade `fake` method to define your mock responses. This is a built-in global MockClient that when used will be applied to _all_ Saloon requests sent in your application.

```php
<?php

use Saloon\Laravel\Saloon;

Saloon::fake([
    MockResponse::make(['name' => 'Sam'], 200),
    MockResponse::make(['name' => 'Alex'], 200),
    MockResponse::make(['error' => 'Server Unavailable'], 500),
]);

// All requests sent with the $forge instance will use the global MockClient

$forge = new ForgeConnector;
$forge->send(new GetUserRequest) // Will return with `['name' => 'Sam']` and status `200`
```

### The MockResponse class

The **MockResponse** class is used to create fake responses Saloon understands. It can accept a body, status, and headers. These properties will be populated in the fake response. The response body accepts an array for a JSON body or plain strings to simulate other responses, like XML.

```php
use Saloon\Http\Faking\MockResponse;

MockResponse::make(['name' => 'Sam'], 200, ['Content-Type' => 'application/json']);
```

### Basic Usage (Sequence Mocking)

Basic sequence testing allows you to define a number of fake responses. When your application uses Saloon, it will pull out the next response in the sequence, removing it from the sequence. Each response can only be consumed once.

```php
use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    MockResponse::make(['name' => 'Sam'], 200),
    MockResponse::make(['name' => 'Alex'], 200),
    MockResponse::make(['error' => 'Server Unavailable'], 500),
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

$otherService = new OtherServiceConnector;
$otherService->withMockClient($mockClient);

$forge->send(new GetUserRequest) // Will return with `['name' => 'Sam']` and status `200`
$forge->send(new GetUserRequest) // Will return with `['name' => 'Alex']` and status `200`
$otherService->send(new GetUserRequest) // Will return with `['error' => 'Server Unavailable']` and status `500`
```

### Connector Mocking

You may also explicitly define the connector to target with mock responses. Unlike sequence tests, these are always used and are never consumed.

```php
use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    ForgeConnector::class => MockResponse::make(['name' => 'Sam'], 200),
    OtherServiceConnector::class => MockResponse::make(['name' => 'Alex'], 200),
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

$otherService = new OtherServiceConnector;
$otherService->withMockClient($mockClient);

$forge->send(new GetUserRequest) // Will return with `['name' => 'Sam']` and status `200`
$forge->send(new GetUserRequest) // Will return with `['name' => 'Sam']` and status `200`
$otherService->send(new OtherServiceRequest) // Will return with `['name' => 'Alex']` and status `200`
```

### Request Mocking

You may also explicitly define the request to target with mock responses that are used. Unlike sequence tests, these are always used and are never consumed.

```php
use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    GetForgeServerRequest::class => MockResponse::make(['name' => 'Sam'], 200),
    OtherServiceRequest::class => MockResponse::make(['name' => 'Alex'], 200),
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

$otherService = new OtherServiceConnector;
$otherService->withMockClient($mockClient);

$forge->send(new GetUserRequest) // Will return with `['name' => 'Sam']` and status `200`
$otherService->send(new OtherServiceRequest) // Will return with `['name' => 'Alex']` and status `200`
```

### URL Mocking

You can also define fake responses for particular URL patterns. Whenever a request is made for a particular pattern, Saloon will respond to that request.

```php
use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    'forge.laravel.com/api/*' => MockResponse::make(['name' => 'Sam'], 200),
    'samcarre.dev/*' => MockResponse::make(['name' => 'Alex'], 200),
    'samcarre.dev/exact' => MockResponse::make(['name' => 'Taylor'], 200), // Exact requests
    '*' => MockResponse::make(['name' => 'Wildcard'], 200), // Any other requests
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

$otherService = new OtherServiceConnector;
$otherService->withMockClient($mockClient);

$forge->send(new GetForgeServerRequest) // Will return with `['name' => 'Sam']` and status `200`
$otherService->send(new OtherServiceRequest) // Will return with `['name' => 'Alex']` and status `200`
$forge->send(new ExactRequest) // Will return with `['name' => 'Taylor']` and status `200`
$forge->send(new WildcardServiceRequest) // Will return with `['name' => 'Wildcard']` and status `200`
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

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

$response = $forge->send(new GetForgeServerRequest(123456));

$mockClient->assertSent(GetForgeServerRequest::class);
```

The **AssertSent / AssertNotSent** are the two most powerful expectation methods. They can accept a Saloon request, a URL pattern or even a closure where you define if a request/response is what you expect.

```php
<?php

// ... 

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

$response = $forge->send(new GetForgeServerRequest(123456));

$mockClient->assertSent(GetForgeServerRequest::class);

$mockClient->assertSent('/servers/*');

$mockClient->assertSent(function (SaloonRequest $request, SaloonResponse $response) {
    return $request instanceof GetForgeServerRequest::class 
    && $request->serverId === 123456;
});
```

### Mocking Exceptions

Your test may require you to mock your own exceptions that might get thrown. To mock an exception, chain the `throw` method after you have defined your mock response.

```php
<?php

$mockClient = new MockClient([
    MockResponse::make(['name' => 'Sam'], 200)->throw(new MyException('Something bad!'))
]);
 
// ...
```

### Using closures for mocking

Sometimes, you may need to return a custom mock response based on the request that is currently trying to be sent. With closure/callable mocking, you can do this. Just provide an anonymous function or an invokable class when defining the mock response, and you will get access to the current `PendingRequest` before it is converted into a mock response. This is great if you have stored fixtures based on the request and need to load the fixture data up. This will work with all of the methods above of mocking the request.

```php
<?php

use Saloon\Contracts\Request;

$mockClient = new MockClient([
    function (PendingRequest $request): MockResponse {
        // Write some custom logic here...
    
        return new MockResponse([...]);
    },
]);
```
