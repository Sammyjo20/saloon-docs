# 🎁 What's new in v2

Version two of Saloon is an upgrade in everything you already love in version one but refines and improves your developer experience. It has many internal changes that make it more maintainable and lightweight. This results in a fantastic experience building API integrations or your next SDK.

### Fundamental Changes

#### Namespace and class name changes

Previously, Saloon had "Sammyjo20" in the namespace. This has now been removed so namespaces start with "Saloon". Many of the classes in Saloon had the word "Saloon" in, which has also been dropped, for example, the request and collection.

* SaloonRequest -> Request
* SaloonConnector -> Connector

#### Reducing the dependency on Guzzle

Previously, Saloon was highly dependent on Guzzle as the underlying HTTP Client that sent requests. While this on its own is not inherently _bad_ it meant that Saloon was very closely tied to Guzzle's versioning, support and breaking changes. Version two still requires Guzzle as its primary HTTP Client, but there have been many changes which decouple the library from Guzzle.&#x20;

Saloon used to send a request through a class called  `RequestSender.` this class would build up all the request properties like headers, config and body and would pass it into the Guzzle client. From version two, Saloon now has a new `PendingRequest` class that is responsible for building up all the request properties. After a PendingRequest has been made, it is sent to a `Sender.` This approach means that in theory you could use any HTTP client you like with Saloon, so if the maintainers of Guzzle decided to drop the project entirely, Saloon wouldn't be left in the dark.

While Saloon still uses Guzzle for it's very useful PSR-compliant objects, in future versions of Saloon, it may not use Guzzle as a dependency. The work has already been completed to allow you to use any HTTP Client of your choice.

<figure><img src="../.gitbook/assets/Saloon v2.png" alt=""><figcaption><p>The flow of Saloon v2</p></figcaption></figure>

#### Improving the developer experience

Saloon's purpose is to make it easy for developers to build and maintain API integrations. From simple integrations in a small PHP or Laravel project to building SDKs as separate packages for people to enjoy. It must be fun for a developer to use so there has been many changes made.

One of the notable changes is the simplification of the request and connector API. Previously, the request class had over 50 methods available and many of those methods didn't add much value. With version two, there are now \~28 methods (at the time of writing). Every feature of Saloon has been re-evaluated to decide if it adds value.

Continue to read to see more information on the rest of the changes made in version two.

{% tabs %}
{% tab title="Version 1" %}
```php
array:57 [
  0 => "defineEndpoint"
  1 => "__construct"
  2 => "boot"
  3 => "getMethod"
  4 => "getConnector"
  5 => "setConnector"
  6 => "getFullRequestUrl"
  7 => "traitExistsOnConnector"
  8 => "setIsRecordingFixture"
  9 => "isRecordingFixture"
  10 => "isNotRecordingFixture"
  11 => "__call"
  12 => "make"
  13 => "defaultData"
  14 => "mergeData"
  15 => "setData"
  16 => "addData"
  17 => "getData"
  18 => "ignoreDefaultData"
  19 => "defaultQuery"
  20 => "mergeQuery"
  21 => "setQuery"
  22 => "addQuery"
  23 => "getQuery"
  24 => "ignoreDefaultQuery"
  25 => "defaultHeaders"
  26 => "mergeHeaders"
  27 => "setHeaders"
  28 => "addHeader"
  29 => "getHeaders"
  30 => "getHeader"
  31 => "ignoreDefaultHeaders"
  32 => "defaultConfig"
  33 => "mergeConfig"
  34 => "setConfig"
  35 => "addConfig"
  36 => "getConfig"
  37 => "ignoreDefaultConfig"
  38 => "addHandler"
  39 => "mergeHandlers"
  40 => "getHandlers"
  41 => "addResponseInterceptor"
  42 => "mergeResponseInterceptors"
  43 => "getResponseInterceptors"
  44 => "defaultAuth"
  45 => "getAuthenticator"
  46 => "withAuth"
  47 => "authenticate"
  48 => "withTokenAuth"
  49 => "withBasicAuth"
  50 => "withDigestAuth"
  51 => "getResponseClass"
  52 => "send"
  53 => "sendAsync"
  54 => "getRequestManager"
  55 => "withMockClient"
  56 => "getMockClient"
]
```
{% endtab %}

{% tab title="Version 2" %}
```php
array:28 [
  0 => "__construct"
  1 => "createPendingRequest"
  2 => "sender"
  3 => "send"
  4 => "sendAsync"
  5 => "getMethod"
  6 => "getAuthenticator"
  7 => "authenticateWith"
  8 => "withTokenAuth"
  9 => "withBasicAuth"
  10 => "withDigestAuth"
  11 => "withQueryAuth"
  12 => "headers"
  13 => "queryParameters"
  14 => "config"
  15 => "middleware"
  16 => "createDtoFromResponse"
  17 => "getResponseClass"
  18 => "withMockClient"
  19 => "getMockClient"
  20 => "hasMockClient"
  21 => "when"
  22 => "unless"
  23 => "connector"
  24 => "setConnector"
  25 => "getRequestUrl"
  26 => "boot"
  27 => "make"
]
```
{% endtab %}
{% endtabs %}

#### Connector-first design

With previous versions of Saloon, the recommended way to send requests was with the request, and using the request `send` methods.

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

Although this is being taken out of the request, you may still add the functionality back with the `HasConnector` trait on the request. Although, if you add it back - you need to be aware of the downsides like not being able to have constructor arguments on your connector.

### New Features

#### Middleware Pipeline

Saloon now has its own middleware pipeline which lets you tap into requests before they are sent and responses before they are handed back to the application. This is really handy to add additional logic to all your requests or modify something before a request is sent to the sender. Middleware methods are chain-able and you can use invokable classes for them too.

This replaces the legacy response interceptors. You may still use Guzzle middleware but you must add them from within the connector's constructor. &#x20;

```php
<?php

$request = new GetForgeServerRequest(12345);

$request->middleware()
        ->onRequest(function (PendingRequest $pendingRequest) {
             // Run before the request is sent
        })
        ->onResponse(function (Response $response) {
             // Run after the request has been sent
        });
```

#### Senders

Senders are responsible for taking a PendingRequest and actually sending it. Currently the default sender is the GuzzleSender, but other senders can be built and you can specify the default sender on the connector.

```php
<?php

class ForgeConnnector extends Connector
{
    protected function defaultSender(): Sender
    {
        return new CustomSender;
    }
}
```

#### Request Concurrency & Pooling

Previously, Saloon would create a new Guzzle client for every request. This means that it didn't support concurrent requests as the curl connection was closed after every request. With Saloon v2, the sender remains active on the connector until the connector is destructed. This means that concurrent requests and pools are supported! You can use the connector's `pool` method to create a pool of requests. Pools accept arrays, generators or closures.

```php
<?php

$forge = new ForgeConnector;

$forge->pool()

$pool = $connector->pool([
    new GetForgeServersRequest,
    new GetForgeServersRequest,
    new GetForgeServersRequest,
]);

$pool->setConcurrency(10);

$pool->withResponseHandler(function (Response $response) {
    //
]);

$pool->withExceptionHandler(function (Exception $exception) {
    //
]);

$promise = $pool->send();
```

#### Better Interface Adoption

Saloon v2 now offers interfaces for all of the major classes so you can build your own implementation for unlimited customisation. Some of the interfaces include:

* Request
* Connector
* PendingRequest
* Sender
* Dispatcher
* Response

#### Better Way To Interact With Request, Headers, Query Parameters and Config

Another notable change would be the simplification of interacting with headers, config and request body. Instead of individual methods for interacting with these properties, they are now wrapped in easy-to-understand methods with unified method names. Additionally, previously you wouldn't be able to access the default properties after instantiating the request, but now you can.

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

#### &#x20;Better Way To Interact With Request Body/Data

Additionally, sending request payload/body has been reworked. Previously the same `data` object was used for all types of data, which meant Saloon had to throw exceptions when you use certain methods, for example, if I tried to "add" when using a string body because you can't easily add to a string. Additionally, the `data()` methods were always available even if you didn't add a trait to activate them, which was confusing to the developer and could lead to annoying issues.

With version two, you can now add a trait based on the data type like before, but when you add the trait, it will add the `body()` method. You also add the `HasBody` interface so Saloon knows you intend to send body. The body method implements a contract called `BodyRepository` but depending on the trait added, it used a different implementation of BodyRepository to support the data you have requested. For example, if you add the `HasJsonBody` trait, it will use the `ArrayBodyRepository` which provides the additional methods like add/merge. However, if you use the `HasXmlBody` trait, it will use `StringBodyRepository` which only has a few methods.

This also means that the `defaultBody` method is implemented differently depending on the trait.

```php
<?php

class CreateForgeServerRequest extends Request implements HasBody
{
    use HasJsonBody;
    
    protected function defaultBody(): array
    {
        return [
            'name' => 'Default',
        ];
    }
}

$request = new CreateForgeServerRequest();

$request->body()->add('name', 'Server-One');
$request->body()->add('os', 'Ubuntu');

$request->body()->all();

// Result

[
    'name' => 'Server-One',
    'os' => 'Ubuntu',
]
```

#### Better Multipart Requests

With the changes to request body, Saloon has also made it easier to send multipart form requests for attaching files. Saloon makes this easy by adding the `HasBody` interface and `HasMultipartBody` trait. You can use the `add` method to attach a file to your request.&#x20;

```php
<?php

class UpdateUserRequest extends Request implements HasBody
{
    use HasMultipartBody;
}

$request = new CreateForgeServerRequest();

$request->body()->add(
    name: 'logo',
    contents: 'your-file-contents-or-stream', 
    filename: 'logo.png', 
    headers: [
       // Optional custom headers
    ]
);

$connector = new ForgeConnector;
$connector->send($request);
```

#### Even Better Laravel Support (HTTP Client)

If you are using Saloon in a Laravel environment, and have installed the Saloon-Laravel library, then version two will come with a different sender than the Guzzle sender that you can use. The HTTP Client sender uses Laravel's HTTP Client. Since this is built on top of Guzzle anyway, all request options will work, but all the typical HTTP Client events will be sent, which means other Laravel packages using these events will now work with Saloon. One great example is recording requests in Telescope.

#### Better Exception Handler

Saloon v2 also ships with a completely overhauled exception handler. You can customise when exceptions are thrown and what exceptions are thrown without creating custom response classes. There is also a brand new set of default exceptions that are thrown depending on status codes.

```
SaloonException
├── FatalRequestException (Connection Errors)
└── RequestException (Request Errors)
    ├── ServerException (5xx)
    │   ├── InternalServerErrorException (500)
    │   ├── ServiceUnavailableException (503)
    │   └── GatewayTimeoutException (504)
    └── ClientException (4xx)
        ├── UnauthorizedException (401)
        ├── ForbiddenException (403)
        ├── NotFoundException (404)
        ├── MethodNotAllowedException (405)
        ├── RequestTimeOutException (408)
        ├── UnprocessableEntityException (422)
        └── TooManyRequestsException (429)
```

#### Solo Requests

Version two will also introduce a new `SoloRequest` class which will be perfect for making just one request for API integration. With SoloRequests, you don't need a connector at all - you can define everything in the request and send it above like you used to do.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

class GetPokemonRequest extends SoloRequest
{
     protected Method $method = Method::GET;
     
     public function resolveEndpoint()
     {
          return 'https://pokeapi.co/api/v2/pokemon';
     }
}
```
{% endtab %}

{% tab title="Usage" %}
```php
<?php

// No Connector Needed!

$request = new GetPokemonRequest;
$response = $request->send();
```
{% endtab %}
{% endtabs %}

### Other Improvements / Changes

#### Tidier Codebase

Like with every library, new methods of programming is learned and better patterns are implemented. Saloon version two is pretty much a rewrite and has a much easier-to-understand, tidier codebase.

#### Reduced Dependencies

Saloon previously depended on Laravel's Illuminate Support package and Carbon. These dependencies have now been completely removed which reduce's Saloon's dependency tree and makes it much more lightweight, especially with the removal of the Illuminate Support package as that contains many Laravel-specific class which would be installed in every simple PHP library or SDK.

Saloon now only has three dependencies

* Guzzle (For The Default GuzzleSender)
* Guzzle's Promise Library (For Request Pooling)
* PSR Message Library (For PSR Interfaces)

#### New Query Authenticator

With Saloon v2, a new query parameter authenticator has added, to support some APIs where their authentication is provided through a query parameter.

#### Request Collections & Magic Methods Removed

Previously, Saloon had request collections which can be added to connectors alongside a `$requests` array which allowed you to define custom properties on the connector. These properties would then link directly to a request or a request collection. Saloon v2 has removed this feature in order to have less "magical" code going on. It was also difficult for IDEs to understand.
