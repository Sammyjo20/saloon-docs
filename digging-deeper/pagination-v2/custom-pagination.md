# Custom Pagination

### Introduction

If you are building an integration with a third-party API that does not use paged, limit/offset or cursor pagination, you can build your own paginator. Saloon's base paginator class has been designed to be flexible and easy to build against.

### Getting Started

First, you will want to import the `Paginator` abstract class and return a new anonymous class that extends Saloon's `Paginator`. This class expects the connector and request to be passed in via the constructor arguments.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Response;
use Saloon\Http\Connector;
<strong>use Saloon\PaginationPlugin\Paginator;
</strong>use Saloon\PaginationPlugin\Contracts\HasPagination;

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
    public function paginate(Request $request): Paginator
    {
        return new class(connector: $this, request: $request) extends Paginator
        {
            //
        };
    }
}
</code></pre>

{% hint style="info" %}
You don't need to use an anonymous class if it doesn't fit your code style. It is recommended to reduce the number of classes, but you can create your own pagination class that extends the base paginator if you prefer.
{% endhint %}

After you have defined your paginator class, you will be required to define three protected methods which are used to power the paginator. These methods are:

* **isLastPage** - This method is used to tell the paginator when to stop processing. Here you can use the response class provided to determine if you are on the last page. Some APIs may provide metadata like remaining results or next page URLs which you can use to check if you are on the last page. Additionally, Saloon has a few properties that can be used to determine if you are on the last page.
* **getPageItems** - This method is used to return the array of results inside of each page. This is used when using the `items` or `collect` method on your paginator class.
* **applyPagination** - This method is used to change each request to apply the pagination for the next page.

Let's implement these three methods on our paginator and dive into how it works.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Response;
use Saloon\Http\Connector;
<strong>use Saloon\PaginationPlugin\Paginator;
</strong>use Saloon\PaginationPlugin\Contracts\HasPagination;

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
    public function paginate(Request $request): Paginator
    {
        return new class(connector: $this, request: $request) extends Paginator
        {
            protected function isLastPage(Response $response): bool
            {
<strong>                return is_null($response->json('next_page_url'));
</strong>            }
            
            protected function getPageItems(Response $response, Request $request): array
            {
<strong>                return $response->json('items');
</strong>            }
            
            protected function applyPagination(Request $request): Request
            {
<strong>                $request->header()->add('X-Page', $this->currentPage);
</strong><strong>                
</strong><strong>                $request->header()->add('X-Per-Page', $this->perPageLimit);
</strong>            }
        };
    }
}
</code></pre>

Let's assume in this hypothetical example that the API provides some useful information to see if we're on the last page. In this example, we know we're on the last page if the `next_page_url` JSON property on the body is null. There are various other ways of knowing if you are on the last page, so it's best to fully understand your API's pagination.&#x20;

To get the page items, we'll use the `json` method on the response to access the `items` array from the body.

After that, to apply the pagination, our API expects us to send a `X-Page` header with the current page on the paginator. We'll use the `$this->currentPage` property on the paginator as it acts like an index for the current iteration of the paginator. We will also make sure to send the `perPageLimit` to our API with the `X-Per-Page` header. This header is optional depending on the third-party API you are integrating with.

### Specifying a default per-page on the paginator

The third-party API you are integrating with may require you to define a page size (per page) on your requests, or you may want to set a default page size used for every request. You can set this default page size as a property on the paginator class.&#x20;

<pre class="language-php"><code class="lang-php">public function paginate(Request $request): Paginator
{
    return new class(connector: $this, request: $request) extends Paginator
    {
<strong>        protected ?int $perPageLimit = 100;
</strong>        
        // ...
    };
}
</code></pre>

You may also use the `setPerPageLimit` method on an instantiated paginator if you want to configure it on the fly. **You must set this before iterating over the paginator.**&#x20;

```php
$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate($request);

$paginator->setPerPageLimit(250);

// foreach($paginator as $response) { ... }
```

{% hint style="warning" %}
For this feature to work, you must implement sending the `perPageLimit` inside of your `applyPagination` method in whichever way your API may expect.
{% endhint %}

### Next Steps

After configuring your paginator, head back to the [Using The Paginator](./#using-the-paginator) section of the documentation.
