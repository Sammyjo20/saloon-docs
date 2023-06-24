# XML Body

To get started, make change your method to **POST, PUT or PATCH** depending on the requirements of the API. After that, you will need to add the `HasBody` interface to your request. This interface is required as it tells Saloon to look for a `body()` method supplied by one of the body traits. Without this interface, Saloon will not send any request body to the HTTP client.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;

<strong>class CreateServerRequest extends Request implements HasBody
</strong>{
    protected Method $method = Method::POST;
}
</code></pre>

Next, you will need to add the `HasXmlBody` trait to your request. This trait will implement the `body()` method that the `HasBody` interface requires. It also provides a method `defaultBody()` which you can extend to provide a default body on your request.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
<strong>use Saloon\Traits\Body\HasXmlBody;
</strong>
class CreateServerRequest extends Request implements HasBody
{
<strong>    use HasXmlBody;
</strong>
    protected Method $method = Method::POST;
}
</code></pre>

{% hint style="info" %}
Saloon will automatically send the `Content-Type: application/xml` header for you when using the `HasXmlBody` trait, however, if you would like to overwrite this behaviour then you can use the `defaultHeaders` method on the request or modify the headers before the request is sent.
{% endhint %}

### Default Body

There are a couple of ways to interact with the request body to prepare it to be sent. You can either use the methods mentioned below to add to the body on any given instance or you can use the `defaultBody` method on your request. This is recommended because you could then define any requirements as constructor arguments in your request and then standardise your request even more.&#x20;

```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasXmlBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasXmlBody;

    protected Method $method = Method::POST;
    
    public function __construct(
        protected string $ubuntuVersion,
        protected string $type,
        protected string $provider
    ){}
    
    protected function defaultBody(): string
    {
        return '
            <?xml version="1.0"?>
            <root>
                <ubuntu-version>' . $this->ubuntuVersion . '</ubuntu-version>
                <type>' . $this->type . '</type>
                <provider>' . $this->provider . '</provider>
            </root>
        ';
    }
}
```

{% hint style="info" %}
While you are expected to return a raw string for the XML body, it can be tedious writing XML as text. Saloon strongly recommends using [Spatie's "Array to XML"](https://github.com/spatie/array-to-xml) package which helps you convert a simple array into XML which is much more developer friendly. See [below](xml-body.md#using-spaties-array-to-xml-package) for an example using the package.
{% endhint %}

### Interacting with the body() method

While you can define the default body on your request, it might be useful to add or modify the body at runtime on a per-request basis. Saloon has the following methods to allow you to modify the XML request body:

* set(string $value) -> **Overwrite the XML body entirely**
* all(): array -> **Get the XML body**&#x20;
* isEmpty(): bool  -> **Check if the body is empty**
* isNotEmpty(): bool -> **Check if the body is not empty**

```php
<?php

$request = new CreateServerRequest;

$request->body()->set('<?xml version="1.0"?><root></root>');

$body = $request->body()->all();

// string: 'plain-text-response-body'

```

### Using Spatie's Array To XML Package

Writing out XML body can be laborious and time-consuming. In some previous integrations, we have leaned on a library to help write out the XML request bodies. This library is [Spatie's "Array to XML"](https://github.com/spatie/array-to-xml) package. You can use this to provide a simple array of keys and values to build up XML data and then pass it through to Saloon.

{% tabs %}
{% tab title="Default Body" %}
```php
<?php

use Saloon\Http\Request;
use Spatie\ArrayToXml\ArrayToXml;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasXmlBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasXmlBody;

    protected Method $method = Method::POST;
    
    public function __construct(
        protected string $ubuntuVersion,
        protected string $type,
        protected string $provider
    ){}
    
    protected function defaultBody(): string
    {
        return ArrayToXml::convert([
            'ubuntu-version' => $this->ubuntuVersion,
            'type' => $this->type,
            'provider' => $this->provider,
        ]);
    }
}
```
{% endtab %}

{% tab title="Using Body Methods" %}
```php
<?php

$request = new CreateServerRequest;

$request->body()->set(ArrayToXml::convert([
   'ubuntu-version' => '22.04',
   'type' => 'web',
   'provider' => 'aws',
]));
```
{% endtab %}
{% endtabs %}

### Connector Body

You can add the same interface and trait to your connector. If you have the trait on both the connector and the request, the body on the request will take priority. If you have the body on just the connector but not on the request, the request will inherit the body on the connector.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasXmlBody;

class ForgeConnector extends Connector implements HasBody
{
    use HasXmlBody;

    protected function defaultBody(): string
    {
        return '<?xml version="1.0"?><root></root>';
    }
}
```
