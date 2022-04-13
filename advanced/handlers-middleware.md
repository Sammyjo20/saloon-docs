# Handlers / Middleware

If you need to modify the underlying Guzzle request/response right before it is sent, you can use handlers. This is an incredibly useful feature that Guzzle provides to view/modify the request before it is sent.

To add a handler or middleware, simply use the `addHandler` method in your plugin or `boot` method on your connector/request.

[Click here to read more about Guzzle Handlers / Middleware](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html)

### Example

```php
use Psr\Http\Message\RequestInterface;

class CreateForgeServerRequest extends SaloonRequest
{
    //...

    public function boot(): void
    {
        $this->addHandler('customHeaderHandler', function (callable $handler) {
            return function (RequestInterface $request, array $options) use ($handler) {
                $request->withHeader('X-Custom-Header', 'Hello');
                
                return $handler($request, $options);             
            };
        });
    }
}
```

{% hint style="info" %}
Saloon will not know about any extra headers, configuration or data you add inside of handlers.
{% endhint %}
