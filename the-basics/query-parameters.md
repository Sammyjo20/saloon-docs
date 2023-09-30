# ❓ Query Parameters

Like headers, query parameters can be added by using the `query()` method on either the connector or the request. When you add query parameters to a connector instance, every request sent through that connector will merge those parameters with the request. When you add query parameters to a request instance, it will just be added to that one request instance.

### Default Query Parameters

You may configure default query parameters on the connector or request using the protected `defaultQuery` method.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    // {...}
    
    protected function defaultQuery(): array
    {
        return [
            'per_page' => 50,
        ];
    }
}
```

{% hint style="info" %}
Default query parameters on a connector will be applied to every request that uses the connector.
{% endhint %}
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
    protected function defaultQuery(): array
    {
        return [
            'per_page' => 50,
        ];
    }
}
```
{% endtab %}
{% endtabs %}

#### Using Properties With Default Query Parameters

You may also use properties in requests to populate query parameters, for example - requesting a specific page on a paginated API.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
    protected int $perPage;
    
    protected int $page;
    
    public function __construct(int $perPage, int $page)
    {
        $this->perPage = $perPage;
        $this->page = $page;
    }
    
    protected function defaultQuery(): array
    {
        return [
            'per_page' => $this->perPage,
            'page' => $this->page,
        ];
    }
}
```
{% endtab %}

{% tab title="Usage" %}
```php
<?php

$request = new GetServersRequest(perPage: 50, page: 5);
```
{% endtab %}
{% endtabs %}

### Using the query Method

Saloon also offers a handy query parameters API to manage them easily after a request instance has been created. Use the `query()` method on your request to manage them. Query parameters added to the request are prioritised more than the connector's query parameters. This is useful for changing them before a request is sent.

```php
<?php

$request = new GetServersRequest();

$request->query()->add('page', 5);

$all = $request->query()->all();

// array: ['page' => 5]
```

{% hint style="warning" %}
When you have default query parameters on a connector, they won't be visible to the request instance as they are merged later in the request lifecycle. Still, request query parameters will have a higher priority than connector query parameters.
{% endhint %}

### Available Methods

#### set(array $items)

Overwrite the query parameters on the request with a new array.

#### merge(...$items)

Merge arrays of query parameters.

#### remove(string $key)

Remove a given query parameter by its key.

#### get(string $key, mixed $default = null)

Get a given query parameter by its key or return the default.

#### all()

Retrieve all query parameters as an array.

#### isEmpty

Check if the query parameter object is empty.

{% hint style="info" %}
Click here to view the API reference for this method.
{% endhint %}
