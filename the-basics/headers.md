# 🤓 Headers

Headers can be added by using the `headers()` method on either the connector or the request. When you add headers to a connector instance, every request sent through that connector will merge those headers with the request. When you add headers to a request instance, it will just be added to that one request instance.

### Default Headers

You may configure default headers on the connector or request using the protected `defaultHeaders` method.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    // {...}
    
    protected function defaultHeaders(): array
    {
        return [
            'Content-Type' => 'application/json'
        ];
    }
}
```

{% hint style="info" %}
Default headers on a connector will be applied to every request. This is handy for providing specific headers like Content-Type or Accept.
{% endhint %}
{% endtab %}

{% tab title="Request" %}
```php
<?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
    protected function defaultHeaders(): array
    {
        return [
            'Content-Type' => 'application/json'
        ];
    }
}
```
{% endtab %}
{% endtabs %}

#### Using Properties With Default Headers

You may also use properties in requests to populate headers, for example populating a username custom header.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Http\Request;

class GetServersRequest extends Request
{
    // {...}
    
    protected string $username;
    
    public function __construct(string $username)
    {
        $this->username = $username;
    }
    
    protected function defaultHeaders(): array
    {
        return [
            'X-Username' => $this->username,
        ];
    }
}
```
{% endtab %}

{% tab title="Usage" %}
```php
<?php

$request = new GetServersRequest('Sammyjo20');

// X-Username: Sammyjo20
```
{% endtab %}
{% endtabs %}

### Using the Headers Method

Saloon also offers a handy headers API to manage your headers easily after a request instance has been created. Use the `headers()` method on your request to manage them. Headers added to the request are prioritised more than the connector's headers. This is useful for changing the headers before a request is sent.

```php
<?php

$request = new GetServersRequest();

$request->headers()->add('Content-Type', 'application/json');

$all = $request->headers()->all();

// Content-Type: application/json
```

{% hint style="warning" %}
When you have default headers to a connector, they won't be visible to the request instance as they are merged later in the request lifecycle. Still, request headers will have a higher priority than connector headers.
{% endhint %}

### Available Methods

#### set(array $items)

Overwrite the headers on the request with a new array.

#### merge(...$items)

Merge arrays of headers.

#### remove(string $key)

Remove a given header by its key.

#### get(string $key, mixed $default = null)

Get a given header by its key or return the default.

#### all()

Retrieve all headers as an array.

#### isEmpty

Check if the header object is empty.

{% hint style="info" %}
Click here to view the API reference for this method.
{% endhint %}
