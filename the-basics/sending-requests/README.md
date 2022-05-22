# Sending Requests

After you have created your request, it is ready to be sent off to the internet! 🚀&#x20;

There are three ways to send requests. You can use the request class directly, send requests through your connector, or you can register requests on your connector and use method-style access to send your requests.

### Sending requests using the request class

Just instantiate your request class and use the `send()` method.&#x20;

```php
<?php

$request = new GetForgeServerRequest(serverId: '123456');

$response = $request->send();
```

### Sending requests using the connector class

This is useful if your connector has any constructor arguments. Just instantiate your connector class and use the `send()` method.&#x20;

```php
<?php

$connector = new ForgeConnector($apiToken);
$connector->request(new GetForgeServerRequest(serverId: '123456'));

$response = $request->send();

// or...

$connector->send(new GetForgeServerRequest(serverId: '123456'));
```

### Sending requests inside an SDK

Saloon offers a great framework for building SDKs. [Click here to read more](sdk-style-connectors.md)

### Modifying the request before it is sent

As mentioned previously, you can also use methods like `addHeader()` and `setConfig()` right before you send the request. This is useful if you have conditional headers, config or form data you need to pass in at the last minute.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;
use App\Http\Saloon\Requests\GetForgeServerRequest;

$request = new GetForgeServerRequest(serverId: '123456');

$request->setHeaders($array);
$request->addConfig('debug', true);
$request->addData('my-field', 'my-value');

$response = $request->send();

// Or if you are using the connector...

$connector = new ForgeConnector;
$request = $connector->request(new GetForgeServerRequest(serverId: '123456'));

$request->setHeaders($array);
$request->addConfig('debug', true);
$request->addData('my-field', 'my-value');

$response = $request->send();

// Or if you are using method-style access

$request = ForgeConnector::getForgeServerRequest(serverId: '123456');

$request->setHeaders($array);
$request->addConfig('debug', true);
$request->addData('my-field', 'my-value');

$response = $request->send();
```

{% hint style="info" %}
The **addData, setData** and **mergeData** methods will not do anything unless you [attach a form data trait](../requests/attaching-data.md).
{% endhint %}
