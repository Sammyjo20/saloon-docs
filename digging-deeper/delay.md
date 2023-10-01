# ⏸ Delaying Requests

Sometimes in your application, you may want to introduce a delay in your requests. This may be to avoid hitting rate limits or to avoid overloading a development environment. With Saloon, you may use the `defaultDelay` and `delay` methods on either your connector or request to define a delay in milliseconds.

### Default Delay

You may also wish to define a default delay on your connector or request. You can do this by extending the `defaultDelay` method.&#x20;

{% tabs %}
{% tab title="Connector" %}
```php
<?php

class ForgeConnector extends Connector
{
    // Every request sent through this connector will have a 500ms delay.

    protected function defaultDelay(): ?int
    {
        return 500;
    }
}
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

class GetServersRequest extends Request
{
    // Every time this request is sent, a 500ms delay is added

    protected function defaultDelay(): ?int
    {
        return 500;
    }
}
```
{% endtab %}
{% endtabs %}

### Using the delay methods

You can also apply a delay to a connector or a request on the fly with the `delay()` method.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

$forge = new ForgeConnector;

// Delay every request sent with the ForgeConnector by 500ms 

$forge->delay()->set(500);
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

$request = new GetServersRequest;

// Delay just this request instance by 500ms

$request->delay()->set(500);
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
If you have defined the delay on both the connector and the request, the request delay will take priority.&#x20;
{% endhint %}
