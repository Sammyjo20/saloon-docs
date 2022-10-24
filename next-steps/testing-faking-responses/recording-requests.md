# Recording Requests

When writing tests for an API integration, it is best to simulate a real request as much as possible. With Saloon's MockResponse class, you can build up example responses however, this can be time-consuming and if an API returns a huge amount of data, it would take a long time to manually write MockResponses and keep it maintained.

Saloon has a feature called fixture recording, this feature will allow you to make a real request to the API you are integrating with and then it will store that response in a file for later. This is a common practice for people writing integrations for APIs, but Saloon makes it effortless.&#x20;

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

// The initial request will check if a fixture called "singleServer" 
// exists. Because it doesn't exist yet, the real request will be
// sent and the response will be recorded.

$request = new GetForgeServerRequest(12345);
$response = $request->send($mockClient);

// However, the next time the request is made, the fixture will 
// exist, and Saloon will not make the request again.

$request = new GetForgeServerRequest(12345);
$response = $request->send($mockClient);
```
{% endtab %}

{% tab title="Laravel" %}
```php
<?php

Saloon::fake([
    GetForgeServerRequest::class => MockResponse::fixture('singleServer')
]);

// The initial request will check if a fixture called "singleServer" 
// exists. Because it doesn't exist yet, the real request will be
// sent and the response will be recorded.

$request = new GetForgeServerRequest(12345);
$response = $request->send();

// However, the next time the request is made, the fixture will 
// exist, and Saloon will not make the request again.

$request = new GetForgeServerRequest(12345);
$response = $request->send();
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
<?php

MockConfig::setFixturePath('tests/other-directory')
```

#### Preventing Unwanted Requests

Once you have written all of your tests, you might want to prevent accidental API requests in the future for fixtures that don't exist. If you would like Saloon to throw exceptions if a fixture does not exist, you may do this with the MockConfig class.

```php
<?php

MockConfig::throwOnMissingFixtures()
```

### Advanced Usage

You may want to return custom fixtures based on the request without specifying exact names of fixtures. For example, I might want to build a fixture name based on the name of the request being sent. You may use a closure inside the mock client and write the custom logic to meet these needs.&#x20;

{% tabs %}
{% tab title="Non-Laravel" %}
```php
<?php

$mockClient = new MockClient([
    '*' => function (SaloonRequest $request) {
        $reflection = new ReflectionClass($request);

        return MockResponse::fixture($reflection->getShortName());
    },
]);

// This will create a fixture called "GetForgeServerRequest"

$request = new GetForgeServerRequest(12345);
$response = $request->send($mockClient);

// This will create a fixture called "GetAllForgeServersRequest"

$request = new GetAllForgeServersRequest($data);
$response = $request->send($mockClient);
```
{% endtab %}

{% tab title="Laravel" %}
```php
<?php

Saloon::fake([
    '*' => function (SaloonRequest $request) {
        $reflection = new ReflectionClass($request);

        return MockResponse::fixture($reflection->getShortName());
    },
]);

// This will create a fixture called "GetForgeServerRequest"

$request = new GetForgeServerRequest(12345);
$response = $request->send();

// This will create a fixture called "GetAllForgeServersRequest"

$request = new GetAllForgeServersRequest($data);
$response = $request->send();
```
{% endtab %}
{% endtabs %}
