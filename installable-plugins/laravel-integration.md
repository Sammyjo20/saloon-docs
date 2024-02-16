# ⛵ Laravel Plugin

Saloon has been built to integrate beautifully with [Laravel](https://laravel.com). The separate Laravel plugin comes with a facade that helps with mocking and recording requests, Artisan console commands to easily build your API integration, and even a separate default sender that uses Laravel's HTTP Client.

### Installation

You can install the separate package with Composer. You must already have`saloonphp/saloon` as a required dependency in your `composer.json` file.

```bash
composer require saloonphp/laravel-plugin "^3.0"
```

Next, publish the configuration file with the following Artisan command

```bash
php artisan vendor:publish --tag=saloon-config
```

### Artisan Console Commands

The Saloon Laravel plugin provides a few useful Artisan commands that can help you build your API integrations.

<table><thead><tr><th width="364">Command</th><th>Description</th></tr></thead><tbody><tr><td>php artisan saloon:connector</td><td>Creates a new connector - You can provide an optional <code>--oauth</code> option if you would like to create an OAuth2 connector.</td></tr><tr><td>php artisan saloon:request</td><td>Creates a new request</td></tr><tr><td>php artisan saloon:response</td><td>Creates a custom response</td></tr><tr><td>php artisan saloon:plugin</td><td>Creates a plugin</td></tr><tr><td>php artisan saloon:auth</td><td>Creates a custom authenticator</td></tr><tr><td>php artisan saloon:list</td><td>List all your API integrations</td></tr></tbody></table>

### Laravel HTTP Client Sender

Saloon comes with a sender built just for Laravel. The HTTP sender uses Laravel's [HTTP client](https://laravel.com/docs/9.x/http-client#main-content) under the hood, which allows your requests to be handled by Laravel just like using the HTTP client directly. This means Saloon's requests can be recorded in Telescope and also picked up by Laravel's event system.

{% hint style="info" %}
The Laravel HTTP sender plugin does not support the **Http::fake()** method.
{% endhint %}

#### Installation

The HTTP client sender comes in as a separate library. This is to keep its versioning separate from Saloon and the Laravel Integration. You can install it with Composer.

```bash
composer require saloonphp/laravel-http-sender "^2.0"
```

#### Configuration

Next, in your `config/saloon.php` file, change the default sender to `HttpSender::class`. Now every connector in your Laravel app will automatically use the HTTP sender. No more configuration is required, Saloon will work the same as before, just now using the HTTP client.

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

### Saloon Facade

The Laravel plugin also provides a facade which can be used to feel more unified with the rest of Laravel. You can use the Facade in tests instead of the `MockClient::global()` method.

```php
use Saloon\Laravel\Facades\Saloon;

Saloon::fake([
    GetServersRequest::class => MockResponse::make(body: '', status: 200),
]);
```

To learn more about testing API integrations, [click here](../the-basics/testing/).
