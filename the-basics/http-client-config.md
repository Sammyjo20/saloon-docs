# 🔧 HTTP Client Config

Like headers and query parameters, HTTP config can be added by using the `config()` method on either the connector or the request. When you add config to a connector instance, every request sent through that connector will merge those configuration options with the request. When you add config to a request instance, it will just be added to that one request instance.

### Guzzle Config

Saloon's default sender is the GuzzleSender. By default, the config options are added to the guzzle request. You can use any of the options Guzzle provide in the Saloon's config. [Click here to see a list of the available options Guzzle provide.](https://docs.guzzlephp.org/en/stable/request-options.html)

### Default HTTP Config

You may configure the default HTTP config on the connector or request using the protected `defaultConfig` method.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    // {...}
    
    protected function defaultConfig(): array
    {
        return [
            'timeout' => 30,
        ];
    }
}
```

{% hint style="info" %}
Default HTTP config on a connector will be applied to every request that uses the connector.
{% endhint %}
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
    protected function defaultConfig(): array
    {
        return [
            'timeout' => 30,
        ];
    }
}
```
{% endtab %}
{% endtabs %}

#### Using Properties With Default HTTP Config

You may also use properties in requests to populate HTTP config, for example - specifying an exact timeout on a per-request basis.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
    protected int $timeout;
    
    public function __constructor(int $timeout)
    {
        $this->timeout = $timeout;
    }
    
    protected function defaultConfig(): array
    {
        return [
            'timeout' => $this->timeout,
        ];
    }
}
```
{% endtab %}

{% tab title="Usage" %}
```php
<?php

$request = new GetServersRequest(timeout: 30);
```
{% endtab %}
{% endtabs %}

### Using the config Method

Saloon also offers a handy config API to manage them easily after a request instance has been created. Use the `config()` method on your request to manage them. HTTP config added to the request is prioritised more than the connector's config. This is useful for changing them before a request is sent.

```php
<?php

$request = new GetServersRequest();

$request->config()->add('timeout', 30);

$all = $request->config()->all();

// array: ['timeout' => 30]
```

{% hint style="warning" %}
When you have default HTTP config on a connector, they won't be visible to the request instance as they are merged later in the request lifecycle. Still, request HTTP config will have a higher priority than connector config.
{% endhint %}

### Available Methods

#### set(array $items)

Overwrite the config on the request with a new array.

#### merge(...$items)

Merge arrays of config.

#### remove(string $key)

Remove a given config option by its key.

#### get(string $key, mixed $default = null)

Get a givenconfig option by its key or return the default.

#### all()

Retrieve all config options as an array.

#### isEmpty

Check if the config object is empty.

{% hint style="info" %}
Click here to view the API reference for this method.
{% endhint %}
