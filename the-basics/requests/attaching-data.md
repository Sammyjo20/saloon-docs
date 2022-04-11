# Attaching Data

Most API integrations you will write will often require sending data using a POST/PUT/PATCH request. Saloon makes this easy for you. There are three traits that can be added to attach data to your request. For example, if the API you are integrating with accepts JSON, you should attach the **HasJsonBody** trait to your request.

* HasJsonBody
* HasFormParams
* HasMultipartBody

After you have added one of these three traits to your request, you will have access to use the available methods and can define default data.

### Example Request

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Features\HasJsonBody;

class CreateForgeSiteRequest extends SaloonRequest
{
    use HasJsonBody;
    
    //...
    
    public function defaultData(): array
    {
        return [
            'domain' => $this->domain,
            'type' => 'php',
        ];
    }
    
    public function __construct(
        public string $serverId,
        public string $domain,
    ){}
}
```

{% hint style="info" %}
If you would like to define data on both the connector and the request, you will need to add the same trait to both classes.
{% endhint %}

### Modifying data

During runtime, you can also overwrite or add to the request's data.

```php
<?php

$request = new CreateForgeSiteRequest($serverId, $domain);

$request->setData(['domain' => $customDomain]);

$request->mergeData(['database' => 'test123']);

$request->addData('name', 'my-saloon-server');

$request->getData('name'); // Returns "my-saloon-server".
```

### Available Methods

#### setData($array)

This method lets you overwrite the default data with an array of new data.

#### mergeData($array)

This method lets you merge a new array into the existing data array.

#### addData($parameterName, $parameterValue)

This method lets you add a single piece of data.

#### getData($parameterName)

This method lets you get a particular item in the data array by name.

### Other Form Body Types

Sometimes you will want to send form data that isn't composed in an array format. Saloon comes with a **HasBody** trait dedicated to custom form data like XML and even file streams. Saloon does have a dedicated XML trait, so if that is what you are looking for, [click here](attaching-data.md#sending-xml).&#x20;

After you have added the trait, you should add a `defineBody()` **** method which returns the response.

```php
<?php

namespace App\Http\Saloon\Requests;

use App\Http\Saloon\Connectors\ForgeConnector;
use Sammyjo20\Saloon\Constants\Saloon;
use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Features\HasBody;

class GraphQLRequest extends SaloonRequest
{
    use HasBody;

    // ...
    
    public function defineBody(): mixed
    {
        return 'custom-string-response';
    }   
}
```

{% hint style="info" %}
When using the **HasBody** trait, you should also specify the Content-Type header.
{% endhint %}

### Sending XML

You can use the **HasBody** trait for sending XML, but Saloon also has a dedicated **HasXMLBody** trait to send XML. It automatically adds the headers you need for sending XML.

After you have added the trait, you should add a `defineXMLBody()` method that should return the XML response as a string.

```php
<?php

namespace App\Http\Saloon\Requests;

use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Traits\Features\HasXMLBody;

class XMLRequest extends SaloonRequest
{
    use HasXMLBody;

    // ...
    
    public function defineXmlBody(): string
    {
        return '<?xml version="1.0" encoding="UTF-8"?>';
    }
}
```
