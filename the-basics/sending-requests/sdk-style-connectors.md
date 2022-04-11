# SDK-style Connectors

If you would like to use your connector like it's an SDK, where each request is their own method on the connector, Saloon makes this easy. Start by registering your requests in the **$requests** array on your connector.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use App\Http\Saloon\Requests\GetForgeServerRequest;

class ForgeConnector extends SaloonConnector
{
    protected array $requests = [
        GetForgeServerRequest::class,
    ];

    // ...
}
```

After that, Saloon will create a method for the request based on its name in camelCase. For example, our registered **GetForgeServerRequest** class will now have a method for it on the connector called `getForgeServerRequest`. When you call this method, Saloon will instantiate the **GetForgeServerRequest** class. Now we can send our request using the connector!

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

$connector = new ForgeConnector();
$request = $connector->getForgeServerRequest(serverId: 123456);

$response = $request->send();
```

You can also access the methods as static methods if you don't want to instantiate your connector.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

$request = ForgeConnector::getForgeServerRequest(serverId: 123456);

$response = $request->send();
```

#### Customising the request methods on the connector

You may want to use your own method names for requests on your connector. If you would like to do this, just add a key for the request to rename the method.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use App\Http\Saloon\Requests\GetForgeServerRequest;

class ForgeConnector extends SaloonConnector
{
    protected array $requests = [
        'get_server' => GetForgeServerRequest::class,
    ];

    // ...
}
```
