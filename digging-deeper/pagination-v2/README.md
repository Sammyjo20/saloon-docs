# 📚 Pagination

### Introduction

When building API integrations, you may encounter a scenario where the server does not provide all the results in a single list. Instead, it divides the results into several pages. This strategy is called pagination, and integrating it into your application can be tedious and repetitive. With Saloon, you can install this pagination plugin to reduce the boilerplate code and iterate through every result across _every_ page in one loop. **It's like magic!** Saloon has three paginators out of the box to support the most common types of pagination:

* **Paged pagination** - where you have to specify the next page as a query parameter (?page=1, ?page=2 and so on)
* **Limit/Offset pagination** - where you must specify two query parameters - a limit and an offset.
* **Cursor pagination** - where each page will return a "cursor" (a string) that you use to query the next page.

Saloon's paginators are custom [PHP iterators](https://www.php.net/manual/en/class.iterator.php)_,_ meaning they can be used in for-loops. They are also memory efficient, so they only keep one page in memory at a time, this means you can use Saloon's paginators to iterate through **thousands of pages and millions of results without running out of memory**.

Additionally, Saloon's paginators have a few really useful methods to traverse through the results, like mapping them into Laravel's famous "Collection" class or simply an array of results. Here's how the pagination looks in your application:

```php
<?php

$spotifyConnector = new SpotifyConnector;

$likedSongs = $spotifyConnector->paginate(new GetLikedSongsRequest);

foreach($likedSongs->items() as $likedSong) {
    // e.g. Luke Combs - When It Rains It Pours 🎵
}
```

{% hint style="info" %}
Saloon's paginators are versatile and can work with any API format, including JSON, XML, and even multipart responses.
{% endhint %}

### Getting Started

Saloon's paginators are provided through a plugin. First, install the following package with Composer.

```sh
composer require saloonphp/pagination-plugin "^1.0"
```

### Making your requests pageable

Once the pagination plugin is installed, you must implement an interface on the requests that use pagination. This interface enables the paginator to decide whether or not to try pagination. It's important to do this as it avoids developer mistakes and potentially causing infinite loops.

All you need to do is add the `Paginatable` interface to your requests.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
<strong>use Saloon\PaginationPlugin\Contracts\Paginatable;
</strong>
<strong>class GetLikedSongsRequest extends Request implements Paginatable
</strong>{
    // ...
}
</code></pre>

After that, you need to add the `HasPagination` interface to your connector. This will require you to define a `paginate` method which we will implement next.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Connector;
use Saloon\PaginationPlugin\Paginator;
use Saloon\PaginationPlugin\Contracts\HasPagination;

class SpotifyConnector extends Connector implements HasPagination
{
    // ...
    
<strong>    public function paginate(Request $request): Paginator
</strong>    {
        //
    }
}
</code></pre>

Now you need to choose a paginator that corresponds to the pagination type used by the third-party API. This documentation is divided into three sections based on Saloon's three pre-installed paginators.

{% content-ref url="paged-pagination.md" %}
[paged-pagination.md](paged-pagination.md)
{% endcontent-ref %}

{% content-ref url="limit-offset-pagination.md" %}
[limit-offset-pagination.md](limit-offset-pagination.md)
{% endcontent-ref %}

{% content-ref url="cursor-pagination.md" %}
[cursor-pagination.md](cursor-pagination.md)
{% endcontent-ref %}

### Building Your Own Paginators

Sometimes an API may implement their own kind of pagination and Saloon might not have a pagination supported out of the box for it. You can create your own paginators really easily. Follow the link below for building your own paginators.

{% content-ref url="custom-pagination.md" %}
[custom-pagination.md](custom-pagination.md)
{% endcontent-ref %}

### Using The Paginator

After you have configured your paginator, it's time to start using it! There are various ways to use the paginator, so we'll cover them in this part of the guide. First, let's take a look at how we instantiate our paginator. When we instantiate the paginator - Saloon won't make any requests. The requests are only made when we start iterating over the paginator.

```php
<?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);
```

As you can see, instantiating the paginator is as simple as running the method and passing in the paginatable request. We are now ready to start using the paginator.

#### Iterating Over Responses

The simplest way to start using the paginator is to use it inside of a `foreach` loop. This will return each page as a Saloon `Response` for you to access. Saloon will only keep one response in memory at a time so you won't have to worry about running out of memory.

```php
<?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);

foreach($paginator as $response) {
    $status = $response->status();
    $data = $response->body();
}
```

#### Iterating Over Items

One of the more exciting methods Saloon provides is the `items` method. This cool method will give you each item across multiple pages. With this, you can handle the individual items. This uses the `getPageItems` method which you defined earlier in your paginator.

```php
<?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);

foreach($paginator->items() as $song) {
    $songName = $song['name'];
    $songArtists = $song['artists'];
}
```

#### Using Laravel Collections (LazyCollection)

Another really cool method Saloon provides is the `collect` method. If you are using Laravel with Saloon, this method will work out of the box, but if you aren't using Laravel - you can add the support for this method by installing the "collections" library via Composer.

```bash
composer require illuminate/collections
```

This method will return a `LazyCollection` class. This class extends Laravel's base `Collection` class but works with custom iterators and keeps memory consumption low. The `LazyCollection` is really powerful when it comes to processing results from a paginated API. You can use the collection to filter, map, sort and much more all within just one iteration!

```php
<?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);
$collection = $paginator->collect();

$results = $collection
    ->filter(function (array $song) {
        return count($song['artists']) > 1;
    })
    ->map(function (array $song) {
        return $song['name'];
    })
    ->sort();
    
foreach($results as $song) {
    // $song: "When It Rains It Pours"
}
```

Even though in this example, we are doing multiple operations (filter, map, sort) the transformation will only happen when we iterate through the `LazyCollection` making it very performant and easy on system memory.

By default, Saloon will iterate through each result within each API page, but if you would rather have a collection that iterates through `Response` classes, you may do this too. Just set the `throughItems` argument to false.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);

<strong>$collection = $paginator->collect(throughItems: false);
</strong></code></pre>

{% hint style="info" %}
To see a full list of available methods on the `LazyCollection` you can [click here to read](https://laravel.com/docs/collections) the Laravel Documentation on Collections.
{% endhint %}

### Advanced Features

Saloon's paginators are a fantastic way of iterating through many page-separated results quickly and easily without writing boilerplate code in your application. So far we've covered the basics of pagination, but there's still plenty you can do. Within this section of the guide, we'll go through some more advanced features.

#### Casting Items Into DTOs

Saloon has a great way of casting responses into DTOs, but what if instead of getting the raw array or string item for each item in the paginator you wanted to resolve your DTO? You can simply use your `dto` or `dtoOrFail` method inside of the `getPageItems` method of your paginator. By using this method, you already have configured your DTO building logic on your connector. If you're not sure how to do this, [click here](../../the-basics/data-transfer-objects.md).

{% tabs %}
{% tab title="Paginator Config" %}
<pre class="language-php"><code class="lang-php">class SpotifyConnector extends Connector implements HasPagination
{
    public function paginate(Request $request): PagedPaginator
    {
        return new class(connector: $this, request: $request) extends PagedPaginator
        {
            // ...
            
            protected function getPageItems(Response $response, Request $request): array
            {
<strong>                return $response->dto();
</strong>            }
        }
    }
}
</code></pre>
{% endtab %}

{% tab title="Request" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

class GetLikedSongsRequest extends Request
{
    // {...}
    
    public function createDtoFromResponse(Response $response): array
    {
<strong>        return array_map(function (array $song) {
</strong><strong>            return Song::fromArray($song);
</strong><strong>        }, $response->json('items'));
</strong>    }
}
</code></pre>
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The `createDtoFromResponse` method on your request/connector must return an array.
{% endhint %}

You can also implement the DTO conversion logic inside of the `getPageItems` method like below.

```php
protected function getPageItems(Response $response, Request $request): array
{
    return array_map(function (array $song) {
        return Song::fromArray($song);
    }, $response->json('items'));
}
```

#### Asynchronous Pagination / Pooling

You may prefer to send your requests asynchronously or use a request pool for better performance. To get started with this, you must implement a new method on your paginator called `getTotalPages`. This method is invoked after the first request and is used to determine how many additional requests Saloon needs to send.

Let's start with defining our `getTotalPages` method. Some APIs may not return this information, and unfortunately, if they do not return this information then you may not be able to do asynchronous pagination.

<pre class="language-php"><code class="lang-php">&#x3C;?php

class SpotifyConnector extends Connector implements HasPagination
{
    public function paginate(Request $request): PagedPaginator
    {
        return new class(connector: $this, request: $request) extends PagedPaginator
        {
            // ...
            
<strong>            protected function getTotalPages(Response $response): int
</strong>            {
<strong>                return $response->json('total_results');
</strong>            }
        }
    }
}
</code></pre>

Next, instantiate your paginator and use the `async` method to enable asynchronous pagination. Now when you iterate through results, you will get a `Promise` instance instead of a Saloon Response.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);

<strong>$paginator->async();
</strong>
<strong>foreach($paginator as $promise) {
</strong>    // $promise->then()
}
</code></pre>

You can also use a paginator pool. If you are unfamiliar with how asynchronous request pools work, [check out this section of the documentation](../concurrency-and-pools.md) first.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);

<strong>$pool = $paginator->pool(concurrency: 10);
</strong>
$pool->withResponseHandler(function (Response $response) { 
    // Handle each response

    $status = $response->status();
    $data = $response->body(); 
});

<strong>$pool->wait();
</strong></code></pre>

#### Custom Per-Request Pagination

Sometimes you may wish to use a different paginator on your request than your connector. For example, an API might not use the same pagination across all its endpoints. To get started with this - first, implement the `HasRequestPagination` interface on your request. This interface will expect you to define a `paginate` method on your request.

```php
<?php

use Saloon\PaginationPlugin\Contracts\HasRequestPagination;

class GetLikedSongsRequest extends Request implements HasRequestPagination
{
    // {...}
    
    public function paginate(Connector $connector): Paginator;
    {
        //
    }
}
```

This method works just like the `paginate` method on your connector, so follow the steps above to configure your paginator. The main difference is that this method expects a connector to be passed into the constructor and not a request.

```php
<?php

use Saloon\PaginationPlugin\Contracts\HasRequestPagination;

class GetLikedSongsRequest extends Request implements HasRequestPagination
{
    // {...}
    
    public function paginate(Connector $connector): Paginator;
    {
        return new class(connector: $connector, request: $this) extends PagedPaginator
        {
            protected function isLastPage(Response $response): bool
            {
                return is_null($response->json('next_page_url'));
            }
            
            protected function getPageItems(Response $response, Request $request): array
            {
                return $response->json('items');
            }
        }
    }
}
```

Next, you can pass the connector into your request to instantiate the paginator.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$spotifyConnector = new SpotifyConnector;
$request = new GetLikedSongsRequest;

<strong>$paginator = $request->paginate($spotifyConnector);
</strong></code></pre>

#### Counting results

You may wish to count the pages of the paginator. You can use the `count()` PHP method to get the total number of pages from a paginator.&#x20;

<pre class="language-php"><code class="lang-php">&#x3C;?php

$spotifyConnector = new SpotifyConnector;

$paginator = $spotifyConnector->paginate(new GetLikedSongsRequest);

<strong>$pages = count($paginator);
</strong></code></pre>

{% hint style="warning" %}
You should use the `count()` method and not `iterator_count()` on the paginator because `iterator_count()` is not supported and you may get inaccurate results.
{% endhint %}

#### Handling Infinite Loops

While building your paginators, sometimes, you might run into a situation when the paginator is attempting to get the same page over and over again, causing an infinite loop. This is usually caused by mis-configuring the `isLastPage` method. To try to mitigate infinite loops, the paginator has a built-in safety feature that will throw an exception if the last five responses have exactly the same body.&#x20;

Saloon will throw a `PaginationException` with a message like:

> Potential infinite loop detected! The last 5 requests have had exactly the same body.

If you would like to disable this check, you can set the `detectInfiniteLoop` property on your paginator to false.

<pre class="language-php"><code class="lang-php">public function paginate(Connector $connector): Paginator;
{
    return new class(connector: $connector, request: $this) extends PagedPaginator
    {
<strong>        protected bool $detectInfiniteLoop = false;
</strong>    }
}
</code></pre>

{% hint style="warning" %}
Detecting infinite loops with asynchronous pagination is not supported.
{% endhint %}
