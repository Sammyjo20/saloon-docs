# 🤠 Lawman

Lawman is a [PestPHP](https://pestphp.com/) Plugin for SaloonPHP which allows you to easily write architecture tests for your API integrations with a focus on them being easy to write and easy to read. After all, if SaloonPHP makes our API integrations beautiful, the tests for them should be beautiful too, right?

Lawman is a third-party plugin written and maintained by [Jon Purvis](https://jonathanpurvis.co.uk) and gives you access to Expectations for your Connectors, Requests, Plugins, Authenticators, DTOs, Middleware and more, to ensure they stick to whichever architecture rules your application has.

One of the great things about Lawman is because it's a Pest plugin, it will work with regular PestPHP Expectations, just chain them up and you're good to go!

### Installation

You can install the plugin using Composer. If you run the following command, it will install it as a dev dependency:

```bash
composer require jonpurvis/lawman --dev
```

Lawman makes use of autoloading with Composer, so once you've installed the package, the Expectations should appear as you type!

### Examples

Let's take a look at how you can use Lawman to easily write Arch tests for your API Integrations.

Let's say we have a Connector class that we want to test, with PestPHP we could do the following:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toExtend('Saloon\Http\Connector')
    ->toUse('Saloon\Traits\Plugins\AcceptsJson')
    ->toUse('Saloon\Traits\Plugins\AlwaysThrowOnErrors');
```

So that test is ensuring our class extends the base Connector and uses the `AcceptsJson` and `AlwaysThrowOnErrors` traits. Whilst that test works, we could perhaps make it quicker to write and easier to read, so with Lawman, you can do:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toBeSaloonConnector()
    ->toUseAcceptsJsonTrait()
    ->toUseAlwaysThrowOnErrorsTrait();
```

Next up, let's take a Request test that we have:

```php
test('request')
    ->expect('App\Http\Integrations\Integration\Requests\Request')
    ->toExtend('\Saloon\Http\Request')
    ->toImplement('Saloon\Contracts\Body\HasBody')
    ->toUse('Saloon\Traits\Body\HasFormBody')
    ->toUse('Saloon\Traits\Plugins\AcceptsJson');
```

Lawman makes this test much nicer to read:

```php
test('request')
    ->expect('App\Http\Integrations\Integration\Requests\Request')
    ->toBeSaloonRequest()
    ->toSendPostRequest()
    ->toHaveFormBody()
    ->toUseAcceptsJsonTrait();
```

What about if we want to test our Connector has an Authentication method? Lawman makes this easy to do, including dedicated helpers for multi auth and custom authenticators:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toBeSaloonConnector()
    ->toUseMultipleAuthenticators()
    ->toUseCertificateAuthentication()
    ->toUseTokenAuthentication();

test('authenticator')
    ->expect('App\Http\Integrations\Integration\Auth\ForgeAuthenticator')
    ->toBeSaloonAuthenticator();
```

Lawman also has Expectations for the Pagination, Cache and Rate Limit Plugins:

```php
test('request')
    ->expect('App\Http\Integrations\Integration\Requests\Request')
    ->toBeSaloonRequest()
    ->toBePaginatable()
    ->toHaveCaching()
    ->toHaveRateLimits();

test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toBeSaloonConnector()
    ->toUsePagedPagination();
```

Maybe our Connector has some Retry instructions that we want to test. Again, with Lawman, it's as simple as:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toBeSaloonConnector()
    ->toBeTriedAgainOnFailure(tries: 3)
    ->toHaveRetryInterval(retryInterval: 1500)
    ->toUseExponentialBackoff();
```

Maybe our Connector is for an API that tends to be quite slow, so we increased some timeouts:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toBeSaloonConnector()
    ->toSetConnectTimeout(connectTimeout: 30)
    ->toSetRequestTimeout(requestTimeout: 60);
```

You can also assert DTOs, Solo Requests, SDK resources, middleware, and Saloon plugins:

```php
test('dto')
    ->expect('App\Http\Integrations\Integration\Data\Server')
    ->toBeSaloonDto();

test('solo request')
    ->expect('App\Http\Integrations\Integration\Requests\GetPokemonRequest')
    ->toBeSoloRequest()
    ->toHaveEndpoint();

test('resource')
    ->expect('App\Http\Integrations\Integration\Resources\PokemonResource')
    ->toBeSaloonResource();

test('middleware')
    ->expect('App\Http\Integrations\Integration\Middleware\LogResponse')
    ->toBeResponseMiddleware();

test('plugin')
    ->expect('App\Http\Integrations\Integration\Plugins\HasLogging')
    ->toBeSaloonPlugin();
```

#### PHPStan Support

Lawman ships with a PHPStan extension so expectations like `toBeSaloonConnector()` and `toSendPostRequest()` are recognised when you analyse your test suite. If your project uses [`phpstan/extension-installer`](https://github.com/phpstan/extension-installer), the extension is registered automatically. Otherwise, add the following to your `phpstan.neon` file:

```neon
includes:
    - vendor/jonpurvis/lawman/extension.neon
```

### Available Expectations

Lawman currently provides **78** Expectations.

#### **Authentication**

* `toUseTokenAuthentication`
* `toUseBasicAuthentication`
* `toUseCertificateAuthentication`
* `toUseHeaderAuthentication`
* `toUseQueryAuthentication`
* `toUseDigestAuthentication`
* `toUseMultipleAuthenticators`
* `toUseAccessTokenAuthentication`
* `toUseNullAuthentication`
* `toBeSaloonAuthenticator`
* `toBeOAuthAuthenticator`

#### **Cache**

* `toHaveCaching`

#### **Connector**

* `toBeSaloonConnector`
* `toHaveDefaultHeaders`
* `toHaveDefaultConfig`
* `toHaveBaseUrl`
* `toUseCustomResponse`
* `toHaveCustomFailureDetection`
* `toHaveCustomException`
* `toHaveDefaultAuth`
* `toHaveDefaultOauthConfig`
* `toBeSaloonResource`
* `toHaveBootMethod`
* `toHandlePsrRequest`
* `toHaveDefaultSender`

#### **Pagination**

* `toUsePagedPagination`
* `toUseOffsetPagination`
* `toUseCursorPagination`
* `toUseCustomPagination`
* `toUseRequestPagination`
* `toBePaginatable`
* `toUseAsyncPagination`
* `toMapPaginatedResponseItems`

#### **Plugin**

* `toBeSaloonPlugin`

#### **Properties**

* `toSetConnectTimeout`
* `toSetRequestTimeout`
* `toBeTriedAgainOnFailure`
* `toHaveRetryInterval`
* `toUseExponentialBackoff`
* `toThrowOnMaxTries`
* `toHaveDefaultDelay`
* `toHaveCustomRetryHandling`

#### **Rate Limit**

* `toHaveRateLimits`

#### **Request**

* `toBeSaloonRequest`
* `toHaveRequestMethod`
* `toSendGetRequest`
* `toSendPostRequest`
* `toSendHeadRequest`
* `toSendPutRequest`
* `toSendPatchRequest`
* `toSendDeleteRequest`
* `toSendOptionsRequest`
* `toSendConnectRequest`
* `toSendTraceRequest`
* `toSendQueryRequest`
* `toHaveJsonBody`
* `toHaveMultipartBody`
* `toHaveXmlBody`
* `toHaveFormBody`
* `toHaveStringBody`
* `toHaveStreamBody`
* `toHaveDefaultQuery`
* `toHaveDefaultBody`
* `toBeSoloRequest`
* `toHaveEndpoint`
* `toCreateDtoFromResponse`

#### **Response**

* `toBeSaloonResponse`

#### **DTO**

* `toBeSaloonDto`

#### **Middleware**

* `toBeRequestMiddleware`
* `toBeResponseMiddleware`

#### **Trait**

* `toUseAcceptsJsonTrait`
* `toUseAlwaysThrowOnErrorsTrait`
* `toUseTimeoutTrait`
* `toUseAuthorisationCodeGrantTrait`
* `toUseClientCredentialsGrantTrait`
* `toUseClientCredentialsBasicAuthGrantTrait`
* `toUseRequiresAuthTrait`
* `toUseApiVersionTrait`

### Contributing

Contributions to Lawman are welcome! If you can think of an Expectation that would be useful for others to utilise in their tests, please submit a PR on GitHub. Please see [CONTRIBUTING.md](https://github.com/JonPurvis/lawman/blob/master/CONTRIBUTING.md). The only real rules are:

* All Expectations must have a test to cover it
* A new Fixture class should be added for every new Expectation

If you just follow how previous Expectations are done, then you should be good.

To report a security vulnerability, please see [SECURITY.md](https://github.com/JonPurvis/lawman/blob/master/SECURITY.md).

#### Reporting an Issue

If you do spot an issue with Lawman, please open an issue on GitHub: [JonPurvis/lawman](https://github.com/JonPurvis/lawman).
