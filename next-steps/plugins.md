# Plugins

Saloon comes with a powerful plugin pattern that allows you to add functionality to your connectors and requests in the form of traits. Saloon has a number of useful plugins that you can make changes to your requests like headers, config or data.

Plugins can be added to a connector to be used on every request or can be added to an individual request.

### Using Plugins

Plugins are really easy to install. They are PHP traits that can be added to either the connector or the request.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use Sammyjo20\Saloon\Traits\Plugins\AcceptsJson;

class ForgeConnector extends SaloonConnector
{
    use AcceptsJson;

    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
}
```

{% hint style="info" %}
&#x20;If they are added to the connector, their logic will be applied to all requests.
{% endhint %}
{% endtab %}

{% tab title="Request" %}
```php
<?php

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Plugins\AcceptsJson;

class GetForgeServerRequest extends SaloonRequest
{
    use AcceptsJson;

    protected ?string $method = Saloon::GET;

    protected ?string $connector = ForgeConnector::class;

    public function defineEndpoint(): string
    {
        return '/servers/' . $this->serverId;
    }
    
    // ...
}
```
{% endtab %}
{% endtabs %}

### Available plugins

#### AcceptsJson

This plugin will add the `Accept: application/json` header to your requests.

#### AlwaysThrowsOnErrors

This plugin will run `$response->throw()` automatically for you so you don't have to.

#### HasTimeout

This plugin allows you to specify your own timeout in the `$requestTimeout` property (in seconds).

#### WithDebugData

This plugin will enable Guzzle debug mode so you can see more information about your requests.

#### DisablesSSLVerification

This disables SSL verification. This is useful for local development when you may not have a signed SSL certificate.

{% hint style="danger" %}
Do not use this on any production API. Disabling SSL verification means hackers could see your request unencrypted and steal sensitive information like API credentials.
{% endhint %}

#### CastsToDto

This allows you to specify a DTO that a request should cast to. [Click here to read more about DTO casting.](../the-basics/responses/data-transfer-objects.md)

### Creating your own plugins

If you would like to add your own plugins, for example, if you want to install a Guzzle middleware package or if you would like to apply a specific header to your requests, you can do this really easily.&#x20;

Firstly, create a trait in your application. Let's create a **WithTransactionID** plugin. Saloon will look for a "boot" function and will execute it during the request lifecycle. The "boot" function should be the name of the class, prefixed with the word "boot".

{% hint style="info" %}
If you are using Laravel, you can use the **php artisan saloon:plugin** Artisan command to create a plugin.
{% endhint %}

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;

trait WithTransactionID
{
    public function bootWithTransactionID(SaloonRequest $request)
    {
        $request->mergeHeaders([
            'X-Transaction-ID' => '123-YeeHaw-456'
        ]);
    }
}
```

### Available Plugin Methods

Plugins will inherit all of the available methods from the connector or request it is added to, including the ability to add [response interceptors](../advanced/response-interceptors.md) and [handlers](../advanced/handlers-middleware.md).

* [Connector Methods](../the-basics/connectors.md#available-methods)
* [Request Methods](../the-basics/requests/#available-methods)
