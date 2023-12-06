# XML Body

To get started, change your method to **POST, PUT or PATCH** depending on the requirements of the API. After that, you will need to add the `HasBody` interface to your request. This interface is required as it tells Saloon to look for a `body()` method supplied by one of the body traits. Without this interface, Saloon will not send any request body to the HTTP client.

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
        protected readonly string $ubuntuVersion,
        protected readonly string $type,
        protected readonly string $provider
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
While you are expected to return a raw string for the XML body, it can be tedious to write XML as text. We have built a library, XML Wrangler which allows you to write XML in a much more developer-friendly way. See [below](xml-body.md#using-spaties-array-to-xml-package) for an example using the package.
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

### Writing XML With XML Wrangler

Writing out XML body as text can be laborious and time-consuming. We have built our own XML writer which makes writing XML easier. This library is called [XML Wrangler](https://github.com/saloonphp/xml-wrangler). You can install it via Composer.

```bash
composer require saloonphp/xml-wrangler
```

Once installed, you can use the `XmlWriter` class to write XML. You can use a simple array structure to define XML elements.

{% tabs %}
{% tab title="Default Body" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\XmlWrangler\XmlWriter;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasXmlBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasXmlBody;

    protected Method $method = Method::POST;
    
    public function __construct(
        protected readonly string $ubuntuVersion,
        protected readonly string $type,
        protected readonly string $provider
    ){}
    
    protected function defaultBody(): string
    {
        return XmlWriter::make()->write('root', [
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

use Saloon\XmlWrangler\XmlWriter;

$request = new CreateServerRequest;

$request->body()->set(XmlWriter::make()->write('root', [
   'ubuntu-version' => '22.04',
   'type' => 'web',
   'provider' => 'aws',
]));
```
{% endtab %}
{% endtabs %}

You can learn more and read the documentation for XML Wrangler by [clicking here](https://github.com/saloonphp/xml-wrangler).

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
