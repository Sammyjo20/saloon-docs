# Requests

Saloon Requests are reusable classes that define each request of the API you want to make. They contain the request's method, as well as any default headers, or data that should be used.

{% hint style="info" %}
If you are using Laravel, you can use the **php artisan saloon:request** Artisan command to create a request.
{% endhint %}

### Example Request

Here is an example of a Saloon Request. Now we have our **ForgeConnector**, let's create a request that fetches a server's details from the API. We must specify the method and the connector that is used to make the request. After that, we can specify the endpoint. We can also create a constructor on the class that accepts any data the request needs to be built, like an ID.

```php
<?php

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;

class GetForgeServerRequest extends SaloonRequest
{
    protected ?string $connector = ForgeConnector::class;

    protected ?string $method = Saloon::GET;

    public function defineEndpoint(): string
    {
        return '/servers/' . $this->serverId;
    }
    
    public function __construct(
        public string $serverId
    ){}
}
```

### Example with headers and config

Requests can also have their own headers and config which will be merged in with the connector's default headers and config. However, the request can overwrite the connector's default headers or config.

```php
class GetForgeServerRequest extends SaloonRequest
{
     // ...
     
    public function defaultHeaders(): array
    {
        return [
            'X-Custom-Header' => 'Hello-World',
        ];
    }
    
    public function defaultConfig(): array
    {
        return [
            'allow_redirects' => true,
            // Timeout will overwrite the connector's header.
            'timeout' => 10
        ];
    }
}
```

{% hint style="info" %}
Saloon provides many pre-written plugins to save you from manually defining headers and config variables for common use cases. [Read more about plugins here](../../next-steps/plugins.md). Alternatively, to see the full list of Guzzle config options, [click here](https://docs.guzzlephp.org/en/stable/request-options.html).
{% endhint %}

### Properties

#### $method

_Required_

The HTTP method used in the request. For example GET, POST, PUT, DELETE.

#### $connector

_Required_

The connector the request uses to know about the API.

### Available Methods

#### defineEndpoint()

If defined, this method should return the endpoint of your API request. This is typically the endpoint that points to a resource of the API for example `/servers`. This can be omitted if the API you are using does not have any specific endpoints, like GraphQL APIs.

#### defaultHeaders()

If defined, this method should return an array of the headers that should be used on every request.

#### defaultConfig()

Similar to the `defaultHeaders()` method, if defined - this should return an array of any Guzzle configuration options you would like the request to use.

#### defaultQuery()

If provided, this method should return an array of the query parameters for the request to use.

#### defaultData()

If provided, this method should return an array of the default form data for the request to use.

{% hint style="warning" %}
Requires either the **hasJsonBody, HasFormParams** or **HasMultipartBody** traits. [Read more](attaching-data.md). If you are sending XML data, click here.
{% endhint %}

#### boot($request)

If provided, this method can be used to add extra functionality to the request. You can use methods like `addHeader(), addConfig(), addHandler() and addResponseInterceptor()`. It is also a useful place to add conditional headers/configuration.

#### getConnector

This method will return you the instance of connector, which you can modify in runtime.

#### getFullRequestUrl

This method will return the full request URL including the connector's base URL.

#### \_\_call

Any other methods will be proxied to the connector. This is useful if you need to use a method on the connector like `setApiKey`.

```php
(new GetForgeServerRequest)->connectorMethod()->send();
```
