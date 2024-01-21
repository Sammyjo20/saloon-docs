# ✅ Testing

### Introduction

Writing tests for your API integrations is a super important part of ensuring your application remains stable over time. This section of the documentation will walk you through how to use Saloon's testing helpers to mock API requests and write assertions to prove they are working.

#### What should you test?

The general rule of thumb is that you should assume that the API you are integrating with has tests that ensure their API always returns what they tell you, so you don't have to worry about testing that specific endpoint returns the right data.

You should, however, always test how your application handles the API response that comes back. What happens if the API is down or it returns a 500 error? What happens if your access token expires? You should make sure your application can handle these scenarios.

#### Why use fake responses and fixtures instead of making the API calls every time?

When running your application tests, you are likely going to be running them over and over again which might incur charges or hit API rate limits. Instead, you can either write a fake response or Saloon can record an API call once and re-use that same response in all subsequent tests.&#x20;

Faking API responses are typically much faster as they do not require an internet connection or potentially long wait times.

### Getting Started

#### The MockClient

Saloon's testing starts with the **MockClient** class. This class can be instantiated locally for testing individual requests or used globally to test an API call nested deep in your application. Inside the MockClient, you can define requests and responses that should be returned instead of a real API call.

```php
use Saloon\Http\Faking\MockClient;

$mockClient = new MockClient([
    GetServersRequest::class => MockResponse::make(body: '', status: 200),
]);
```

You use the key of the MockClient to define a request that should be mocked. You can also use a URL pattern. You can use an asterisk (**\***) to act as a wild card.

```php
$mockClient = new MockClient([
    'forge.laravel.com/api/v1/servers' => MockResponse::make(body: '', status: 200),
    'forge.laravel.com/*' => MockResponse::make(body: '', status: 200),
]);
```

You can also use a sequence of responses which will be returned regardless of the API call.

```php
$mockClient = new MockClient([
    MockResponse::make(body: 'First', status: 200),
    MockResponse::make(body: 'Second', status: 200),
    MockResponse::make(body: 'Third', status: 200),
]);
```

You can also use a closure for more advanced mock responses.

```php
$mockClient = new MockClient([
    GetServersRequest::class => function (PendingRequest $pendingRequest) {
        return MockResponse::make(...);
    },
]);
```

#### Manual Fake Responses

There are two ways to define a response, you can do it manually with `MockResponse::make()` which is suitable for simple responses. It will accept three arguments: body, HTTP status code and headers. You may pass a string or an array into the body argument. If you provide an array, it will automatically be converted into JSON.

```php
$mockClient = new MockClient([
    GetUserRequest::class => MockResponse::make(
        body: ['user' => ['name' => 'Sam']], 
        status: 200, 
        headers: ['Content-Type' => 'application/json']
    ),
]);
```

#### Recorded Responses (Fixtures)

The second way to define a response is with `MockResponse::fixture()` this is suitable for testing larger API responses or to save time. With this, Saloon will make a real API call for the first time, and then store the API call inside of a file in your application, so that when you run the test again, the response will be used and not make a real API call.

```php
$mockClient = new MockClient([
    GetUserRequest::class => MockResponse::fixture('user');
]);
```

### Testing your application

{% hint style="info" %}
This section of the documentation uses the new **Global Mock Client** introduced in Saloon v3.5. Make sure that you are using this version of Saloon by running: `composer update "saloonphp/*"`
{% endhint %}

Now that you have an idea of how testing works in Saloon, let's put it into practice. This section of the documentation will teach you how to write a MockClient for your application. This guide assumes you have already set up a test suite for your application. If you are new to testing, we recommend [PEST](https://pestphp.com/).

#### Setup

First of all, you will need to add a method which should run before every test in your application. Without this, the global mock client will leak into other tests and may break your test suite.&#x20;

{% tabs %}
{% tab title="PEST" %}
If you are using PEST, then you should add the following code to your `Pest.php` file.

```php
use Saloon\Http\Faking\MockClient;

uses()
    ->beforeEach(fn () => MockClient::destroyGlobal())
    ->in(__DIR__);
```
{% endtab %}

{% tab title="PHPUnit" %}
If you are using PHPUnit, then you should add the following line inside of your `setUp` method in your test or application's `TestCase.php` file.

```php
use Saloon\Http\Faking\MockClient;

class RequestTest extends TestCase
{
    protected function setUp(): void
    {
        MockClient::destroyGlobal();
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If you are using Laravel and have installed the [Laravel plugin](../../installable-plugins/laravel-integration.md), you do not need to destroy the global mock client before each test as the plugin will do this for you.
{% endhint %}

#### Writing tests

Let's say that your application has a controller which queries the Laravel Forge API and stores the list of servers in the database. We'll use the global mock client to return fake data so we don't send an API call to the external API.

Saloon will detect the global mock client and will use the mock response which you have defined instead of sending a real API call. It's recommended to define your mock at the top of a test.

{% tabs %}
{% tab title="Test" %}
```php
<?php

use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

test('can store servers in the database from laravel forge', function () {
    MockClient::global([
        GetServersRequest::class => MockResponse::make(
            body: [
                'data' => [
                    ['name' => 'WEB-1', 'ip' => '192.168.0.1'],
                    ['name' => 'WEB-2', 'ip' => '192.168.0.2'],
                ],
            ],
            status: 200,
        ),
    ]);

    $this->assertDatabaseCount('servers', 0);
    
    // Call our controller which will invoke the Saloon request

    $this
        ->getJson('/api/servers/sync')
        ->assertOk();
        
    // Assert our database has created servers!
        
    $this->assertDatabaseCount('servers', 2);
    
    $this->assertDatabaseHas('servers', [
        'name' => 'WEB-1',
        'ip' => '192.168.0.1',
    ]);
    
    $this->assertDatabaseHas('servers', [
        'name' => 'WEB-2',
        'ip' => '192.168.0.2',
    ]);
});
```
{% endtab %}

{% tab title="Controller Code" %}
```php
<?php

class ForgeController
{
    // api/servers/sync
    
    public function sync()
    {
        $forge = new ForgeConnector;
        $servers = $forge->send(new GetServersRequest)->array('data');

        foreach($servers as $server) {
            DB::table('servers')->insert([
                'name' => $server['name'],
                'ip' => $server['ip'],
            ]);
        }

        return response('Synced servers', 200);
    }
}
```
{% endtab %}
{% endtabs %}

### Recording Requests (Fixtures)

So far we have covered testing where you have to write your mock response yourself. This works well for small responses but most API responses will have a lot more data. As previously mentioned, Saloon supports recording requests in your tests. This works by allowing the test to make a real API call the first time and then on subsequent API calls, the same response will be used.&#x20;

This works by saving the response as a file in your application which can be committed in your project and used for later.

{% hint style="warning" %}
Make sure that you do not accidentally store sensitive information from the real API call. You can read more about redacting fixtures [here](./#redacting-recorded-responses-fixtures).
{% endhint %}

#### Example with fixtures

Let's take a look at the example above but with fixtures, as you can see nothing else has to change except that we are using `MockResponse::fixture()`. It accepts one argument which is the name of the fixture.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

test('can store servers in the database from laravel forge', function () {
    MockClient::global([
<strong>        GetServersRequest::class => MockResponse::fixture('servers'),
</strong>    ]);

    $this->assertDatabaseCount('servers', 0);
    
    // Call our controller which will invoke the Saloon request

    $this
        ->getJson('/api/servers/sync')
        ->assertOk();
        
    // Assert our database has created servers!
        
    $this->assertDatabaseCount('servers', 2);
    
    $this->assertDatabaseHas('servers', [
        'name' => 'REAL-SERVER-1',
        'ip' => '192.168.0.1',
    ]);
    
    $this->assertDatabaseHas('servers', [
        'name' => 'REAL-SERVER-2',
        'ip' => '192.168.0.2',
    ]);
});
</code></pre>

#### Customising Fixture Location

By default, fixtures will be stored in `tests/Fixtures/Saloon`. You can customise the fixture location by using the `setFixturePath` method on the `MockConfig` class.

```php
MockConfig::setFixturePath('tests/other-directory');
```

#### Namespacing Fixtures

With multiple API integrations, having all the fixtures in one directory can get messy. You can use `/` to create folders for your fixtures.

```php
MockClient::global([
   GetServersRequest::class => MockResponse::fixture('forge/servers'),
]);
```

#### Refreshing Fixtures

If you need to refresh a fixture, all you have to do is delete the JSON file and re-run your test. Saloon will see the file is no longer there and will re-record the request.

### Testing individual requests and SDKs

Occasionally, you might want to write unit tests for individual requests to ensure you are sending the right information to an API. If you are building an SDK, then you may wish to follow this guide for testing your SDK.

#### Using the withMockClient method on your connector

Instead of using the global mock client, you can create a local instance of a MockClient and pass it into your connector or request using the `withMockClient` method.

<pre class="language-php"><code class="lang-php">&#x3C;?php

test('a request sends the correct body', function () {
    $mockClient = new MockClient([
        CreateServerRequest::class => MockResponse::make('Server Created', 200),
    ]);

    $connector = new ForgeConnector;
<strong>    $connector->withMockClient($mockClient);
</strong>    
    $connector->send(new CreateServerRequest($name, $ip));
    
    $mockClient->assertSent(function (Request $request) {
        return $request->body()->all() === ['name' => $name, 'ip' => $ip];
    });
});
</code></pre>

### Assertions

Saloon has a variety of built-in assertion methods which you can also use in your test to assert that Saloon is sending the right information to the third-party API. You can define a variable for your MockClient and call the various assertion methods inside of your test.

```php
<?php

use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

test('can store servers in the database from laravel forge', function () {
    $mockClient = MockClient::global([
        GetServersRequest::class => MockResponse::make(...),
    ]);

    // Your application code...
    
    $mockClient->assertSent(GetServersRequest::class);
    
    $mockClient->assertSentCount(1);
});
```

Saloon has the following assertions:

#### assertSent / assertNotSent

These methods will check that a specific request has been sent.

```php
$mockClient->assertSent(GetServersRequest::class);
```

The method can also accept a closure which should return true for more advanced assertions.

```php
$mockClient->assertSent(function (Request $request, Response $response) {
    return $request instanceof GetServersRequest;
});
```

#### assertSentCount

This method allows you to assert that a specific number of requests has been sent.

```php
$mockClient->assertSentCount(1);
```

The method also accepts a second argument which allows you to check the amount a specific request class was sent.

```php
$mockClient->assertSentCount(2, GetServersRequest::class);
$mockClient->assertSentCount(1, CreateServerRequest::class);
```

#### assertNothingSent

This method will assert that no requests have been sent.

```php
$mockClient->assertNothingSent();
```

### Preventing Stray Requests

You may want to prevent real API calls from being attempted in your tests. You can use `Config::preventStrayRequests()` which will throw an exception if a real API call is attempted.

{% tabs %}
{% tab title="PEST" %}
If you are using PEST, then you should add the following code to your `Pest.php` file.

```php
use Saloon\Config;

Config::preventStrayRequests();
```
{% endtab %}

{% tab title="PHPUnit" %}
If you are using PHPUnit, then you should add the following line inside of your `setUp` method in your test or application's `TestCase.php` file.

```php
use Saloon\Config;

class RequestTest extends TestCase
{
    protected function setUp(): void
    {
        Config::preventStrayRequests();
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Fixtures will still be recorded and not counted as stray requests.
{% endhint %}

### Redacting Recorded Responses / Fixtures

When using fixtures to record real responses from an API - sometimes the API will return some sensitive information that you shouldn't store in your application's repository, like names of real people, financial data or emails. With Saloon, you can create a custom fixture class and provide a few methods to obscure the information when the data is stored. You can even provide closures for the data replacement so you can use tools like Faker to replace data like-for-like.

{% hint style="info" %}
The first time the request is made and the fixture is stored, the original response won't be redacted. Only future requests made with the fixture will use the redacted recording.
{% endhint %}

First, create a new class in your tests directory and extend the base `Fixture` class provided by Saloon. Next, extend the `defineName` method and give the fixture a name. You can still use slashes in this directory to denote folders.

Now you can use a few different methods to redact your fixture data.&#x20;

* defineSensitiveHeaders for headers
* defineSensitiveJsonParameters for JSON responses
* defineSensitiveRegexPatterns for other body types

The methods expect a key => value array where the key is the property that is being redacted and the value is the replacement. You can use a string as the replacement or use a closure for more advanced replacement engines like using faker.

<pre class="language-php"><code class="lang-php">&#x3C;?php

namespace Tests\Fixtures\Forge\SingleServerFixture;

use Saloon\Http\Faking\Fixture;

class SingleServerFixture extends Fixture
{
    protected function defineName(): string
    {
        return 'forge/singleServer';
    }
    
<strong>    protected function defineSensitiveHeaders(): array
</strong>    {
        return [
            'Authorization' => 'REDACTED',
        ];
    }

<strong>    protected function defineSensitiveJsonParameters(): array
</strong>    {
        return [
            'name' => 'REDACTED',
            'password' => function () {
                return faker()->password;
            },
        ];
    }
    
<strong>    protected function defineSensitiveRegexPatterns(): array
</strong>    {
        return [
            '/@[a-z0-9_]{0,100}/' => 'REDACTED-TWITTER-HANDLE',
        ];
    } 
}
</code></pre>

#### Using your custom fixtures

Once you have created your custom fixture class with the redaction configuration, you can simply use it instead of `MockResponse::fixture()` in your tests. All detection methods in the mock client work with this type of fixture too.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$mockClient = new MockClient([
<strong>    GetServerRequest::class => new SingleServerFixture;
</strong>]);
</code></pre>
