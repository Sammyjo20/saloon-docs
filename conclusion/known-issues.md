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

    public function __constuct()
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
