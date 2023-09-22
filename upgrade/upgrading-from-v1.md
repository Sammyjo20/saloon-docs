# 🌿 Upgrading from v1

### Introduction

It's time for the big upgrade to version two! There have been several major changes to Saloon in version two, so it's recommended that you read through [what's new in v2](whats-new-in-v2.md) before starting the upgrade. You'll definitely need a cup of tea, coffee or beer if you fancy.

#### Estimated Upgrade Time

Simple integrations: 15-30 minutes

Advanced integrations and SDKs: \~30-45 minutes

### Need help migrating from version one?

You can [open a discussion](https://github.com/sammyjo20/saloon) on Saloon's Github page if you need any help with the migration process.

### Installation

First, update Saloon in your `composer.json` file to use the version `^2.0`. if you are using the additional Laravel package, you should update this to `^2.0` too. After that, run `composer update`.

Additionally, Saloon has moved to it's own Github organization so the name has been changed from `sammyjo20` to `saloonphp` and the Laravel package has been renamed from `sammyjo20/saloon-laravel` to `saloonphp/laravel-plugin.`

{% tabs %}
{% tab title="Non-Laravel" %}
```json
"require": {
    "saloonphp/saloon": "^2.0"
}
```
{% endtab %}

{% tab title="Laravel" %}
```json
"require": {
    "saloonphp/saloon": "^2.0",
    "saloonphp/laravel-plugin": "^2.0"
}
```

> If you previously used `sammyjo20/saloon-laravel` or `saloonphp/laravel-plugin` it's recommended that you add "saloonphp/saloon" as an additional required dependency.
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
* Replace: `use Saloon\Http\Faking\MockResponse`

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

The `request` method has been removed.

The `__call` and `__callStatic` methods have been removed alongside the magic methods that build up requests. Including the `requests` property.

### Request Changes

<mark style="color:red;">Estimated Impact: High</mark>

#### Request Methods

The request’s `defineEndpoint` method has changed.

* Find: `public function defineEndpoint(): string`
* Replace: `public function resolveEndpoint(): string`

The method to define a custom response has changed.

* Find: `public function getResponseClass(): string`
* Replace: `public function resolveResponseClass(): string`

The `send`, `sendAsync`, `getConnector` and `setConnector` methods and the `connector` property has been removed from the request. Please see below to add connector support back to your request if you need it

The `getFullRequestUrl` method has been removed from the request. You can get the URL of the request with the `PendingRequest` inside of the boot method, traits and middleware.

The `__call` method has been removed from the request. Any methods that no longer exist on the request will not be proxied to the connector.

The `traitExistsOnConnector` method has been removed from the request.

#### Request Properties

The request’s method property has changed to use a new `Saloon\Enums\Method` Enum. Make sure to migrate to use the new Enum too. For example: `protected Method $method = Method::GET`.

* Find: `protected ?string $method`
* Replace: `protected Method $method`

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

$request->query()->add($value);
$request->query()->get($value, $default);
$request->query()->set($value);
$request->query()->merge(...$values);
$request->query()->all();

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

Saloon has also rebuilt the way that request data/body is sent using POST/PUT/PATCH requests. First, make sure that your data traits are using the new namespaces. It's recommended that you [read through the new section](../the-basics/request-body-data/) on request body/data to understand why the changes have been made.

**HasJsonBody**

* Find: `use Saloon\Traits\Plugins\HasJsonBody`
* Replace: `Saloon\Traits\Body\HasJsonBody`

**HasFormParams**

* Find: `use Saloon\Traits\Plugins\HasFormParams`
* Replace: `use Saloon\Traits\Body\HasFormBody`

**HasMultipartBody**

* Find: `use Saloon\Traits\Plugins\HasMultipartBody`
* Replace: `use Saloon\Traits\Body\HasMultipartBody`

**HasXMLBody**

* Find: `use Saloon\Traits\Plugins\HasXMLBody`
* Replace: `use Saloon\Traits\Body\HasXmlBody`
* Replace: `use HasXMLBody`
* Find: `use HasXmlBody`

**HasBody**

* Find: `use Saloon\Traits\Plugins\HasBody`
* Replace: `use Saloon\Traits\Body\HasBody`

Next, add the `HasBody` interface to your request or connector. This interface is required for Saloon to properly detect if you are using request body or not. Since you have already added a body trait, the required `body` method should be implemented. You may need to re-index your IDE before it understands the changes made.

#### **Syntax changed from data to body**

Previously, Saloon called request body "data". To match PSR standards better, this has been renamed to "body".

#### Changing default

Previously, you may have defined a method like `defaultData` this needs to be renamed to `defaultBody`. The methods have also changed from being public to protected.

{% tabs %}
{% tab title="Version One" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

class GetServersRequest extends Request
{
    // {...}
    
<strong>    public function defaultData(): array    
</strong>    {
        return [
            // ...
        ];
    }
}
</code></pre>
{% endtab %}

{% tab title="Version Two" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

class GetServersRequest extends Request
{
    // {...}
    
<strong>    public function defaultBody(): array
</strong>    {
        return [
            // ...
        ];
    }
}
</code></pre>
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Make sure that you define the correct return type for your request body trait.
{% endhint %}

#### HasXMLBody defineXmlBody removed

Saloon has removed the `defineXmlBody` method when you use the `HasXmlTrait`. You must replace this with `defaultBody`

#### Request Body Methods

Saloon has also changed the previous request data methods. You should update your code accordingly if you used these old methods.

{% tabs %}
{% tab title="Version One" %}
```php
<?php

$request = new CreateForgeSiteRequest($serverId, $domain);

$request->setData(['domain' => $customDomain]);
$request->mergeData(['database' => 'test123']);
$request->addData('name', 'my-saloon-server');
$request->getData('name');
```
{% endtab %}

{% tab title="Version Two" %}
```php
<?php

$request = new CreateForgeSiteRequest($serverId, $domain);

$request->body()->set(['domain' => $customDomain]);
$request->body()->merge(['database' => 'test123']);
$request->body()->add('name', 'my-saloon-server');
$request->body()->get('name');
```
{% endtab %}
{% endtabs %}

### Removing Connector Magic Properties & Request Collections

<mark style="color:red;">Estimated Impact: High</mark>

With regards to request collections/request groups, Saloon has removed support for them entirely in v2. Previously, Saloon had a lot of "magic" logic which was cool, but tricky for IDEs to support. As request collections were just classes that passed in the connector, it's recommended that you create your own classes that support this, and then add methods into your connector.

[Click here for an example SDK resource instead of request collections](../digging-deepeer/building-sdks.md#resources)

### Authentication

<mark style="color:red;">Estimated Impact: High</mark>

Saloon version two has removed the `withAuth` method. You should use the `authenticate` method instead.

### Response Interceptors

<mark style="color:red;">Estimated Impact: High</mark>

Previously, Saloon had the concept of `ResponseInterceptors` which were functions that Saloon would call before returning the response back to the application. This API has been removed in favour of using the new [Middleware API](../digging-deeper/middleware.md). It's recommended that you get yourself familiar with middleware, but here is an example of migrating from response interceptors to response middleware.

{% tabs %}
{% tab title="Version One" %}
```php
<?php

$request->addResponseInterceptor(function (SaloonRequest $request, SaloonResponse $response) {
    $response->throw();
    
    return $response;
});
```
{% endtab %}

{% tab title="Version Two" %}
```php
<?php

use Saloon\Contracts\Response;

$request->middleware()->onResponse(function (Response $response) {
    $response->throw();
});
```
{% endtab %}
{% endtabs %}

### AlwaysThrowOnErrors Trait Rename

<mark style="color:red;">Estimated Impact: High</mark>

From Saloon version two, the `AlwaysThrowsOnErrors` trait has been renamed to `AlwaysThrowOnErrors`.

* Find: `Saloon\Traits\Plugins\AlwaysThrowsOnErrors`
* Replace: `Saloon\Traits\Plugins\AlwaysThrowOnErrors`
* Find: `use AlwaysThrowsOnErrors`
* Replace: `use AlwaysThrowOnErrors`

### Boot Method Arguments

<mark style="color:red;">Estimated Impact: High</mark>

Saloon has a method that you can add on your connector and request to write logic while a request is being sent. This `boot` method has changed arguments in version two. It used to provide you with an instance of `Request` but will now provide you with an instance of `PendingRequst.` You should ensure any modifications are made on this PendingRequest instance and not use `$this` or modify the connector/request.

{% tabs %}
{% tab title="Version One" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

class CreateForgeServerRequest extends SaloonRequest
{
    // {...}

<strong>    public function boot(SaloonRequest $request): void
</strong>    {
        $request->addHeader('X-Example', 'Hello');
    }
}
</code></pre>
{% endtab %}

{% tab title="Version Two" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Contracts\PendingRequest;

class CreateForgeServerRequest extends Request
{
    // {...}

<strong>    public function boot(PendingRequest $pendingRequest): void
</strong>    {
        $pendingRequest->headers()->add('X-Example', 'Hello');
    }
}
</code></pre>
{% endtab %}
{% endtabs %}

### Caching Plugin

<mark style="color:red;">Estimated Impact: High</mark>

Saloon's caching plugin has also had a full overhaul to work with Saloon v2. It's recommended that you follow the steps for configuring the new caching plugin in the [documentation here.](../plugins/caching-responses.md)

### Data Transfer Objects

<mark style="color:red;">Estimated Impact: High</mark>

Previously, Saloon provided data transfer objects through a `CastsToDto` trait. This trait could be added to the connector or request and would allow you to define a `castToDto` method. From Saloon v2, DTO casting is a built-in feature for every connector and request and the `CastsToDto` trait has been removed. All you need to do is change the **protected**  `castToDto` method to a **public** `createDtoFromResponse` method.

{% tabs %}
{% tab title="Version One" %}
```php
<?php

use Sammyjo20\Saloon\Traits\Plugins\CastsToDto;

class ForgeConnector extends SaloonConenctor
{
    use CastsToDto;
    
    protected function castToDto(SaloonResponse $response): mixed
    {
        // ...
    }
}
```
{% endtab %}

{% tab title="Version Two" %}
```php
<?php

class ForgeConnector extends SaloonConenctor
{
    protected function createDtoFromResponse(SaloonResponse $response): mixed
    {
        // ...
    }
}
```
{% endtab %}
{% endtabs %}

### Responses

<mark style="color:purple;">Estimated Impact: Medium</mark>

Saloon’s `Response` class has changed to be a more generic, PSR-compatible response. If you are extending the existing Response class, you should make sure that it is still working correctly.

### Plugin Traits

<mark style="color:purple;">Estimated Impact: Medium</mark>

From version two, Saloon has updated its plugins. You can choose to add plugins to both your connector or your request. Previously, plugins would receive an instance of `SaloonRequest` in the arguments. Now, plugins will receive a `PendingRequest` instance. You should update your plugins accordingly.

You should also make any changes to the `PendingRequest` instance and **not** use `$this` as it's bad practice to overwrite the connector/request instance.

### Guzzle Handlers/Middleware

<mark style="color:purple;">Estimated Impact: Medium</mark>

Previously, Saloon allowed you to use the `addHandler` method to use a Guzzle middleware. From version two, Guzzle middleware is still supported with the default GuzzleSender, but you must migrate your handlers to the new API.

It's also recommended that you move any Guzzle middleware from requests into your connector class as middleware should only be executed once.

{% tabs %}
{% tab title="Version One" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

class Forge extends SaloonConnector
{
    //...

    public function boot(SaloonRequest $request): void
    {
<strong>        $this->addHandler('customHeaderHandler', function (callable $handler) {
</strong>            return function (RequestInterface $request, array $options) use ($handler) {
                $request->withHeader('X-Custom-Header', 'Hello');
                
                return $handler($request, $options);             
            };
        });
    }
}
</code></pre>
{% endtab %}

{% tab title="Version Two" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

class Forge extends Connector
{
    //...

    public function boot(PendingRequest $pendingRequest): void
    {
<strong>        $pendingRequest->sender()->addMiddleware(function (callable $handler) {
</strong>            return function (RequestInterface $request, array $options) use ($handler) {
                $request->withHeader('X-Custom-Header', 'Hello');
                
                return $handler($request, $options);             
            };
<strong>        }, 'customHandlerMiddleware');
</strong>    }
}
</code></pre>
{% endtab %}
{% endtabs %}

### Authenticator Traits

<mark style="color:blue;">Estimated Impact: Low</mark>

Previously, Saloon had five traits which would throw an exception if a request or connector wasn’t authenticated. The following traits have now been removed:

* RequiresBasicAuth
* RequiresDigestAuth
* RequiresTokenAuth

You should now use the generic `RequiresAuth` trait if you would still like to throw an exception.

### OAuth Carbon Removal

<mark style="color:blue;">Estimated Impact: Low</mark>

Saloon no longer has Carbon as a dependency, so all dates returned that used to return a `CarbonInterface` now return `DateTimeImmutable`

* OAuthAuthenticator: `getExpiresAt()`
* AccessTokenAuthenticator: `getExpiresAt()`

### Mock Response From Request

<mark style="color:blue;">Estimated Impact: Low</mark>

The `MockResponse::fromRequest` method has been removed from version two.
