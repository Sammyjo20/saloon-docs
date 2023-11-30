# 📸 Recording Responses

When writing tests for an API integration, it is best to simulate a real request as much as possible. With Saloon's MockResponse class, you can build up example responses manually. This is useful, but it can be time-consuming, especially if an API returns a huge amount of data, it would take a long time to manually write MockResponses and keep it maintained.

Saloon has a feature called fixture recording, this feature will allow you to make a real request to the API you are integrating with and then it will store that response in a file for later. This is a common practice for people writing integrations for APIs, but Saloon makes it effortless.

### Registering a MockClient

Request recording starts with a `MockClient`. This class can be applied directly to a connector instance to be used across all requests, or it can be applied on a per-request basis.

{% tabs %}
{% tab title="All Requests (Connector)" %}
```php
<?php

use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    MockResponse::fixture('servers.index'),
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);
```
{% endtab %}

{% tab title="Individual Request" %}
```php
<?php

use Saloon\Http\Faking\MockClient;
use Saloon\Http\Faking\MockResponse;

$mockClient = new MockClient([
    MockResponse::fixture('servers.index'),
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

use Saloon\Laravel\Facades\Saloon;

Saloon::fake([
    MockResponse::fixture('servers.index'),
]);
```

### Setup

Getting started with fixture recording is easy. When defining your mock responses, instead of defining a MockResponse with headers, data and config - use the fixture static property. This property will accept a single argument, the fixture name.

{% tabs %}
{% tab title="Non-Laravel" %}
```php
<?php

$mockClient = new MockClient([
    GetForgeServerRequest::class => MockResponse::fixture('singleServer')
]);
```
{% endtab %}

{% tab title="Laravel" %}
```php
<?php

Saloon::fake([
    GetForgeServerRequest::class => MockResponse::fixture('singleServer')
]);
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The above example will configure a fixture to be used every time the **UserRequest** is called during mocking. You may also use a sequence of fixtures, connector mocking, or use a fixture on a specific URL path. Read the mocking pages for more information.
{% endhint %}

### How does it work?

Once you have defined a fixture to be used for a particular request pattern, you can make a request just like you normally would. Saloon will check if the fixture already exists, and if it doesn't - it will make the real API request and store the response for next time.

{% tabs %}
{% tab title="Non-Laravel" %}
```php
<?php

$mockClient = new MockClient([
    GetForgeServerRequest::class => MockResponse::fixture('singleServer')
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

// The initial request will check if a fixture called "singleServer" 
// exists. Because it doesn't exist yet, the real request will be
// sent and the response will be recorded.

$request = new GetForgeServerRequest(12345);
$response = $forge->send($request);

// However, the next time the request is made, the fixture will 
// exist, and Saloon will not make the request again.

$request = new GetForgeServerRequest(12345);
$response = $forge->send($request);
```
{% endtab %}

{% tab title="Laravel" %}
```php
<?php

Saloon::fake([
    GetForgeServerRequest::class => MockResponse::fixture('singleServer')
]);

$forge = new ForgeConnector;

// The initial request will check if a fixture called "singleServer" 
// exists. Because it doesn't exist yet, the real request will be
// sent and the response will be recorded.

$request = new GetForgeServerRequest(12345);
$response = $forge->send($request);

// However, the next time the request is made, the fixture will 
// exist, and Saloon will not make the request again.

$request = new GetForgeServerRequest(12345);
$response = $forge->send($request);
```
{% endtab %}
{% endtabs %}

### Namespacing

Depending on the size of your application and the number of API integrations you have, you may want to namespace the fixtures into their own folders, for example, I may have a "forge" namespace and a "digitalOcean" namespace.

{% tabs %}
{% tab title="Non-Laravel" %}
```php
<?php

$mockClient = new MockClient([
    GetForgeServerRequest::class => MockResponse::fixture('forge/singleServer'),
    GetDigitalOceanServerRequest::class => MockResponse::fixture('digitalOcean/singleServer'),
]);
```
{% endtab %}

{% tab title="Laravel" %}
```php
<?php

Saloon::fake([
    GetForgeServerRequest::class => MockResponse::fixture('forge/singleServer'),
    GetDigitalOceanServerRequest::class => MockResponse::fixture('digitalOcean/singleServer'),
]);
```
{% endtab %}
{% endtabs %}

### Configuration

#### Fixture Path

Ordinarily, Saloon will store all fixtures in a **tests/Fixtures/Saloon** directory. If you would like to customise this, you may use the MockConfig class in your tests or in your setUp methods.

```php
use Saloon\MockConfig;

MockConfig::setFixturePath('tests/other-directory')
```

#### Preventing Unwanted Requests

Once you have written all of your tests, you might want to prevent accidental API requests in the future for fixtures that don't exist. If you would like Saloon to throw exceptions if a fixture does not exist, you may do this with the MockConfig class.

```php
use Saloon\MockConfig;

MockConfig::throwOnMissingFixtures()
```

### Redacting Fixture Information

When using fixtures to record real responses from an API - sometimes the API will return some sensitive information that you shouldn't store in your application's repository, like names of real people, financial data or emails. With Saloon, you can create a custom fixture class and provide a few methods to obscure the information when the data is stored. You can even provide closures for the data replacement so you can use tools like Faker to replace data like-for-like.

{% hint style="info" %}
The first time the request is made and the fixture is stored, the original response won't be redacted. Only future requests made with the fixture will use the redacted recording.
{% endhint %}

Firstly, create a new class in your tests directory and give the class a name. We'll call this class `SingleServerFixture`. Then make sure to extend the base `Fixture` class provided by Saloon.&#x20;

<pre class="language-php"><code class="lang-php">&#x3C;?php

namespace Tests\Fixtures\Forge\SingleServerFixture;

<strong>use Saloon\Http\Faking\Fixture;
</strong>
<strong>class ForgeSingleServerFixture extends Fixture
</strong>{
    //
}
</code></pre>

Next, we need to give the fixture a name. Just extend the `defineName` method and give the fixture a name. You can still use slashes in this directory to denote folders.

```php
<?php

namespace Tests\Fixtures\Forge\SingleServerFixture;

use Saloon\Http\Faking\Fixture;

class SingleServerFixture extends Fixture
{
    protected function defineName(): string
    {
        return 'forge/singleServer';
    }
}
```

Now you can use a few different methods to redact your fixture data. You can use the `defineSensitiveHeaders` method to swap any headers out with sensitive data, the `defineSensitiveJsonParameters` for JSON responses or `defineSensitiveRegexPatterns` to define regex patterns for Saloon to find.&#x20;

```php
<?php

namespace Tests\Fixtures\Forge\SingleServerFixture;

use Saloon\Http\Faking\Fixture;

class SingleServerFixture extends Fixture
{
    protected function defineName(): string
    {
        return 'forge/singleServer';
    }
    
    protected function defineSensitiveHeaders(): array
    {
        return [];
    }

    protected function defineSensitiveJsonParameters(): array
    {
        return [];
    }
    
    protected function defineSensitiveRegexPatterns(): array
    {
        return [];
    } 
}
```

#### Replacing Sensitive Headers

If a particular header contains sensitive information, you may use this method to define the headers that are sensitive and what to replace them with. You may use a value or a closure for a custom replacement based on the value

{% tabs %}
{% tab title="Simple - Using Arrays" %}
```php
protected function defineSensitiveHeaders(): array
{
    return [
        // Key = Header Name
        // Value = Replacement Value
    
        'Content-Type' => 'REDACTED',
    ];
}
```
{% endtab %}

{% tab title="Advanced - Using Closures" %}
```php
protected function defineSensitiveHeaders(): array
{
    return [
        // Key = Header Name
        // Value = Replacement Value / Closure
    
        'Content-Type' => static function (string $value) {
            return substr_replace($value, 'xxx', 1);
        },
    ];
}
```
{% endtab %}
{% endtabs %}

#### Replacing Sensitive JSON Parameters

If the response you are dealing with is JSON - you can define sensitive JSON keys that should be replaced. For example, if there is a "name" key - anytime this is found the value will be replaced with the replacement you define. The keys provided are recursive, so if "name" is within a nested JSON array, it will still be replaced.

{% tabs %}
{% tab title="Simple - Using Arrays" %}
```php
protected function defineSensitiveJsonParameters(): array
{
    return [
        // Key = JSON Key
        // Value = Replacement Value
    
        'name' => 'REDACTED',
    ];
}
```
{% endtab %}

{% tab title="Advanced - Using Closures" %}
```php
protected function defineSensitiveJsonParameters(): array
{
    return [
        // Key = JSON Key
        // Value = Replacement Value / Closure
    
        'name' => static function (string $value) {
            // Example: You could use something like Faker to generate
            // fake information and provide a similar structure.
        
            return faker()->firstName();
        },
    ];
}
```
{% endtab %}
{% endtabs %}

#### Finding and Replacing From Regex Patterns

When the API response you are given is not JSON - it can be difficult to replace the information. When you encounter APIs like these, you can use the `defineSensitiveRegexPatterns` method to find and replace regex patterns in an all-string response body. The key of the array is the regex pattern and the value is the replacement.

{% tabs %}
{% tab title="Simple - Using Arrays" %}
```php
protected function defineSensitiveRegexPatterns(): array
{
    return [
        // Key = Regex Pattern
        // Value = Replacement Value
        
        '/@[a-z0-9_]{0,100}/' => 'REDACTED-TWITTER-HANDLE',
    ];
}
```
{% endtab %}

{% tab title="Advanced - Using Closures" %}
```php
protected function defineSensitiveRegexPatterns(): array
{
    return [
        // Key = Regex Pattern
        // Value = Replacement Value
        
        '/@[a-z0-9_]{0,100}/' => static function (string $value) {
            // Example: You could use something like Faker to generate
            // fake information and provide a similar structure.
        
            return faker()->userName();
        },
    ];
}
```
{% endtab %}
{% endtabs %}

#### Using your custom fixtures

Once you have created your custom fixture class with the redaction configuration, you can simply use it instead of `MockResponse::fixture()` in your tests. All detection methods in the mock client work with this type of fixture too.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$mockClient = new MockClient([
<strong>    GetServerRequest::class => new SingleServerFixture;
</strong>]);
</code></pre>

### Preventing Stray API Requests

Once you have written your tests - it's a good idea to ensure that no real API requests are made in the future while running those tests. This is because you could be making requests when you don't intend to which could incur charges or worse, make real changes to data you don't expect. With Saloon, you can prevent stray API requests with the global `Config` helper. Simply in your tests, call the `Config::preventStrayRequests()` method and you should be good to go!&#x20;

It's recommended that you place this in your `Pest.php` file or in your `setUp()` method to make sure it's used on every test.

{% tabs %}
{% tab title="Pest.php (PEST)" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

<strong>use Saloon\Config;
</strong>
beforeEach(function () {
    Config::preventStrayRequests();
});
</code></pre>
{% endtab %}

{% tab title="TestCase.php (PHPUnit)" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

<strong>use Saloon\Config;
</strong>
class TestCase {

    public function setUp()
    {
<strong>        Config::preventStrayRequests();
</strong>    }

}
</code></pre>
{% endtab %}
{% endtabs %}

### Advanced Usage

You may want to return custom fixtures based on the request without specifying exact names of fixtures. For example, I might want to build a fixture name based on the name of the request being sent. You may use a closure inside the mock client and write the custom logic to meet these needs.

{% tabs %}
{% tab title="Non-Laravel" %}
```php
<?php

$mockClient = new MockClient([
    '*' => function (PendingRequest $request) {
        $reflection = new ReflectionClass($request->getRequest());

        return MockResponse::fixture($reflection->getShortName());
    },
]);

$forge = new ForgeConnector;
$forge->withMockClient($mockClient);

// This will create a fixture called "GetForgeServerRequest"

$request = new GetForgeServerRequest(12345);
$response = $forge->send($request);

// This will create a fixture called "GetAllForgeServersRequest"

$request = new GetAllForgeServersRequest($data);
$response = $forge->send($request);
```
{% endtab %}

{% tab title="Laravel" %}
```php
<?php

Saloon::fake([
    '*' => function (PendingRequest $request) {
        $reflection = new ReflectionClass($request->getRequest());

        return MockResponse::fixture($reflection->getShortName());
    },
]);

$forge = new ForgeConnector;

// This will create a fixture called "GetForgeServerRequest"

$request = new GetForgeServerRequest(12345);
$response = $forge->send($request);

// This will create a fixture called "GetAllForgeServersRequest"

$request = new GetAllForgeServersRequest($data);
$response = $forge->send($request);
```
{% endtab %}
{% endtabs %}

Another example from [Astrotomic's Steam SDK](https://github.com/Astrotomic/steam-sdk) allows you to create a directory for each request. This is really useful for organising your mock fixtures.

```php
<?php

$mockClient = new MockClient([
    '*' => function (PendingRequest $pendingRequest) {
        $name = implode('/', array_filter([
             parse_url($pendingRequest->getUrl(), PHP_URL_HOST),
             mb_strtoupper($pendingRequest->getMethod() ?? 'GET'),
             parse_url($pendingRequest->getUrl(), PHP_URL_PATH),
             http_build_query(array_diff_key($pendingRequest->query()->all(), array_flip(['key', 'format']))),
        ]));
          
        return MockResponse::fixture($name);
    },
]);
```
