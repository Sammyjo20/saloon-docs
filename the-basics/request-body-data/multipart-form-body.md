# Multipart Form Body

Multipart body (multipart/form-data) is often used in modern APIs as a way to upload a mixture of files and data. Saloon makes handling multipart bodies easy by providing you with a standardised way of interacting with the values to be sent. You don't have to worry about calculating boundaries or properly encoding the multipart values, you can just use simple arrays and a value object to define the values.

To get started, change your method to **POST, PUT or PATCH** depending on the requirements of the API. After that, you will need to add the `HasBody` interface to your request. This interface is required as it tells Saloon to look for a `body()` method supplied by one of the body traits. Without this interface, Saloon will not send any request body to the HTTP client.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;

<strong>class UploadProfilePictureRequest extends Request implements HasBody
</strong>{
    protected Method $method = Method::POST;
}
</code></pre>

Next, you will need to add the `HasMultipartBody` trait to your request. This trait will implement the `body()` method that the `HasBody` interface requires. It also provides a method `defaultBody()` which you can extend to provide a default body on your request.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
<strong>use Saloon\Traits\Body\HasMultipartBody;
</strong>
class UploadProfilePictureRequest extends Request implements HasBody
{
<strong>    use HasMultipartBody;
</strong>
    protected Method $method = Method::POST;
}
</code></pre>

{% hint style="info" %}
Saloon will automatically send the `Content-Type: multipart/form-data` header for you when using the `HasMultipartBody` trait, however, if you would like to overwrite this behaviour then you can use the `defaultHeaders` method on the request or modify the headers before the request is sent.
{% endhint %}

### The MultipartValue

To build up multipart data, Saloon uses a class `Saloon\Data\MultipartValue`. This class is used as a value object and when your request is sent, it will automatically be converted into a stream to be sent to the API. To see how to use this class, continue reading below.

```php
<?php

use Saloon\Data\MultipartValue;

new MultipartValue(
    name: 'picture', // Required: the name of the multipart value
    value: 'file-path-or-stream', // Required: Absolute path or file stream
    filename: 'profile.png', // Optional: File name
    headers: [], // Optional: Headers to be sent with the individual value
)
```

### Default Body

There are a couple of ways to interact with the request body to prepare it to be sent. You can either use the methods mentioned below to add to the body on any given instance or you can use the `defaultBody` method on your request. This is recommended because you could then define any requirements as constructor arguments in your request and then standardise your request even more. In this example, we are uploading a profile picture to a server so we will expect a file path through our constructor - however, Saloon can support file paths or streams.

```php
<?php

use Saloon\Http\Request;
use Saloon\Data\MultipartValue;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasMultipartBody;

class UploadProfilePictureRequest extends Request implements HasBody
{
    use HasMultipartBody;

    protected Method $method = Method::POST;
    
    public function __construct(
        protected string $filePath,
    ){}
    
    protected function defaultBody(): array
    {
        return [
            new MultipartValue(name: 'picture', value: $this->filePath)
        ];
    }
}
```

### Interacting with the body() method

While you can define the default body on your request, it might be useful to add or modify the body at runtime on a per-request basis. Saloon has the following methods to allow you to modify the multipart request body:

* add(string $name, mixed $value, string $filename = null, array $headers = \[]) -> **Add a multipart value to the  multipart body**
* attach(MultipartValue $value) -> **Attach a multipart directly to the  multipart body**
* remove(string $key) -> **Remove an item from the multipart body**
* merge(... $arrays) -> **Merge another array of multipart values into the multipart body**
* set(array $value) ->  **Overwrite the entire multipart body with a different set of values**
* all() -> **Get the array of multipart values**
* get(string $key) **-> Get an individual multipart value**
* isEmpty() **-> Check if the multipart body is empty**
* isNotEmpty() **-> Check if the multipart body is not empty**

```php
<?php

$request = new UploadProfilePictureRequest;

$request->body()->add(
    name: 'picture',
    contents: 'your-file-contents-or-stream', 
    filename: 'picture.png', // Optional file name
    headers: [
       // Optional custom headers
    ]
);

// You may also get a multipart value by name.

$request->body()->get('picture'); // MultipartValue class

```

{% hint style="info" %}
The `contents` of the `MultipartValue` class can be either the raw-text contents of the file or a PHP stream/resource.
{% endhint %}

### Connector Body

If you would like to also have multipart body on your connector, you can add the same interface and trait to your connector. If you have the trait on both the connector and the request, the properties will be merged. This is useful if you want to have a shared value across every request.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Data\MultipartValue;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasMultipartValue;

class ForgeConnector extends Connector implements HasBody
{
    use HasMultipartValue;

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
