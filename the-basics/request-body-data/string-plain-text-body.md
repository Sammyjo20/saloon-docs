# String / Plain Text Body

This body type has been created if you need to send a plain string/text to the server. This body type does not come with a default Content-Type, so you must provide this.

To get started, change your method to **POST, PUT or PATCH** depending on the requirements of the API. After that, you will need to add the `HasBody` interface to your request. Without this interface, Saloon will not send any request body to the HTTP client.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;

<strong>class CreateServerRequest extends Request implements HasBody
</strong>{
    protected Method $method = Method::POST;
}
</code></pre>

Next, you will need to add the `HasStringBody` trait to your request. This trait will implement the `body()` method that the `HasBody` interface requires. It also provides a method `defaultBody()` which you can extend to provide a default body on your request.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
<strong>use Saloon\Traits\Body\HasStringBody;
</strong>
class CreateServerRequest extends Request implements HasBody
{
<strong>    use HasStringBody;
</strong>
    protected Method $method = Method::POST;
    
    // Define our own Content-Type...
    
    protected function defaultHeaders(): array
    {
        return [
            'Content-Type' => 'text/plain',
        ];
    }
}
</code></pre>

{% hint style="info" %}
Saloon won't add a Content-Type header for you for plain string bodies so you must provide your own Content-Type header.
{% endhint %}

### Default Body

There are a couple of ways to interact with the request body to prepare it to be sent. You can either use the methods mentioned below to add to the body on any given instance or you can use the `defaultBody` method on your request. This is recommended because you could then define any requirements as constructor arguments in your request and then standardise your request even more.&#x20;

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
<strong>use Saloon\Traits\Body\HasStringBody;
</strong>
class CreateServerRequest extends Request implements HasBody
{
<strong>    use HasStringBody;
</strong>
    protected Method $method = Method::POST;
    
    public function __construct(
        protected string $ubuntuVersion,
        protected string $provider
    ){}
    
    protected function defaultBody(): string
    {
        return 'Howdy, Partner. I want a ' . $this->ubuntuVersion . ' server through ' . $this->provider . ' provider!';
    }
}
</code></pre>

### Interacting with the body() method

While you can define the default body on your request, it might be useful to add or modify the body at runtime on a per-request basis. Saloon has the following methods to allow you to modify the string request body:

* set(string $value) -> **Overwrite the string body entirely**
* all(): array -> **Get the string body**&#x20;
* isEmpty(): bool  -> **Check if the body is empty**
* isNotEmpty(): bool -> **Check if the body is not empty**

```php
<?php

$request = new CreateServerRequest;

$request->body()->set('Howdy, Partner');

$body = $request->body()->all();

// string: 'plain-text-response-body'
```

### Connector Body

You can add the same interface and trait to your connector. If you have the trait on both the connector and the request, the body on the request will take priority. If you have the body on just the connector but not on the request, the request will inherit the body on the connector.

```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasStringBody;

class ForgeConnector extends Connector implements HasBody
{
    use HasStringBody;

    protected function defaultBody(): string
    {
        return 'Howdy, Partner';
    }
}
```
