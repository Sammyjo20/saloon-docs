# 📖 Pagination

### Introduction

Most APIs you will integrate with will implement some sort of pagination for their records. Traditionally, traversing through many pages of results can be time-consuming and slow to build. Saloon helps to solve this problem with its pre-build paginators that can be used to iterate through hundreds of pages of results in a fast, beautiful way.&#x20;

```php
<?php

$connector = new SpotifyConnector;

// Create a paginator and pass in a request class, in this example
// we'll pass in the LikedSongsRequest which will retrieve all
// the liked songs of the authenticated user.

$paginator = $connector->paginate(new LikedSongsRequest);

// Create a Laravel LazyCollection from the paginator and iterate
// over each of the results. Traditionally, the liked songs endpoint
// only lets you get 50 tracks per request, but the paginator will
// automatically grab every page of results and pass it into a 
// single collection! 🔥

$collection = $paginator->collect('items')->map(function ($track) {
    return sprintf('%s - %s', $track['artist'], $track['name']);
});

// Convert the LazyCollection into an array.

$data = $collection->all();
```

Saloon's paginators provide many ways to iterate over responses and even iterate through the results within a response, like the example above. This is incredible because you can retrieve all of the results at once instead of having to write logic to build up an array of results.

Salooon uses a custom-built iterator behind the scenes so memory usage is low and it's incredibly fast to iterate through hundreds of pages, especially when you combine it with asynchronous requests/pooling.

You can see the above example fully in the [Saloon Spotify Example Repository](https://github.com/Sammyjo20/saloon-v2-spotify-example/blob/main/app/Http/Controllers/TracksController.php) &#x20;

{% hint style="warning" %}
Saloon's pagination currently only works with JSON APIs
{% endhint %}

### Supported Pagination Methods

Saloon has support for the three most common types of pagination. You should use the one that is used by the API you are integrating with. You can typically determine this from the query parameters that are provided.

* Paged Pagination (?page=1)
* Limit / Offset Pagination (?limit=100\&offset=0)
* Cursor Pagination (?cursor=string)

### Getting Started

Before you get started, make sure that the request you are going to paginate through works as expected. Make the request like you normally would and read the JSON response output. Take note of the pagination properties like the "next\_page\_url", the "results" count, and the "limit" and "offset" variables. You will need these to configure the paginator. **It's recommended you use a tool like Postman or Saloon in a test to see what the response looks like.**

Once you have recorded an example response, go to your connector and add the `HasPagination` interface. Next, you should define the `paginate` method which accepts a `Request` and additional arguments.

```php
<?php

use Saloon\Contracts\HasPagination;
use Saloon\Contracts\Paginator;
use Saloon\Contracts\Request;

class SpotifyConnector extends Connector implements HasPagination
{
    // {..}

    public function paginate(Request $request, mixed ...$additionalArguments): Paginator;
    {
        //
    }    
}
```

Next, you need to configure a paginator to use. By default, each paginator has its own assumptions for query parameters and response body arguments for the data. You should customise those to suit the API you are integrating with.

{% tabs %}
{% tab title="PagedPaginator" %}
The PagedPaginator requires you to define the request and the per-page/limit. This is required to calculate how many pages the Paginator needs to iterate through. For example, the API may only allow me to paginate over a maximum of 50 results.

```php
<?php

use Saloon\Http\Paginators\PagedPaginator;

public function paginate(Request $request, mixed ...$additionalArguments): PagedPaginator;
{
    return new PagedPaginator($this, $request, 50, ...$additionalArguments);
} 
```

#### Defaults

The `PagedPaginator` has assumed the following items are present in your API:

* The per-page query parameter is called `limit`
* The page query parameter is called `page`
* The next page URL is called `next_page_url` in the response.
* The results count is called `total`
* The first page is zero

#### Configuring the PagedPaginator

To configure the `PagedPaginator` you can use the following methods

```php
<?php

use Saloon\Http\Paginators\PagedPaginator;

public function paginate(Request $request, mixed ...$additionalArguments): PagedPaginator;
{
    $paginator = new PagedPaginator($this, $request, 50, ...$additionalArguments);
    
    $paginator->setLimitKeyName('limit');
    $paginator->setTotalKeyName('count');
    $paginator->setPageKeyName('page');
    
    // You can use "dot" notion for nested data
    
    $paginator->setNextPageKeyName('meta.next');
    $paginator->setCurrentPage(0);
    
    return $paginator;
} 
```
{% endtab %}

{% tab title="OffsetPaginator" %}
The OffsetPaginator requires you to define the request and the limit. This is required to calculate how many pages the Paginator needs to iterate through. For example, the API may only allow me to paginate over a maximum of 50 results.

```php
<?php

use Saloon\Http\Paginators\OffsetPaginator;

public function paginate(Request $request, mixed ...$additionalArguments): OffsetPaginator;
{
    return new OffsetPaginator($this, $request, 50, ...$additionalArguments);
} 
```

#### Defaults

The `OffsetPaginator`has assumed the following items are present in your API:

* The limit query parameter is called `limit`
* The offset query parameter is called `offset`
* The results count is called `total`
* The default offset is zero

#### Configuring the OffsetPaginator

To configure the `OffsetPaginator` you can use the following methods

```php
<?php

use Saloon\Http\Paginators\OffsetPaginator;

public function paginate(Request $request, mixed ...$additionalArguments): OffsetPaginator;
{
    $paginator = new OffsetPaginator($this, $request, 50, ...$additionalArguments);
    
    $paginator->setLimitKeyName('top');
    $paginator->setOffsetKeyName('skip');
    $paginator->setTotalKeyName('count');
    $paginator->setCurrentOffset(1000);
    
    return $paginator;
} 
```
{% endtab %}

{% tab title="CursorPaginator" %}
The CursorPaginator requires you to define the request and the limit. This is required to calculate how many pages the Paginator needs to iterate through. For example, the API may only allow me to paginate over a maximum of 50 results.

```php
<?php

use Saloon\Http\Paginators\CursorPaginator;

public function paginate(Request $request, mixed ...$additionalArguments): CursorPaginator;
{
    return new CursorPaginator($this, $request, 50, ...$additionalArguments);
} 
```

The cursor paginator works by looking for a `next_page_url` parameter in your response body and will attempt to find a `cursor` query parameter. It will then strip out just the `cursor` query parameter and will use that as the cursor string. If your cursor is implemented differently, you should extend the `CursorPaginator` and change the `getCursor` method.

```php
<?php

class CustomPaginator extends CursorPaginator
{
    public function getCursor(): string|int|null
    {
        return $this->currentResponse->json('meta.cursor');
    }
}
```

#### Defaults

The `CursorPaginator`has assumed the following items are present in your API:

* The limit query parameter is called `limit`
* The cursor query parameter is called `cursor`
* The next page URL is called `next_page_url` in the response
* The results count is called `total`
* The default cursor is `null`

#### Configuring the CursorPaginator

To configure the `CursorPaginator` you can use the following methods

```php
<?php

use Saloon\Http\Paginators\CursorPaginator;

public function paginate(Request $request, mixed ...$additionalArguments): CursorPaginator;
{
    $paginator = new CursorPaginator($this, $request, 50, ...$additionalArguments);
    
    $paginator->setLimitKeyName('top');
    $paginator->setCursorKeyName('nextCursor');
    $paginator->setTotalKeyName('count');
    $paginator->setNextPageKeyName('meta.next');
    
    return $paginator;
} 
```
{% endtab %}
{% endtabs %}

### Iterating over responses and results

Now you have tested your request and configured the paginator, you're ready to give it for a spin! Saloon's paginators have many different ways to iterate over the results. The simplest way is using a foreach loop, however, you can use Laravel's Collections which helps you map and filter results really easily.

#### Foreach Loop

This is the simplest way to iterate over results. When you pass the paginator inside of a for-loop you will get each response.&#x20;

```php
<?php

$connector = new SpotifyConnector;

$paginator = $connector->paginate(new LikedSongsRequest);

foreach($paginator as $response) {
    $pageData = $response->json();
}
```

#### Collection

{% hint style="info" %}
The `collect` method requires Laravel's `illuminate/collections` package.
{% endhint %}

Similar to a for-loop, you can use a collection to wrap around the results.

```php
$connector = new SpotifyConnector;

$paginator = $connector->paginate(new LikedSongsRequest);

$collection = $paginator->collect();

// Supports everything you expect!

$collection->map(...)->filter(...)
```

By default, Saloon's collect method will return a LazyCollection to be as memory efficient as possible. You may choose to use the `lazy: false` argument.

```php
$collection = $paginator->collect(lazy: false);
```

You may choose to iterate over the results inside of the collection instead of the responses. If you would like to do this, provide the JSON key of the results...

```php
$collection = $paginator->collect('results');

$collection->map(function (array $track) {
    // You now get access to the internal results of the collection!
});
```

When you provide a key to the `collect` method, Saloon will automatically collapse the collection. This means it reduces it to individual items. If this causes unexpected behaviour, you can disable collapsing with `collapse: false`.

```php
$collection = $paginator->collect('results', collapse: false);
```

#### JSON method

The JSON method allows you to iterate over the internal results of the paginator, like iterating over each of the tracks in the Spotify request. Just pass in the JSON property of the results.

```php
<?php

$paginator = $connector->paginate(new LikedSongsRequest);

foreach($paginator->json('results') as $tracks) {
    foreach($tracks as $track) {
        //
    }
}
```

### Asynchronous requests and pooling

Saloon also supports asynchronous requests and request concurrency/pooling with paginators. This is really exciting because you can make hundreds of API calls in a fraction of the time compared to sending normally. If you are unsure of how Saloon handles request concurrency, [read here](concurrency-and-pools.md).

{% hint style="warning" %}
To use asynchronous requests/pooling, Saloon requires the API to return the "count" or "results" of the API. This is because asynchronous requests are not sent right away, Saloon does not know when to stop sending requests.&#x20;
{% endhint %}

#### Asynchronous Requests

You may send requests asynchronously with the paginator by using the `async` method. When this is enabled, every response will be an instance of `PromiseInterface`.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$paginator = $connector->paginate(new LikedSongsRequest);
<strong>$paginator->async();
</strong>
foreach ($paginator as $promise) {
    // Handle $promise
}
</code></pre>

#### Request Concurrency/Pooling

You may also use pools with the paginator. This allows you to send requests concurrently. You should provide the concurrency integer and the response handler.

```php
<?php

$paginator = $connector->paginate(new LikedSongsRequest);

$pool = $paginator->pool(concurrency: 5, responseHandler: function (Response $response) use (): void {
    //
})->send()
```

[Click here](concurrency-and-pools.md) to read more about request concurrency/pools.
