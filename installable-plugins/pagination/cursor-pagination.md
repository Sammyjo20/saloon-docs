# Cursor Pagination

First, you will want to import the `CursorPaginator` class and return a new anonymous class that extends Saloon's `CursorPaginator`. This class expects the connector and request to be passed in via the constructor arguments.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Response;
use Saloon\Http\Connector;
<strong>use Saloon\PaginationPlugin\CursorPaginator;
</strong>use Saloon\PaginationPlugin\Contracts\HasPagination;

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
    public function paginate(Request $request): CursorPaginator
    {
<strong>        return new class(connector: $this, request: $request) extends CursorPaginator
</strong>        {
            //
        };
    }
}
</code></pre>

{% hint style="info" %}
You don't need to use an anonymous class if it doesn't fit your code style. It is recommended to reduce the number of classes, but you can create your own pagination class that extends the base paginator if you prefer.
{% endhint %}

After you have defined your paginator class, you will be required to define three protected methods which are used to power the paginator. These methods are:

* **getNextCursor** - This method is used to find the next cursor from the response. This method must return either an integer or a string. The next cursor is used in the next request to get a different page.
* **isLastPage** - This method is used to tell the paginator when to stop processing. Here you can use the response class provided to determine if you are on the last page. Some APIs may provide metadata like remaining results or next page URLs which you can use to check if you are on the last page. Additionally, Saloon has a few properties that can be used to determine if you are on the last page.
* **getPageItems** - This method is used to return the array of results inside of each page. This is used when using the `items` or `collect` method on your paginator class.

Let's implement these three methods on our paginator and dive into how it works.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Response;
use Saloon\Http\Connector;
use Saloon\PaginationPlugin\CursorPaginator;
use Saloon\PaginationPlugin\Contracts\HasPagination;

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
    public function paginate(Request $request): CursorPaginator
    {
        return new class(connector: $this, request: $request) extends CursorPaginator
        {
<strong>            protected function getNextCursor(Response $response): int|string
</strong>            {
                return $response->json('next_cursor');
            }
        
<strong>            protected function isLastPage(Response $response): bool
</strong>            {
                return is_null($response->json('next_cursor'));
            }
            
<strong>            protected function getPageItems(Response $response, Request $request): array
</strong>            {
                return $response->json('items');
            }
        };
    }
}
</code></pre>

Let's assume in this hypothetical example that the API provides a `next_cursor` property in the JSON response. We'll return the value of this property in the `getNextCursor` method. After that, we need to tell Saloon when to stop paginating. We'll use the same `next_cursor` property again. If it's null, we know that there isn't another page to look up. There are various other ways of knowing if you are on the last page, so it's best to fully understand your API's pagination.

To get the page items, we'll use the `json` method on the response to access the `items` array from the body.

### Specifying a default per-page on the paginator

The third-party API you are integrating with may require you to define a page size (per page) on your requests, or you may want to set a default page size used for every request. You can set this default page size as a property on the paginator class.&#x20;

<pre class="language-php"><code class="lang-php">public function paginate(Request $request): CursorPaginator
{
    return new class(connector: $this, request: $request) extends CursorPaginator
    {
<strong>        protected ?int $perPageLimit = 100;
</strong>        
        // ...
    }
}
</code></pre>

You may also use the `setPerPageLimit` method on an instantiated paginator if you want to configure it on the fly. **You must set this before iterating over the paginator.**&#x20;

```php
$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate($request);

$paginator->setPerPageLimit(250);

// foreach($paginator as $response) { ... }
```

### **Assumptions made with the CursorPaginator**

The CursorPaginator will apply pagination by sending two **query parameters**:

* cursor
* per\_page

This assumption might not be the way your third-party API works. You can really easily change this by extending the `applyPagination` method. For example, let's say our API expects a "pageCursor" and a "pageSize" instead. We can simply extend the `applyPagination` method and change the keys. You can apply the pagination in whichever the API requires.

<pre class="language-php"><code class="lang-php">public function paginate(Request $request): CursorPaginator
{
    return new class(connector: $this, request: $request) extends CursorPaginator
    {
        protected function getNextCursor(Response $response): int|string
        {
            return $response->json('next_cursor');
        }
    
        protected function isLastPage(Response $response): bool
        {
            return is_null($response->json('next_cursor'));
        }
        
        protected function getPageItems(Response $response, Request $request): array
        {
            return $response->json('items');
        }
        
        protected function applyPagination(Request $request): Request
        {
            if ($this->currentResponse instanceof Response) {
<strong>                $request->query()->add('pageCursor', $this->getNextCursor($this->currentResponse));
</strong>            }
    
            if (isset($this->perPageLimit)) {
<strong>                $request->query()->add('pageSize', $this->perPageLimit);
</strong>            }
    
            return $request;
        }
    };
}
</code></pre>

### **Useful Properties On The CursorPaginator**

We previously mentioned that the CursorPaginator class provides a few methods that can be used to help with last-page calculation. These methods are:

* **totalResults** - An integer which will return the total number of items returned. This can be used to check if it's equal to the number of total results in the list.
* **page** - An integer which will return the current page that the paginator is currently on.

### Next Steps

After configuring your paginator, head back to the [Using The Paginator](./#using-the-paginator) section of the documentation.
