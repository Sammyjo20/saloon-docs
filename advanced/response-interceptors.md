# Response Interceptors

Saloon already allows you to add functionality to your requests in the form of plugins, but if you would like to intercept the response before it is passed back to you, you can add a response interceptor. These can be added into Saloon plugins, or they can be added to the `boot` method on the Connector/Request.

### Example Response Interceptor

This response interceptor will tell all responses to "throw" an exception if it fails. Response interceptors provide you with the **SaloonRequest** and the request and the **SaloonResponse**.&#x20;

```php
<?php

class CreateForgeServerRequest extends SaloonRequest
{
    //...

    public function boot(): void
    {
        $this->addResponseInterceptor(function (SaloonRequest $request, SaloonResponse $response) {
            $response->throw();
    
            return $response;
        });
    }
}
```

{% hint style="info" %}
This interceptor example is a plugin pre-built into Saloon, have a look at the [**AlwaysThrowsOnErrors** plugin](../the-basics/plugins.md#available-plugins)
{% endhint %}

### Macroable

Saloon Responses are also "Macroable" which means you can add your own methods to them to use later.&#x20;

```php
<?php

class CreateForgeServerRequest extends SaloonRequest
{
    //...

    public function boot(): void
    {
        $this->addResponseInterceptor(function (SaloonRequest $request, SaloonResponse $response) {
            $response::macro('hello', function ($name) {
                return 'Hello ' . $name;
            });
    
            return $response;
        });
    }
}
```

After we have defined the macro, we can use it like this.

```php
$response->hello('Sam'); // Returns "Hello Sam" 
```

### Custom Responses&#x20;

If you are looking to overwrite Saloon's response methods or if you would like to add lots of your own methods, consider creating a custom response. [Read more](../the-basics/responses/#custom-responses).
