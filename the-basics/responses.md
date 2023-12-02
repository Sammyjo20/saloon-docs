# 📡 Responses

After sending a request, Saloon will return a `Response` class. This response class contains many helpful methods for interacting with your HTTP response like seeing the HTTP status code and retrieving the body.

```php
<?php

$forge = new ForgeConnector;
$request = new GetServersRequest;

$response = $forge->send($request);

$status = $response->status();
$body = $response->body();
```

{% hint style="info" %}
By default, Saloon will not throw an exception if a synchronous request fails. [Refer to the handling failures section for handling errors.](handling-failures.md)
{% endhint %}

### Useful Methods

Below are some of the key methods provided by Saloon's `Response` class.

<table><thead><tr><th width="342">Method</th><th>Description</th></tr></thead><tbody><tr><td><code>status</code></td><td>Returns the response status code.</td></tr><tr><td><code>headers</code></td><td>Returns all response headers</td></tr><tr><td><code>header</code></td><td>Returns a given header</td></tr><tr><td><code>body</code></td><td>Returns the raw response body as a string</td></tr><tr><td><code>json</code></td><td>Retrieves a JSON response body and json_decodes it into an array.</td></tr><tr><td><code>array</code></td><td>Alias of <code>json</code></td></tr><tr><td><code>collect</code></td><td>Retrieves a JSON response body and json_decodes it into a Laravel Collection. Requires <code>illuminate/collections</code>.</td></tr><tr><td><code>object</code></td><td>Retrieves a JSON response body and json_decodes it into an object.</td></tr><tr><td><code>xmlReader</code></td><td>Used for XML responses - returns a <a href="https://github.com/saloonphp/xml-wrangler">XML Wrangler</a> reader. Requires <code>saloonphp/xml-wrangler</code>.</td></tr><tr><td><code>dom</code></td><td>Used for HTML responses - returns a <a href="https://symfony.com/doc/current/components/dom_crawler.html">Symfony DOM Crawler</a> instance. Requires <code>symfony/dom-crawler</code>.</td></tr><tr><td><code>stream</code></td><td>Returns the response body as an instance of <code>StreamInterface</code></td></tr><tr><td><code>saveBodyToFile</code></td><td>Allows you to save the raw body to a file or open file resource.</td></tr><tr><td><code>dto</code></td><td>Converts the response into a data-transfer object. You must define your DTO first, <a href="data-transfer-objects.md">click here to read more.</a></td></tr><tr><td><code>dtoOrFail</code></td><td>Will work just like <code>dto</code> but will throw an exception if the response is considered "failed".</td></tr><tr><td><code>ok</code>, <code>successful</code>, <code>redirect</code>, <code>failed</code>, <code>clientError</code>, <code>serverError</code></td><td>Methods used to determine if a request was successful or not based on status code. The <code>failed</code> method <a href="handling-failures.md#customising-when-saloon-thinks-a-request-has-failed">can be customised</a>.</td></tr><tr><td><code>throw</code></td><td>Will throw an exception if the response is considered "failed".</td></tr><tr><td><code>getPendingRequest</code></td><td>Returns the <code>PendingRequest</code> class that was built up for the request.</td></tr><tr><td><code>getPsrRequest</code></td><td>Returns the PSR-7 request that was built up by Saloon</td></tr><tr><td><code>getPsrResponse</code></td><td>Return the PSR-7 response that was built up by the HTTP client/sender.</td></tr></tbody></table>

### Asynchronous Responses

When  `sendAsync` or concurrent requests, Saloon will respond with a `GuzzleHttp\Promise\PromiseInterface.` The promise will contain a `Response` a class described above. When the request fails, Saloon will not use the `then` method but return an instance of `RequestException`in the `otherwise` block.

```php
<?php

use Saloon\Http\Response;

$forge = new ForgeConnector;
$promise = $forge->sendAsync(new GetServersRequest);

$promise
    ->then(function (Response $response) {
        // Handle successful response
    })
    ->otherwise(function (Exception $exception) {
        // Handle failed request
    });
```

### Custom Responses

Sometimes you may want to use your response class. This is useful if you want to add your methods or overwrite Saloon's response methods. Saloon allows you to overwrite the response at a connector level for all requests or at a per-request level for a granular response.

The simplest way of registering a custom response is to use the `$response` property on either the connector or request.

{% tabs %}
{% tab title="Connector" %}
<pre class="language-php"><code class="lang-php">class ForgeConnector extends Connector
{
    // ...
    
<strong>    protected ?string $response = CustomResponse::class;
</strong>}
</code></pre>
{% endtab %}

{% tab title="Request" %}
<pre class="language-php"><code class="lang-php">class GetServersRequest extends Request
{
    // ...
    
<strong>    protected ?string $response = CustomResponse::class;
</strong>}
</code></pre>
{% endtab %}
{% endtabs %}

When you need a more advanced way to define a custom response, use the `resolveResponseClass` method on either the connector or request.

{% tabs %}
{% tab title="Connector" %}
```php
class ForgeConnector extends Connector
{
    // ...
    
    public function resolveResponseClass(): string
    {
        return CustomResponse::class;
    }
}
```
{% endtab %}

{% tab title="Request" %}
```php
class GetServersRequest extends Request
{
    // ...
    
    public function resolveResponseClass(): string
    {
        return CustomResponse::class;
    }
}
```
{% endtab %}
{% endtabs %}
