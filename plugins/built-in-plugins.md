# 🤠 Built-In Plugins

The following plugins come built-in with Saloon - however, there are some more advanced plugins that require a separate installation process. You can also build your own plugins to extend your Saloon integrations.

### AcceptsJson

This plugin will add the `Accept: application/json` header to your request before it is sent. This is a common header for JSON APIs.

```php
<?php

use Saloon\Traits\Plugins\AcceptsJson;

class ForgeConnector extends Connector
{
    use AcceptsJson;
    
    // ...
}
```

### AlwaysThrowOnErrors

This plugin will call the `$response->throw()` method. This method will throw an exception if the response has failed, rather than just returning a failed response. To learn more about how this plugin works, [click here](../the-basics/handling-failures.md#always-throw-exceptions-on-failed-requests).

```php
<?php

use Saloon\Traits\Plugins\AlwaysThrowOnErrors;

class ForgeConnector extends Connector
{
    use AlwaysThrowOnErrors;
    
    // ...
}
```

### HasTimeout

This plugin allows you to define a `connectTimeout` and `requestTimeout` property on your request or connector to overwrite the default connection timeout of 10 seconds and request timeout of 30 seconds.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Traits\Plugins\HasTimeout;

class ForgeConnector extends Connector
{
    use HasTimeout;

<strong>    protected int $connectTimeout = 30;
</strong>    
<strong>    protected int $requestTimeout = 120;
</strong>    
    // ...
}
</code></pre>
