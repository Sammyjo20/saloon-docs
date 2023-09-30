# 🍳 Cookbook

Here you can find common scenarios encountered when building API integrations with Saloon. If you have any suggestions, please contribute by editing this page!

### Authenticating before every request&#x20;

Some API integrations require you to make a request to create an access token that a subsequent request will use. When building this, it can add a level of complexity to your application because you have to make two requests for every single resource. Let's say you have two requests for an API. You have:

* GetAccessTokenRequest&#x20;
* GetSongsByArtistRequest

But in order to make an API call `GetSongsByArtistRequest` you must obtain an access token by calling `GetAccessTokenRequest` first. Let's also assume that the `GetAccessTokenRequest` requires a username and password. Let's see how this may be implemented currently in your application.

```php
<?php

$api = new MusicApi();
$authResponse = $api->send(new GetAccessTokenRequest(username: 'Sam', password: 'yee-haw'));
$token = $authResponse->json()['token'];

$songsRequest = new GetSongsByArtistRequest('Luke Combs');
$songsRequest->withTokenAuth($token);

$response = $api->send($songsRequest);
```

This is pretty cumbersome. Every time you have to make a request, you have to write all these lines of code. One way you can get around this is by utilizing the `boot` method on your connector. Let's try refactoring this.

In our connector, we'll extend the `boot()` method which is a method that runs before every request that is sent through that connector, and you get access to the special `PendingRequest` class which is a class that is used to store all the information of a request to be sent in a temporary way. We'll move our logic into here.&#x20;

```php
<?php

use Saloon\Http\PendingRequest;

class MusicApi extends Connector
{
    // You should move any authentication requirements into the constructor
    // of the connector so your users have to provide the authentication
    // requirements before instantiating the connector.

    public function __construct(
        protected string $username,
        protected string $password,
    )
    {
        //
    }
    
    public function boot(PendingRequest $pendingRequest): void
    {
        // Let's start by returning early if the request being sent is the
        // GetAccessTokenRequest. We don't want to create an infinite loop
    
        if ($pendingRequest->getRequest() instanceof GetAccessTokenRequest) {
            return;
        }
        
        // Now let's make our authentication request. Since we are in the
        // context of the connector, we can just simply call $this and
        // make another request!
        
        $authResponse = $this->send(new GetAccessTokenRequest($this->username, $this->password));
        
        // Now we'll take the token from the auth response and then pass it
        // into the $pendingRequest which is the original GetSongsByArtistRequest.
        
        $pendingRequest->withTokenAuth($authResponse->json()['token']);
    }
}
```

Now when we send any request using this connector, the authentication will be handled every time! We do have to make sure to pass in our authentication requirements (username, password) into the connector, but this is way better than repeating the same code. Saloon is all about standardisation!

```php
<?php

$api = new MusicApi(
    username: 'Sam',
    password: 'yee-haw',
);

$response = $api->send(new GetSongsByArtistRequest('Luke Combs'));

// 200 body: [{name: 'When it rains, it pours'}]
```

You may also write additional logic inside of the boot method, like caching access tokens and reusing them.&#x20;

### Speeding Up Requests By Utilizing Laravel's Container

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
