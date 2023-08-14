# 👋 Installation

To get started with Saloon, you will need to install it through Composer.&#x20;

```bash
composer require saloonphp/saloon "^2.0"
```

> Saloon supports PHP 8.1+

### Dependencies

Saloon has just three dependencies.

* [Guzzle](https://github.com/guzzle/guzzle) (For The Default GuzzleSender)
* [Guzzle's Promise Library](https://github.com/guzzle/promises) (For Request Pooling & Asynchronous Requests)
* [PSR Message Library](https://github.com/php-fig/http-message) (For PSR-7 Interfaces)

### Using Laravel?

There is an additional Laravel Plugin you can install for Saloon which provides useful testing helpers, a Saloon facade and Artisan commands! To read more about the Laravel plugin [click here](../digging-deeper/laravel-integration.md)
