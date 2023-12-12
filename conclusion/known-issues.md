# 💥 Known Issues

### Usage of anonymous functions with long-running processes like Laravel Queues

There appears to be an issue with PHP which means any object that has an anonymous function inside of one of its properties will result in it not properly being destructed or running the `__destruct()` method. With Saloon, take care when using anonymous functions inside your connector or solo request because we have found that if you have an anonymous function/closure inside of the connector and when running it from within a long-running process like Laravel Queues, it won't close connections.

This is because as mentioned above, the connector isn't able to be appropriately destructed which results in the sender and the sender's HTTP client like Guzzle not being able to be destructed and finally resulting in a "leak" of connections. This can cause further issues like the operating system running out of file handles and causing errors like "Too many files open".&#x20;

You can workaround this by changing the way you define your anonymous closure. You can:

* Move the logic into an invokable class
* Make the closure _static_

```php
<?php

class SDK extends Connector
{
    protected Closure $callable;

    public function __construct()
    {  
        // ❌ This will cause connection leaks
        $this->middleware()->onRequest(function () {
            //
        });
        
        // ✅ Using an invokable class will work
        $this->middleware()->onRequest(new MyRequestMiddleware);

        // ✅ Using static will work
        $this->middleware()->onRequest(static function () {
            //
        });
    }
}
```

### Minimum TLS Version

From version three, Saloon uses Guzzle `^7.6` which introduced a new option to configure a minimum TLS version. With Saloon v3, Saloon has set this default to **TLS 1.2.** If any of your API integrations require a lower level of TLS security, you can change the Saloon config while your application is loading.

```php
use Saloon\Config;

Config::$defaultTlsMethod = STREAM_CRYPTO_METHOD_TLSv1_1_CLIENT;
```

You can [visit this page on the PHP documentation](https://www.php.net/manual/en/function.stream-socket-enable-crypto.php) to see the different TLS options available.&#x20;

{% hint style="info" %}
**TLS 1.1** has been deprecated since 2021 so it is unlikely that APIs are still using it, but older systems may have not upgraded yet.
{% endhint %}
