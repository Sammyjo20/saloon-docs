# Connectors

Saloon Connectors are classes that define the basic requirements of an API. Within a connector, you provide the Base URL of the API, default headers, default config and add reusable Saloon Plugins. You should create a separate connector for each API integration.

{% hint style="info" %}
If you are using Laravel, you can use the **php artisan saloon:connector** Artisan command to create a connector.
{% endhint %}

### Example Connector

Here is an example of a Saloon Connector. I have created one for the popular Laravel Forge service.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class ForgeConnector extends SaloonConnector
{
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
}
```

### Example with headers and config&#x20;

Connectors can also define default headers and default Guzzle options.

```php
class ForgeConnector extends SaloonConnector
{
    // ...
    
    public function defaultHeaders(): array
    {
        return [
            'Authorization' => 'Bearer ' . config('services.forge.key') // "config" is a built in Laravel function.
        ];
    }
    
    public function defaultConfig(): array
    {
        return [
            'timeout' => 5,
        ];
    }
}
```

{% hint style="info" %}
Saloon provides many pre-written plugins to save you from manually defining headers and config variables for common use cases. [Read more about plugins here](plugins.md). Alternatively, to see the full list of Guzzle config options, [click here](https://docs.guzzlephp.org/en/stable/request-options.html).
{% endhint %}

### Available Methods

#### defineBaseUrl()

_Required_

This method should return the base URL of the API. This is typically the hostname of the API and any suffix like `/api/v1`.

#### defaultHeaders()

If defined, this method should return an array of the headers that should be used on every request the connector makes.

#### defaultConfig()

Similar to the `defaultHeaders()` method, if defined - this should return an array of any Guzzle configuration options you would like the connector to use in all of its requests.

#### defaultQuery()

If provided, this method should return an array of the query parameters for the connector to use in all of its requests.

#### defaultData()

If provided, this method should return an array of the default form data for the connector to use in all of its requests.

{% hint style="warning" %}
Requires either the **hasJsonBody, HasFormParams** or **HasMultipartBody** traits. [Read more](requests/attaching-data.md). If you are sending XML data, [click here](requests/attaching-data.md#sending-xml).
{% endhint %}

#### boot($request)

If provided, this method can be used to add extra functionality to the connector. You can use methods like `addHeader(), addConfig(), addHandler() and addResponseInterceptor()`. It is also a useful place to add conditional headers/configuration.

### Constructors

If your connector has a constructor inside with specific data, you should consider sending requests through the connector rather than the request. [Click here to read more.](sending-requests/#sending-requests-using-your-connector)&#x20;

Alternatively, you can overwrite the connector that a request uses, by using the **setConnector()** method on the request before calling the **send()** method.

```php
<?php

$connector = new ForgeConnector($apiKey);
$request = new GetForgeServerRequest(serverId: '123456');

$request->setConnector($connector);

$request->send();
```
