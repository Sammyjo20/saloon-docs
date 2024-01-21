# ⛵ Laravel Plugin

Saloon has been built to integrate beautifully with [Laravel](https://laravel.com). The separate Laravel plugin comes with a facade that helps with mocking and recording requests, Artisan console commands to really easily build your API integration, and even a separate default sender that uses Laravel's HTTP Client.

### Installation

You can install the separate package with Composer. You must already have`saloonphp/saloon` as a required dependency in your `composer.json` file.

```bash
composer require saloonphp/laravel-plugin "^3.0"
```

> Requires Laravel 9+

Next, publish the configuration file with the following Artisan command

```bash
php artisan vendor:publish --tag=saloon-config
```

### Available Laravel Commands

Each of the commands will create files within the `App\Http\Integrations` namespace. Each integration name is required for its own namespace. For example: `App\Http\Integrations\Forge`.

<table><thead><tr><th width="364">Command</th><th>Description</th></tr></thead><tbody><tr><td>saloon:connector &#x3C;Integration Name> &#x3C;Connector Name> <em>--oauth</em></td><td>Creates a new connector - You can provide an optional <code>--oauth</code> option if you would like to create an OAuth2 connector.</td></tr><tr><td>saloon:request &#x3C;Integration Name> &#x3C;Request Name></td><td>Creates a new request</td></tr><tr><td>saloon:response &#x3C;Integration Name> &#x3C;Response Name></td><td>Creates a custom response</td></tr><tr><td>saloon:plugin &#x3C;Integration Name> &#x3C;Plugin Name></td><td>Creates a plugin</td></tr><tr><td>saloon:auth &#x3C;Integration Name> &#x3C;Authenticator Name></td><td>Creates a custom authenticator</td></tr></tbody></table>

You can use the `saloon:list` command to get information about Saloon usage within your application. This includes details about your integrations, as well as associated requests, connectors, plugins, responses, and authenticators.

### Laravel HTTP Client Sender

Saloon comes with a sender built just for Laravel. The HTTP sender uses Laravel's [HTTP client](https://laravel.com/docs/9.x/http-client#main-content) under the hood, which allows your requests to be handled by Laravel just like using the HTTP client directly. This means Saloon's requests can be recorded in Telescope and also picked up by Laravel's event system.

#### Installation

The HTTP client sender comes in as a separate library. This is to keep its versioning separate from Saloon and the Laravel Integration. You can install it with Composer.

```bash
composer require saloonphp/laravel-http-sender "^2.0"
```

#### Configuration

Next, in your `config/saloon.php` file, change the default sender to `HttpSender::class`. Now every connector in your Laravel app will automatically use the HTTP sender. No more configuration is required, Saloon should work exactly the same as before, just now with full HTTP client support.

```php
<?php

declare(strict_types=1);

use Saloon\HttpSender\HttpSender;

return [

    /*
    |--------------------------------------------------------------------------
    | Default Saloon Sender
    |--------------------------------------------------------------------------
    |
    | This value specifies the "sender" class that Saloon should use by
    | default on all connectors. You can change this sender if you
    | would like to use your own. You may also specify your own
    | sender on a per-connector basis.
    |
    */

    'default_sender' => HttpSender::class,

];
```

Now when you send requests, they will be sent through Laravel's HTTP client - if you have Laravel Telescope installed, you should see the requests appearing under the "HTTP Client" tab of Telescope.

### Events

With the Laravel plugin installed, Saloon will start sending events when requests are being sent. These events are:

* SendingSaloonRequest
* SentSaloonRequest

These events can be added to your `EventServiceProvider` and you can create listeners to handle when these happen.

### Mocking Requests

Typically when mocking requests in Saloon, you are only limited to the current test you are in, without having to pass your `MockClient` down into every call. With the Laravel plugin installed, you may use the `Saloon::fake` method to configure mocking globally across your application. This is super handy if you want to test your API requests nested deep in your application.

[Click here](broken-reference) to read more about mocking requests.

### Laravel Zero

If you are using [Laravel Zero](https://github.com/laravel-zero/laravel-zero), the `SaloonServiceProvider` that registers the `Saloon` facade as well as some default middleware might not be registered. You can register Saloon's service provider in your `AppServiceProvider.php`'s `register()` method definition.

```php
<?php

use Saloon\Laravel\SaloonServiceProvider;

public function register()
{
    $this->app->register(SaloonServiceProvider::class);
}
```
