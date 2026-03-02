# ⛵ Laravel Plugin

Saloon has been built to integrate beautifully with [Laravel](https://laravel.com). The separate Laravel plugin comes with a facade that helps with mocking and recording requests, Artisan console commands to easily build your API integrations, and events which you can listen to and build further functionality from.

### Installation

You can install the separate package with Composer. You must already have `saloonphp/saloon` as a required dependency in your `composer.json` file.

```bash
composer require saloonphp/laravel-plugin "^3.0"
```

Next, publish the configuration file with the following Artisan command

```bash
php artisan vendor:publish --tag=saloon-config
```

### Artisan Console Commands

The Laravel plugin provides a few useful Artisan commands that can help you build your API integrations.

<table><thead><tr><th width="364">Command</th><th>Description</th></tr></thead><tbody><tr><td>php artisan saloon:connector</td><td>Creates a new connector - You can provide an optional <code>--oauth</code> option if you would like to create an OAuth2 connector.</td></tr><tr><td>php artisan saloon:request</td><td>Creates a new request</td></tr><tr><td>php artisan saloon:response</td><td>Creates a custom response</td></tr><tr><td>php artisan saloon:plugin</td><td>Creates a plugin</td></tr><tr><td>php artisan saloon:auth</td><td>Creates a custom authenticator</td></tr><tr><td>php artisan saloon:list</td><td>List all your API integrations</td></tr></tbody></table>

### Saloon Facade

The Laravel plugin also provides a facade which can be used to feel more unified with the rest of Laravel. You can use the Facade in tests instead of the `MockClient::global()` method.

```php
use Saloon\Laravel\Facades\Saloon;

Saloon::fake([
    GetServersRequest::class => MockResponse::make(body: '', status: 200),
]);
```

To learn more about testing API integrations, [click here](../the-basics/testing/).

### Laravel Nightwatch Support

From v3.6.0 the Saloon Laravel plugin natively supports Laravel Nightwatch. The plugin will automatically register a Nightwatch middleware on all of your connectors.

### Laravel Telescope Support

From v3.8.0 the Saloon Laravel plugin natively supports Laravel Telescope. The plugin will automatically register a Telescope middleware on all of your connectors.

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
