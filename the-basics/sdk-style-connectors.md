# Building SDKs

Saloon is a great tool to build SDKs. While you can build your own SDK patterns with Saloon, it also provides a simple framework for building SDKs rapidly.

{% hint style="info" %}
If you would like to get up and running quickly, try out the official Saloon SDK template repository on GitHub. [https://github.com/Sammyjo20/saloon-sdk-template](https://github.com/Sammyjo20/saloon-sdk-template)
{% endhint %}

### Individual Requests

If you define a \`$requests\` array property on your connector. You can specify individual requests that will be converted into "magic" methods.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use App\Http\Saloon\Requests\GetForgeServerRequest;

class ForgeConnector extends SaloonConnector
{
    protected array $requests = [
        GetForgeServerRequest::class, // $connector->getForgeServerRequest()
    ];

    // ...
}
```

Saloon will create a method for the request based on its name in camelCase. For example, our registered **GetForgeServerRequest** class will now have a method for it on the connector called `getForgeServerRequest`. When you call this method, Saloon will instantiate the **GetForgeServerRequest** class. Now we can send our request using the connector!

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

### Request Collections

You will often have many requests in an SDK, each separated into its own groups. Saloon also makes this easy with two ways of doing it. You can either define an array of requests in the \`$requests\` property on the connector, or you can create a **RequestCollection** class for unlimited customization and method-hinting.

#### Custom Request Collections

You can build custom request collections that will be returned automatically through magic methods. This is great if you would like to separate your SDK into different groups, like a "servers" group and a "sites" group containing different requests.

To get started, create a class and extend the base \`RequestCollection\` abstract class. Request collections will be given an instance of the connector which you can access.&#x20;

```php
<?php

use Sammyjo20\Saloon\Http\RequestCollection;

class ServerCollection extends RequestCollection
{
    public function all(): array
    {
        // You can access $this->connector to make requests...
        
        $request = $this->connector->request(new GetForgeServersRequest);
        $response = $request->send();
        
        return $response->json();
    }
    
    // ... Your other methods...
}
```

{% hint style="success" %}
The base **RequestCollection** class contains a constructor which provides the class with the connector instance.
{% endhint %}

After you have created the class, specify the collection in your \`$requests\` array on your connector.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use App\Http\Saloon\Requests\GetForgeServerRequest;

class ForgeConnector extends SaloonConnector
{
    protected array $requests = [
        'servers' => ServerCollection::class,
    ];
}
```

Now you can call the custom collection and methods inside of it!&#x20;

```php
<?php

$forgeConnector->servers()->all(); // ServerCollection@all

// You can even use static methods!

ForgeConnector::servers()->all();
```

#### Anonymous Request Collections

{% hint style="warning" %}
Anonymous request collections can be handy, but it will be difficult to define IDE-friendly type-hints for the methods inside of a collection. It is recommended to use the custom request collections above.
{% endhint %}

To create an anonymous request collection, just define a nested array keyed with the name of the group. Saloon will automatically create a magic-method that will return the request.&#x20;

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use App\Http\Saloon\Requests\GetForgeServerRequest;

class ForgeConnector extends SaloonConnector
{
    protected array $requests = [
        'servers' => [
            'all' => GetForgeServerRequest::class,
        ],
    ];
}
```

{% hint style="success" %}
You must provide a key to the array to define the name of the collection.
{% endhint %}

After you have defined the group, you can access the group like below!

```php
<?php

$forgeConnector = new ForgeConnector();

$forgeConnector->servers()->all(); // Will return GetForgeServerRequest

// You can even use static methods!

ForgeConnector::servers()->all();
```
