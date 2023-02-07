# 📦 Request Body/Data

When sending HTTP requests, a common requirement is to attach a payload/body to POST, PUT, or PATCH requests, like JSON, XML or multipart data. Saloon makes this easy for you with built-in body traits.

### Getting Started

To get started, you will need to add the `HasBody` interface to your request. This interface is required as it tells Saloon to look for a `body()` method supplied by one of the body traits. Without this interface, Saloon will not send any request body to the HTTP client.

```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;

class CreateServerRequest extends Request implements HasBody
{
    //
}
```

Next, you will need to add a trait to provide an implementation for the missing `body()` method. Saloon has a trait for all the common types of request bodies.

* HasJsonBody - Sending JSON requests (`application/json`)
* HasFormBody - Sending `application/x-www-form-urlencoded` requests
* HasMultipartBody - Sending `multipart/form-data` requests
* HasXmlBody - Sending `application/xml` requests
* HasBody - Sending plain string body, you must specify the content-type.

### HasJsonBody or HasFormBody

These two traits allow you to interact with the request body the same way, but each applies different strategies to encode the data and apply a different Content-Type header.

#### Adding Traits

{% tabs %}
{% tab title="HasJsonBody" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasJsonBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasJsonBody;
}
```
{% endtab %}

{% tab title="HasFormBody" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasFormBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasFormBody;
}
```
{% endtab %}
{% endtabs %}

#### Interacting with the body() method

Once you have added one of the traits, you can access the `body()` method on your request. When using either the `HasJsonBody` or `HasFormBody` traits, you will have methods to add, remove, merge, set and retrieve all the properties in the data.

```php
<?php

$request = new CreateServerRequest;

$request->body()->add('ubuntu_version', '22.04');

$request->body()->merge([
    'type' => 'app',
    'provider' => 'ocean2',
]);

$body = $request->body()->all();

// array: [
//    'ubuntu_version' => '22.04',
//    'type' => 'app',
//    'provider' => 'ocean2',
// ]
```

#### Available Methods

* add(string $key, mixed $value)
* remove(string $key)
* merge(…$values)
* set(array $value)
* all(): array
* isEmpty(): bool
* isNotEmpty(): bool

### Multipart Body

Saloon has also made it easier to send multipart form requests for attaching files. Once you have added the `HasBody` interface, add the `HasMultipartBody` trait. Next, you can use the available methods below to add/remove/get and set the multipart body. You may use the `MultipartValue` object if you want to define default multipart bodies to send.

{% tabs %}
{% tab title="Default Multipart Body Example" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasMultipartBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasMultipartBody;
    
    protected function defaultBody(): array
    {
        return [
            new MultipartValue(name: 'logo', value: 'image-contents'), 
            new MultipartValue(name: 'logo', value: StreamObject), 
            new MultipartValue(name: 'logo', value: 'image-contents', filename: 'logo.png', headers: [...]), 
        ];
    }
} 
```
{% endtab %}

{% tab title="Adding Multipart File Example" %}
```php
<?php

$request = new CreateServerRequest;

$request->body()->add(
    name: 'logo',
    contents: 'your-file-contents-or-stream', 
    filename: 'logo.png', 
    headers: [
       // Optional custom headers
    ]
);

// You may also get a multipart value by name.

$request->body()->get('logo'); // MultipartValue class
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The `contents` of the `MultipartValue` class can be either the raw-text contents of the file or a PHP stream/resource.
{% endhint %}

#### Available Methods

* add(string $key, mixed $value)
* remove(string $key)&#x20;
* merge(…$values)&#x20;
* set(array $value)&#x20;
* all(): array&#x20;
* isEmpty(): bool
* isNotEmpty(): bool

### HasXmlBody or HasBody

These two traits do not encode the body. The `HasXmlBody` trait will apply the `application/xml` Content-Type header but the `HasBody` trait has been designed to be available for just sending raw string text with your Content-Type.

#### Adding Traits

{% tabs %}
{% tab title="HasXmlBody" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Traits\Body\HasXmlBody;
use Saloon\Contracts\Body\HasBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasXmlBody;
}
```
{% endtab %}

{% tab title="HasBody" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Traits\Body\HasBody;
use Saloon\Contracts\Body\HasBody as HasBodyContract;

class CreateServerRequest extends Request implements HasBodyContract
{
    use HasBody;
}
```
{% endtab %}
{% endtabs %}

#### Interacting with the body() method

Once you have added one of the traits, you can access the `body()` method on your request. When using either the `HasXmlBody` or `HasBody` traits, you will have basic methods to set and retrieve the plain string body.

```php
<?php

$request = new CreateServerRequest;

$request->body()->set('plain-text-response-body');

$body = $request->body()->all();

// string: 'plain-text-response-body'
```

#### Available methods

* set(?string $body)
* all(): ?string
* isEmpty(): bool
* isNotEmpty(): bool

### Default Body

Managing the body on the fly to requests is handy, but Saloon is all about standardising and re-using requests. You can also extend a `defaultBody()` method on the request to provide data; you could then accept constructor arguments for the request body. You may even want to accept a DTO through the constructor properties.

{% tabs %}
{% tab title="HasJsonBody, HasFormBody, HasMultipartBody" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasJsonBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasJsonBody;
    
    public function __construct(
        protected string $ubuntuVersion,
        protected string $type,
        protected string $provider
    ){}
    
    protected function defaultBody(): array
    {
        return [
            'ubuntu_version' => $this->ubuntuVersion,
            'type' => $this->type,
            'provider' => $this->provider,
        ];
    }
} 
```
{% endtab %}

{% tab title="HasXmlBody, HasBody" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasBody;
    
    public function __construct(
        protected string $ubuntuVersion,
        protected string $type,
        protected string $provider
    ){}
    
    protected function defaultBody(): string
    {
        return 'Default Raw Body';
    }
} 
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Depending on the type of body you add, the `defaultBody`method will expect an array or string.
{% endhint %}

### Connector Body

You may also use the `HasBody` and a body trait on your connector. When you have a connector with a body trait and a request, Saloon will prefer the request’s body type over the connector’s. Additionally, if you are using the `HasJsonBody`, `HasFormBody`, or `HasMultipartBody` traits on both, the arrays will be merged.

The same method is applied for adding body to a connector, add the interface and a body trait. You may also extend the `defaultBody` method.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasJsonBody;

class ForgeConnector extends Connector implements HasBody
{
    use HasJsonBody;

    protected function defaultBody(): array
    {
        return [
            'name' => 'Sam',
        ];
    }
}
```

### Custom Request Body

While Saloon has covered most of the body types required for day-to-day integrations, you don't have to use one of the traits. You can build your own "BodyRepository" classes and use them in the `body()` method on your connector or request. Just make sure to add the `HasBody` interface as explained above.

```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Contracts\Body\BodyRepository;

class CreateServerRequest extends Request implements HasBody
{
    //
    
    public function body(): BodyRepository
    {
        return new CustomBodyRepository();
    }
}
```
