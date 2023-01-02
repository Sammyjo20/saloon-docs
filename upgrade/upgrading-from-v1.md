# 🌿 Upgrading from v1

### Introduction

There have been several changes in version two, it's recommended that you read through [what's new in v2](whats-new-in-v2.md) before starting the upgrade.

### Estimated Upgrade Time

Simple applications/Laravel integrations: \~15-30 minutes

Advanced integrations and SDKs: \~30-45 minutes

### Installation

Firstly, update your composer.json file to `^2.0.` if you are using the additional Laravel package, you should update this to `^2.0` too. After that, run `composer update`.

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
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
From version two, the minimum required PHP version is 8.1 and the minimum Laravel version is 9 (when using the additional Laravel package)
{% endhint %}

### Namespace Changes

All of Saloon’s classes have a new namespace `Saloon\\` instead of `Sammyjo20\\Saloon`. This change will affect every use of Saloon’s internal classes, so it’s recommended to run a find-and-replace.

* Find: `use Sammyjo20\\Saloon`
* Replace: `use Saloon`

#### Laravel Namespace Changes

If you are using the Laravel helpers package for Saloon, you should also change the namespaces.

* Find: `use Sammyjo20\\SaloonLaravel`
* Replace: `use Saloon\\Laravel`

### Class Name Changes

To help make Saloon more readable and to improve the developer experience, Saloon’s classes have changed names. There were a number of classes that had the name `Saloon` within, like `SaloonRequest` and `SaloonConnector.` you should find and replace these too. Please make sure that you have renamed the namespaces first.

#### Connector

* Find: `use Saloon\\Http\\SaloonConnector`
* Replace: `use Saloon\\Http\\Connector`
* Find: `extends SaloonConnector`
* Replace: `extends Connector`

#### Request

* Find: `use Saloon\\Http\\SaloonRequest`
* Replace: `use Saloon\\Http\\Request`
* Find: `extends SaloonRequest`
* Replace: `extends Request`

#### Response (if using custom responses)

* Find `use Saloon\\Http\\SaloonResponse`
* Replace: `use Saloon\\Http\\Response`
* Find: `extends SaloonResponse`
* Replace: `extends Response`

#### MockClient

* Find: `use Saloon\\Clients\\MockClient`
* Replace: `use Saloon\\Http\\Faking\\MockClient`

#### MockResponse

* Find: `use Saloon\\Http\\MockRespone`
* Replace: `use Salooon\\Http\\Faking\\MockResponse`

### Method Changes

Saloon has also had a major refactor with the methods that are used to build and interact with connectors and requests. You should carefully find and replace the given strings.

#### Connector

The connector’s base URL method has changed.

* Find: `public function defineBaseUrl(): string`
* Replace: `public function resolveBaseUrl(): string`

The connector’s properties types have changed.

* Find: `protected ?string $response`
* Replace: `protected string $response`

The method to define a custom response has changed.

* Find: `public function getResponseClass(): string`
* Replace: `public function resolveResponseClass(): string`

The `boot` method’s single argument has changed from `SaloonRequest $request` to use a `Saloon\\Http\\PendingRequest` instead.

* Find: `public function boot(SaloonRequest $request): void`
* Replace: `public function boot(PendingRequest $pendingRequest): void`

The `request` method has been removed.

The `__call` and `__callStatic` methods have been removed alongside the magic methods that build up requests. Including the `requests` property.

#### Request

The request’s method property has changed to use a new `Saloon\\Enums\\Method` Enum. Make sure to migrate to use the new Enum too. For example: `protected Method $method = Method::GET`.

* Find: `protected ?string $method`
* Replace: `protected Method $method`

The request’s define endpoint method has changed.

* Find: `protected function defineEndpoint(): string`
* Replace: `public function resolveEndpoint(): string`

The request’s properties types have changed.

* Find: `protected ?string $response`
* Replace: `protected string $response`

The method to define a custom response has changed.

* Find: `public function getResponseClass(): string`
* Replace: `public function resolveResponseClass(): string`

The `boot` method’s single argument has changed from `SaloonRequest $request` to use a `Saloon\\Http\\PendingRequest` instead.

* Find: `public function boot(SaloonRequest $request): void`
* Replace: `public function boot(PendingRequest $pendingRequest): void`

The `send`, `sendAsync`, `getConnector` and `setConnector` methods and the `connector` property has been removed from the request. Please see below to add connector support back to your request if you need it

The `getFullRequestUrl` method has been removed on the request. You can get the URL of the request with the `PendingRequest` inside of the boot method, traits and middleware.

The `__call` method has been removed from the request. Any methods that no longer exist on the request will not be proxied to the connector.

The `traitExistsOnConnector` method has been removed from the request.

#### Responses

Saloon’s response has changed to be a generic-PSR compatible response. If you are extending the existing Response, you should make sure that it is still working correctly.

#### Authenticator Traits

Previously, Saloon had five traits which would throw an exception if a request or connector wasn’t authenticated. The following traits have now been removed:

* RequiresBasicAuth
* RequiresDigestAuth
* RequiresTokenAuth

You should now use the generic `RequiresAuth` trait if you would still like to throw an exception

### Plugin Traits

* Now is static method
* Now accepts PendingRequest

### Connector-first design

#### Add connector support back

### Response Interceptors

### Guzzle Handlers/Middleware

* No longer add handlers on a per-request or connector basis, you must add it to the sender

### Migrating to the new headers, query and config API

### Migrating to the new request body API

* No more data() method and only added once you add a body trait

#### Migrating to body traits

### OAuth Carbon Removal

Saloon no longer uses Carbon as a dependency, so all dates returned that used to return a `CarbonInterface` now return `DateTimeImmutable`

* OAuthAuthenticator: `getExpiresAt()`
* AccessTokenAuthenticator: `getExpiresAt()`

### Removing Connector Magic Properties

### Migrating to use PendingRequest in traits

### Other Changes

Namespace renaming

Class renaming

Changing data to body and using WithBody interface

Changing location of body traits

Changing defineEndpoint to resolveEndpoint

Changing defineBaseUrl to resolveBaseUrl

Removing the connector property on requests or adding back HasConnector

Changing response interceptors to middleware

Moving where Guzzle handlers are added

withAuth renamed to authenticate

Authenticators are executed at the end of a request preparation lifecycle rather than at the beginning

toPsrResponse renamed to getPsrResponse on responses

defineXMLBody renamed to defaultBody

Request collections and magic request registration removed

MockResponse::fromRequest has been removed

Carbon has been replaced with DateTime instances

MockClient and MockResponse moved to a different folder

Request changed to PendingRequest inside of plugins and boot methods
