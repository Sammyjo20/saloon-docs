# Improving Speed With Laravel

If you are using Laravel, then you are probably aware of its powerful container that keeps objects open in the background while you make requests or process jobs. Saloon uses Guzzle under the hood, which keeps connections open for as long as it can - which is why subsequent requests using the same connector is usually faster. Typically though, the Guzzle HTTP client will be destructed and connections closed between requests/jobs. This results in increased request times if you are doing lots of API calls across jobs or requests because it is establishing a new connection each time.

If you are using Laravel, you can bind an instance of Saloon's `GuzzleSender` inside of its container and it will keep connections open between queued jobs and requests (if you are using Octane). This can provide a nice speed boost if you are using the same API service, especially if you are frequently accessing an API, like migrating data from an external API into your application.

First, you should create a new singleton in one of your service provider's register methods, and return a new instance of `GuzzleSender`. The `AppServiceProvider` is a good place to put this, or you could create a new `HttpServiceProvider` for this.

```php
<?php

use Saloon\Http\Senders\GuzzleSender;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(GuzzleSender::class, fn () => new GuzzleSender);
    }
}
```

Now, inside your Saloon connectors, you should overwrite the `defaultSender` method and resolve the sender which you have bound to the container.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Contracts\Sender;
use Saloon\Http\Senders\GuzzleSender;

class SpotifyConnector extends Connector
{
    protected function defaultSender(): Sender
    {
        return resolve(GuzzleSender::class);
    }
}
```

Now when you make requests with this connector, it will re-use the same Guzzle sender and the same Guzzle client under the hood!
