# 👋 Installation

To get started with Saloon, you will need to install it through [Composer](https://getcomposer.org/).&#x20;

```bash
composer require saloonphp/saloon "^3.0"
```

### Using Laravel?

You can install an additional Laravel Plugin for Saloon which provides Artisan console commands, a facade and useful testing helpers. To read more about the Laravel plugin [click here](../plugins/laravel-integration.md).

### Dependencies

Saloon has just five dependencies, but two of them are used to provide Saloon with PSR interfaces.

* [Guzzle](https://github.com/guzzle/guzzle) (For the default GuzzleSender)
* [Guzzle's Promise Library](https://github.com/guzzle/promises) (For asynchronous requests and concurrency)
* [Guzzle's PSR-7 Library](https://github.com/guzzle/psr7) - (For Guzzle's PSR factories)
* [PSR Message Library](https://github.com/php-fig/http-message) (For PSR-7 interfaces)
* [PSR HTTP Factory](https://github.com/php-fig/http-factory) (For PSR-17 interfaces)

{% hint style="info" %}
All the Guzzle-related libraries are already installed with `guzzlehttp/guzzle` but as Saloon has used them directly, they have been explicitly required.
{% endhint %}
