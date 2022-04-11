# Handlers / Middleware

If you need to modify the underlying Guzzle request/response right before it is sent, you can use handlers. This is an incredibly useful feature in Guzzle to add functionality to connectors or requests. Handlers are also useful for adding community built middleware like caching middleware.

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
