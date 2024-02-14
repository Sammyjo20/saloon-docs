#  👮‍♂️ Lawman

Lawman is a [PestPHP](https://pestphp.com/) Plugin for SaloonPHP which allows you to easily write 
architecture tests for your API integrations with a focus on them being easy 
to write and easy to read. After all, if SaloonPHP makes our API integrations 
beautiful, the tests for them should be beautiful too, right?

Lawman is a third-party plugin written and maintained by 
[Jon Purvis](https://twitter.com/JonPurvis_) and gives you access to 
Expectations for your Connectors, Requests, Plugins and more, to ensure they 
stick to whichever architecture rules your application has.

One of the great things about Lawman is because it's a Pest plugin, it will 
work with regular PestPHP Expectations, just chain them up and you're good 
to go!

### Installation

You can install the plugin using Composer. If you run the following command, 
it will install it as a dev dependency:

```bash
composer require jonpurvis/lawman --dev
```

Lawman makes use of autoloading with Composer, so once you've installed the 
package, the Expectations should appear as you type!

### Examples

Let's take a look at how you can use Lawman to easily write Arch tests for your
API Integrations.

Let's say we have a Connector class that we want to test, with PestPHP we could
do the following:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toExtend('Saloon\Http\Connector')
    ->toUse('Saloon\Traits\Plugins\AcceptsJson')
    ->toUse('Saloon\Traits\Plugins\AlwaysThrowOnErrors');
```

So that test is ensuring our class extends the base Connector and uses the 
`AcceptJson` and `AlwaysThrowOnErrors` traits. Whilst that test works, we 
could perhaps make it quicker to write and easier to read, so with Lawman, 
you can do:

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

What about if we want to test our Connector has an Authentication method? 
Lawman makes this easy to do, it even works with multi auth:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toBeSaloonConnector()
    ->toUseCertificateAuthentication()
    ->toUseTokenAuthentication();
```

Lawman also has Expectations for the Pagination, Cache and Rate Limit Plugins:

```php
test('request')
    ->expect('App\Http\Integrations\Integration\Requests\Request')
    ->toBeSaloonRequest()
    ->toSendPostRequest()
    ->toUsePagedPagination()
    ->toHaveCaching()
    ->toHaveRateLimits()
```

Maybe our Connector has some Retry instructions that we want to test. Again, 
with Lawman, it's as simple as:

```php
test('connector')
    ->expect('App\Http\Integrations\Integration\Connector')
    ->toBeSaloonConnector()
    ->toBeTriedAgainOnFailure()
    ->toHaveRetryInterval()
    ->toUseExponentialBackoff()
```

### Available Expectations

#### Authentication

- `toUseTokenAuthentication`
- `toUseBasicAuthentication`
- `toUseCertificateAuthentication`
- `toUseHeaderAuthentication`

#### Cache

- `toHaveCaching`

#### Connector

- `toBeSaloonConnector`

#### Pagination

- `toUsePagedPagination`
- `toUseOffsetPagination`
- `toUseCursorPagination`
- `toUseCustomPagination`

#### Properties

- `toSetConnectTimeout`
- `toSetRequestTimeout`
- `toBeTriedAgainOnFailure`
- `toHaveRetryInterval`
- `toUseExponentialBackoff`
- `toThrowOnMaxTries`

#### Rate Limit

- `toHaveRateLimits`

#### Request

- `toBeSaloonRequest`
- `toHaveRequestMethod`
- `toSendGetRequest`
- `toSendPostRequest`
- `toSendHeadRequest`
- `toSendPutRequest`
- `toSendPatchRequest`
- `toSendDeleteRequest`
- `toSendOptionsRequest`
- `toSendConnectRequest`
- `toSendTraceRequest`
- `toHaveJsonBody`
- `toHaveMultipartBody`
- `toHaveXmlBody`
- `toHaveFormBody`
- `toHaveStringBody`
- `toHaveStreamBody`

#### Response

- `toBeSaloonResponse`

#### Trait

- `toUseAcceptsJsonTrait`
- `toUseAlwaysThrowOnErrorsTrait`
- `toUseTimeoutTrait`
- `toUseAuthorisationCodeGrantTrait`
- `toUseClientCredentialsGrantTrait`

### Contributing

Contributions to Lawman are welcome! If you can think of an Expectation that
would be useful for others to utilise in their tests, please submit a PR on 
GitHub. The only real rules are:

- All Expectations must have a test to cover it
- A new Fixture class should be added for every new Expectation

If you just follow how previous Expectations are done, then you should be good.

### Reporting an Issue

If you do spot an issue with Lawman, please 
[open an issue](https://github.com/JonPurvis/lawman/issues) on GitHub. 