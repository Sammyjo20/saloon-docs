# 🐞 Known Issues

### Usage of anonymous functions with long-running processes like Laravel Queues

There appears to be an issue with PHP which means any object that has an anonymous function inside of one of its properties will result in PHP not properly destructing the class or running the `__destruct()` method. With Saloon, take care when using anonymous functions inside your connector or solo request. We have found that if you have an anonymous function/closure inside of the connector and when running it from within a long-running process like Laravel Queues, it won't close cURL connections.

This is because as mentioned above, the connector isn't able to be appropriately destructed which results in the sender and the sender's HTTP client like Guzzle not being able to be destructed and finally resulting in a "leak" of cURL connections. This can cause further issues like the operating system running out of file handles and causing errors like "Too many files open".&#x20;

You can workaround this by either using invokable classes or binding your closures to a new class instance directly.

```php
<?php

class SDK extends Connector
{
    public function __constuct()
    {
        // This will *not* work:
    
        $this->callable = function () {
            // Do something
        };
        
        // This will work: 
        
        $this->callable = Closure::bind(function () {
            // Do something
        }, new class {});
    
    }
}
```
