# 🌿 Upgrading from v1

{% hint style="warning" %}
This documentation is still a work in progress while Saloon v2 is in beta.
{% endhint %}

### Introduction

It's time for the big upgrade to version two! There have been several major changes to Saloon in version two, so it's recommended that you read through [what's new in v2](whats-new-in-v2.md) before starting the upgrade. You'll definitely need a cup of tea, coffee or beer if you fancy.&#x20;

#### Estimated Upgrade Time

Simple integrations: 15-30 minutes

Advanced integrations and SDKs: \~30-45 minutes

### Installation

First, update Saloon in your `composer.json` file to use the version `^2.0.` if you are using the additional Laravel package, you should update this to `^2.0` too. After that, run `composer update`.&#x20;

{% tabs %}
{% tab title="Non-Laravel" %}
```json
"require": {
    "sammyjo20/saloon": "^2.0"
}
```
{% endtab %}

{% tab title="Laravel" %}
```json
"require": {
    "sammyjo20/saloon": "^2.0",
    "sammyjo20/saloon-laravel": "^2.0"
}
```

> If you previously used just "saloon-laravel" it's recommended that you add "sammyjo20/saloon" as an additional dependency to your require block.
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
Version two requires PHP 8.1 and Laravel 9 if you are using the additional Laravel helper package.
{% endhint %}

### Namespace Changes

<mark style="color:red;">Estimated Impact: High</mark>

All of Saloon’s classes have a new namespace `Saloon` instead of `Sammyjo20\Saloon`. This change will affect every use of Saloon’s internal classes, so it’s recommended to run a find-and-replace.

* Find: `use Sammyjo20\Saloon`
* Replace: `use Saloon`

#### Laravel Namespace Changes

<mark style="color:red;">Estimated Impact: High</mark>

If you are using the Laravel helpers package for Saloon, you should also change the namespaces.

* Find: `use Sammyjo20\SaloonLaravel`
* Replace: `use Saloon\Laravel`

### Class Name Changes

<mark style="color:red;">Estimated Impact: High</mark>

To help make Saloon more readable and to improve the developer experience, Saloon’s classes have changed names. There were a number of classes that had the name `Saloon` within, like `SaloonRequest` and `SaloonConnector.` you should find and replace these too. Please make sure that you have renamed the namespaces as instructed above first.

#### Connector

* Find: `use Saloon\Http\SaloonConnector`
* Replace: `use Saloon\Http\Connector`
* Find: `extends SaloonConnector`
* Replace: `extends Connector`

#### Request

* Find: `use Saloon\Http\SaloonRequest`
* Replace: `use Saloon\Http\Request`
* Find: `extends SaloonRequest`
* Replace: `extends Request`

#### MockClient

* Find: `use Saloon\Clients\MockClient`
* Replace: `use Saloon\Http\Faking\MockClient`

#### MockResponse

* Find: `use Saloon\Http\MockResponse`
* Replace: `use Salooon\Http\Faking\MockResponse`

#### Response (if using custom responses)

* Find `use Saloon\Http\SaloonResponse`
* Replace: `use Saloon\Http\Response`
* Find: `extends SaloonResponse`
* Replace: `extends Response`

### Connector Changes

<mark style="color:red;">Estimated Impact: High</mark>

Saloon has also had a major refactor with the methods that are used to build and interact with connectors and requests. You should carefully find and replace the given strings.

#### Connector Methods

The connector’s base URL method has changed.

* Find: `public function defineBaseUrl(): string`
* Replace: `public function resolveBaseUrl(): string`

The method to define a custom response has changed.

* Find: `public function getResponseClass(): string`
* Replace: `public function resolveResponseClass(): string`

If you are using the `boot` method, the argument has changed from `SaloonRequest $request` to use a `Saloon\Contracts\PendingRequest` instead.

* Find: `public function boot(SaloonRequest $request): void`
* Replace: `public function boot(PendingRequest $pendingRequest): void`

The `request` method has been removed.

The `__call` and `__callStatic` methods have been removed alongside the magic methods that build up requests. Including the `requests` property.

#### Connector Properties

The connector’s properties types have changed.

* Find: `protected ?string $response`
* Replace: `protected string $response`

### Request Changes

<mark style="color:red;">Estimated Impact: High</mark>

#### Request Methods

The request’s `defineEndpoint` method has changed.

* Find: `public function defineEndpoint(): string`
* Replace: `public function resolveEndpoint(): string`

The method to define a custom response has changed.

* Find: `public function getResponseClass(): string`
* Replace: `public function resolveResponseClass(): string`

The `boot` method’s single argument has changed from `SaloonRequest $request` to use a `Saloon\Contracts\PendingRequest` instead.

* Find: `public function boot(SaloonRequest $request): void`
* Replace: `public function boot(PendingRequest $pendingRequest): void`

The `send`, `sendAsync`, `getConnector` and `setConnector` methods and the `connector` property has been removed from the request. Please see below to add connector support back to your request if you need it

The `getFullRequestUrl` method has been removed from the request. You can get the URL of the request with the `PendingRequest` inside of the boot method, traits and middleware.

The `__call` method has been removed from the request. Any methods that no longer exist on the request will not be proxied to the connector.

The `traitExistsOnConnector` method has been removed from the request.

#### Request Properties

The request’s method property has changed to use a new `Saloon\Enums\Method` Enum. Make sure to migrate to use the new Enum too. For example: `protected Method $method = Method::GET`.

* Find: `protected ?string $method`
* Replace: `protected Method $method`

The request’s property types have changed.

* Find: `protected ?string $response`
* Replace: `protected string $response`

### Updated way to send requests

<mark style="color:red;">Estimated Impact: High</mark>

With previous versions of Saloon, the recommended way to send requests was with the request and using the request `send` methods.

```php
$request = new UserRequest;
$response = $request->send();
```

One of the points developers found frustrating was defining a connector class on every request that you make. This was solely so you could make a request directly without instantiating the connector.

This approach was very minimalist, but it introduced complexity and friction for the developer.

From version two, the connector property is being dropped entirely from the request. This means that you must send your requests through the connector like this:

```php
$connector = new TwitterConnector;
$response = $connector->send(new UserRequest);

// or

TwitterConnector::make()->send(new UserRequest);
```

This allows you to have constructor arguments on the connector, perfect for API tokens or configuration. Similar to before, the request can have its own headers, config, query parameters and body but the connector will provide the very top-level defaults.

You should make sure that your requests use this new way of sending requests.

#### Using Request-First Sending

Although this is being taken out of the request, you may still add the functionality back with the `HasConnector` trait on the request. Although, if you add it back - you need to be aware of the downsides like not being able to have constructor arguments on your connector.

### Updated Request, Headers, Query Parameters and Config Methods

<mark style="color:red;">Estimated Impact: High</mark>

Another notable change would be the simplification of interacting with headers, config and request body. Instead of individual methods for interacting with these properties, they are now wrapped in easy-to-understand methods with unified method names. Additionally, previously you wouldn't be able to access the default properties after instantiating the request, but now you can. You should make sure any references to headers, query parameters or config use the new methods.

{% tabs %}
{% tab title="Version 1" %}
```php
<?php

$request = new GetForgeServerRequest(12345);

$request->addHeader($value);
$request->getHeader($value);
$request->setHeaders($value);
$request->mergeHeaders(...$values);
$request->getHeaders();

$request->addQuery($value);
$request->getQuery(?$value);
$request->setQuery($value);
$request->mergeQuery(...$values);

$request->addConfig($value);
$request->getConfig(?$value);
$request->setConfig($value);
$request->mergeConfig(...$values);

$request->addData($value);
$request->getData(?$value);
$request->setData($value);
$request->mergeData(...$values);
```
{% endtab %}

{% tab title="Version 2" %}
```php
<?php

$request = new GetForgeServerRequest(12345);

$request->headers()->add($value);
$request->headers()->get($value, $default);
$request->headers()->set($value);
$request->headers()->merge(...$values);
$request->headers()->all();

$request->queryParameters()->add($value);
$request->queryParameters()->get($value, $default);
$request->queryParameters()->set($value);
$request->queryParameters()->merge(...$values);
$request->queryParameters()->all();

$request->config()->add($value);
$request->config()->get($value, $default);
$request->config()->set($value);
$request->config()->merge(...$values);
$request->config()->all();

// Data has been moved to body()... more on that below
```
{% endtab %}
{% endtabs %}

### Migrating to the new request body API

<mark style="color:red;">Estimated Impact: High</mark>

\-- SECTION IN PROGRESS --

* No more data() method and only added once you add a body trait
* Changing data to body and using HasBody interface
* Changing location of body traits
* defineXMLBody renamed to defaultBody

### Removing Connector Magic Properties & Request Collections

<mark style="color:red;">Estimated Impact: High</mark>

With regards to request collections/request groups, Saloon has removed support for them entirely in v2. Previously, Saloon had a lot of "magic" logic which was cool, but tricky for IDEs to support. As request collections were just classes that passed in the connector, it's recommended that you create your own classes that support this, and then add methods into your connector. For example:

```php
// Properties Example
// e.g $forge->servers->get();

class Forge extends Connector
{
    public ServersResource $servers;

    public function __construct()
    {
        $this->servers = new ServersResource($this);
    }
}

// Method example
// e.g $forge->servers()->get();

class Forge extends Connector
{
    public function servers(): ServersResource
    {
         return new ServersResource($this);
    }
}

// Resource Class

use Saloon\Contracts\Connector;
use Saloon\Contracts\Response;

class Resource 
{
    public function __construct(
           protected Connector $connector;
    }{}
    
    public function get()
    {
        return $this->connector->send(new GetServersRequest);
    }
}
```

### Authentication

<mark style="color:red;">Estimated Impact: High</mark>

Saloon version two has removed the `withAuth` method. You should use the `authenticate` method instead.

### Response Interceptors

<mark style="color:red;">Estimated Impact: High</mark>

\-- SECTION IN PROGRESS --

### Responses

<mark style="color:purple;">Estimated Impact: Medium</mark>

Saloon’s `Response` class has changed to be a more generic, PSR-compatible response. If you are extending the existing Response class, you should make sure that it is still working correctly.&#x20;

* toPsrResponse renamed to getPsrResponse on responses

### Plugin Traits

<mark style="color:purple;">Estimated Impact: Medium</mark>

From version two, Saloon has updated its plugins. You can choose to add plugins to both your connector or your request. Previously, plugins would receive an instance of `SaloonRequest` in the arguments. Now, plugins will return a `PendingRequest` instance. You should update your plugins accordingly.&#x20;

You should also make any changes to the `PendingRequest` instance and **not** use `$this` as it's bad practice to overwrite the connector/request instance.

### Guzzle Handlers/Middleware

<mark style="color:purple;">Estimated Impact: Medium</mark>

* No longer add handlers on a per-request or connector basis, you must add it to the sender
* You should use Saloon's onRequest middleware instead-but if you need to access Guzzle's middleware you can do it&#x20;

### Authenticator Traits

<mark style="color:blue;">Estimated Impact: Low</mark>

Previously, Saloon had five traits which would throw an exception if a request or connector wasn’t authenticated. The following traits have now been removed:

* RequiresBasicAuth
* RequiresDigestAuth
* RequiresTokenAuth

You should now use the generic `RequiresAuth` trait if you would still like to throw an exception.

#### Migrating to body traits

### OAuth Carbon Removal

<mark style="color:blue;">Estimated Impact: Low</mark>

Saloon no longer has Carbon as a dependency, so all dates returned that used to return a `CarbonInterface` now return `DateTimeImmutable`

* OAuthAuthenticator: `getExpiresAt()`
* AccessTokenAuthenticator: `getExpiresAt()`

### Notes

Authenticators are executed at the end of a request preparation lifecycle rather than at the beginning

MockResponse::fromRequest has been removed

Carbon has been replaced with DateTime instances

MockClient and MockResponse moved to a different folder

Request changed to PendingRequest inside of plugins and boot methods
