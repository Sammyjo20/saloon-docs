# Per-request Authentication

Some API integrations require you to make a request to create an access token that a subsequent request will use. When building this, it can add a level of complexity to your application because you have to make two requests for every single resource. Let's say you have two requests for an API. You have:

* GetAccessTokenRequest
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

In our connector, we'll extend the `boot()` method which is a method that runs before every request that is sent through that connector, and you get access to the special `PendingRequest` class which is a class that is used to store all the information of a request to be sent in a temporary way. We'll move our logic into here.

```php
<?php

use Saloon\Http\PendingRequest;
use Saloon\Http\Auth\TokenAuthenticator;

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
        
        $pendingRequest->authenticate(new TokenAuthenticator($authResponse->json()['token']));
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

You may also write additional logic inside of the boot method, like caching access tokens and reusing them.
