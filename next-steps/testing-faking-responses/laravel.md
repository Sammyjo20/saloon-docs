# Laravel

When testing in Laravel, use the Saloon facade and call `Saloon::fake()` method before testing your application logic that makes a Saloon request. Saloon will automatically detect when an API request is about to be made and will respond with the fake response - stopping the real API request from being sent. This powerful feature allows you to test **any** feature in your application that uses Saloon requests.

### The MockResponse class

The **MockResponse** class is used to create fake responses Saloon understands. It can accept data, a status, headers and config.

```php
use Sammyjo20\Saloon\Http\MockResponse;

MockResponse::make(['name' => 'Sam'], 200, $headers, $config);
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
    MockResponse::make(['name' => 'Sam'], 200),
    MockResponse::make(['name' => 'Alex'], 200),
    MockResponse::make(['error' => 'Server Unavailable'], 500),
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
    ForgeConnector::class => MockResponse::make(['name' => 'Sam'], 200),
    OtherServiceConnector::class => MockResponse::make(['name' => 'Alex'], 200),
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
    GetForgeServerRequest::class => MockResponse::make(['name' => 'Sam'], 200),
    OtherServiceRequest::class => MockResponse::make(['name' => 'Alex'], 200),
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
    'forge.laravel.com/api/*' => MockResponse::make(['name' => 'Sam'], 200),
    'samcarre.dev/*' => MockResponse::make(['name' => 'Alex'], 200),
    'samcarre.dev/exact' => MockResponse::make(['name' => 'Taylor'], 200), // Exact requests
    '*' => MockResponse::make(['name' => 'Wildcard'], 200), // Any other requests
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

### Mocking Exceptions

Your test may require you to mock exceptions like Guzzle's RequestException or even your own exceptions that might get thrown. To mock an exception, chain the `throw` method after you have defined your mock response.

```php
<?php

Saloon::fake([
    MockResponse::make(['name' => 'Sam'], 200)->throw(new MyException('Something bad!'))
]);
 
// ...
```

If you would like to test one of the Guzzle exceptions like RequestException, it will expect you to pass in a PSR-7 request as one of the arguments. For these exceptions, provide a closure and Saloon will fulfil the closure with the PSR-7 request.

```php
<?php

use GuzzleHttp\Exception\ConnectException;

Saloon::fake([
    MockResponse::make()->throw(fn ($guzzleRequest) => new ConnectException('Unable to connect!', $guzzleRequest))
]);
```

### Using closures for mocking

Sometimes, you may need to return a custom mock response based on the request that is currently trying to be sent. With closure/callable mocking, you can do this! Just provide an anonymous function or an invokable class when defining the mock response, and you will get access to the current request before it is converted into a mock response. This is great if you have stored fixtures based on the request and need to load the fixture data up. Yes, this will work with all of the methods above of mocking the request.

```php
<?php

Saloon::fake([
    function (SaloonRequest $request): MockResponse {
        // Write some custom logic here...
    
        return new MockResponse([...]);
    },
]);
```
