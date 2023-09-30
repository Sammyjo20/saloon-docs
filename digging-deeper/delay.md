# ⏸ Request Delay

Sometimes in your application, you may want to introduce a delay in your requests. This may be to avoid hitting rate limits or to avoid overloading a development environment. With Saloon, you may use the `delay` method on either your connector or request. With this method, you can set a delay in milliseconds.&#x20;

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

### Default Delay

You may also wish to define a default delay on your connector or request. You can do this by extending the `defaultDelay` method.&#x20;

{% tabs %}
{% tab title="Connector" %}
```php
<?php

class ForgeConnector extends Connector
{
    // {...}

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
    // {...}

    // Every time this request is sent, a 500ms delay is added

    protected function defaultDelay(): ?int
    {
        return 500;
    }
}
```
{% endtab %}
{% endtabs %}
