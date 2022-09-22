# Query Parameters

Saloon Connectors and Requests can also be configured to send query parameters with the request.&#x20;

### Example Request

To add query parameters to your request, you can use the **defaultQuery** method on the connector or request. Here you can specify any query parameters that should be sent.

```php
<?php

namespace App\Http\Saloon\Requests;

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;

class GetForgeServersRequest extends SaloonRequest
{
    // ...
    
    public function defaultQuery(): array
    {
        return [
            'sort' => 'updated_at',
        ];
    }
}
```

### Modifying query parameters

During runtime, you can also overwrite or add to the request's query parameters.&#x20;

```php
<?php

$request = new GetForgeServersRequest();

$request->setQuery(['sort' => $sort]);

$request->mergeQuery(['include' => 'user']);

$request->addQuery('X-Identifier', 'Saloon');

$request->getQuery('X-Identifier'); // Returns "Saloon".
```

### Available Methods

#### setQuery($array)

This method lets you overwrite the default query parameters with an array of new ones.

#### mergeQuery($array)

This method lets you merge a new array into the existing query parameter array.

#### addQuery($parameterName, $parameterValue)

This method lets you add a query parameter in your code after you have instantiated the request class.

#### getQuery($parameterName)

This method lets you get a particular query parameter by name.

