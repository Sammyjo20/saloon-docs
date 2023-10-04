# Limit/Offset Pagination

First, you will want to import the `OffsetPaginator` class and return a new anonymous class that extends Saloon's `OffsetPaginator`. This class expects the connector and request to be passed in via the constructor arguments.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Response;
use Saloon\Http\Connector;
<strong>use Saloon\PaginationPlugin\OffsetPaginator;
</strong>use Saloon\PaginationPlugin\Contracts\HasPagination;

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
    public function paginate(Request $request): OffsetPaginator
    {
<strong>        return new class(connector: $this, request: $request) extends OffsetPaginator
</strong>        {
            //
        };
    }
}
</code></pre>

{% hint style="info" %}
You don't need to use an anonymous class if it doesn't fit your code style. It is recommended to reduce the number of classes, but you can create your own pagination class that extends the base paginator if you prefer.
{% endhint %}

After you have defined your paginator class, you will be required to define a single property and two protected methods which are used to power the paginator. These properties/methods are:

* **perPageLimit** - This property is required as this is the default limit of the paginator. Since this paginator requires limit/offset - the limit is always required. If you need to change the limit on a per-request basis you can change it on the paginator instance.
* **isLastPage** - This method is used to tell the paginator when to stop processing. Here, you can use the response class provided to determine if you are on the last page. Some APIs may provide metadata like remaining results or next-page URLs, which you can use to check if you are on the last page. Additionally, Saloon has a few properties that can be used to determine if you are on the last page.
* **getPageItems** - This method is used to return the array of results inside of each page. This is used when using the `items` or `collect` method on your paginator class.

Let's implement the property and the two methods on our paginator and dive into how it works.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Response;
use Saloon\Http\Connector;
use Saloon\PaginationPlugin\OffsetPaginator;
use Saloon\PaginationPlugin\Contracts\HasPagination;

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
    public function paginate(Request $request): OffsetPaginator
    {
        return new class(connector: $this, request: $request) extends OffsetPaginator
        {
<strong>            protected ?int $perPageLimit = 100;
</strong>        
<strong>            protected function isLastPage(Response $response): bool
</strong>            {
                return $this->getOffset() >= (int)$response->json('total');
            }
            
<strong>            protected function getPageItems(Response $response, Request $request): array
</strong>            {
                return $response->json('items');
            }
        };
    }
}
</code></pre>

First, we'll define a `perPageLimit` of 100. This means that every request will attempt to use a "limit" of 100.&#x20;

Next, let's assume in this hypothetical example that the API provides some useful information that helps us calculate if we're on the last page. For example, our API returns a `total` integer with every page - this number is simply the total results across all pages. We can use this to compare it with the current offset, and if the offset is greater than or equal to the total, we will stop. There are other ways of knowing if you are on the last page, so it's best to understand your API's pagination fully.

To get the page items, we'll use the `json` method on the response to access the `items` array from the body.

### Changing the per-page limit on the paginator

As mentioned previously, you must specify a "per-page limit" on your offset paginator. You may also use the `setPerPageLimit` method on an instantiated paginator if you want to change it on the fly. **You must set this before iterating over the paginator.**&#x20;

```php
$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate($request);

$paginator->setPerPageLimit(250);

// foreach($paginator as $response) { ... }
```

### **Assumptions made with the OffsetPaginator**

The OffsetPaginator will apply pagination by sending two **query parameters**:

* limit
* offset

This assumption might not be the way your third-party API works. You can really easily change this by extending the `applyPagination` method. For example, let's say our API expects the "top" and a "skip" query parameters instead. We can simply extend the `applyPagination` method and change the keys. You can apply the pagination in whichever the API requires.

<pre class="language-php"><code class="lang-php">public function paginate(Request $request): OffsetPaginator
{
    return new class(connector: $this, request: $request) extends OffsetPaginator
    {
        protected ?int $perPageLimit = 100;
    
        protected function isLastPage(Response $response): bool
        {
            return $this->getOffset() >= (int)$response->json('total');
        }
        
        protected function getPageItems(Response $response, Request $request): array
        {
            return $response->json('items');
        }
        
<strong>        protected function applyPagination(Request $request): Request
</strong>        {
            $request->query()->merge([
<strong>                'top' => $this->perPageLimit,
</strong><strong>                'skip' => $this->getOffset(),
</strong>            ]);
    
            return $request;
        }
    };
}
</code></pre>

{% hint style="info" %}
The `getOffset()` method is a useful helper that calculates the current offset for you based on current interation and the per-page limit.
{% endhint %}

### **Useful Methods/Properties On The OffsetPaginator**

We previously mentioned that the `OffsetPaginator` class provides a few methods that can be used to help with last-page calculation. These methods are:

* **totalResults** - An integer which will return the total number of items returned. This can be used to check if it's equal to the number of total results in the list.
* **page** - An integer which will return the current page that the paginator is currently on.
* **getOffset()** - A helpful method that calculates the current offset for you based on current interation and the per-page limit.

### Next Steps

After configuring your paginator, head back to the [Using The Paginator](./#using-the-paginator) section of the documentation.
