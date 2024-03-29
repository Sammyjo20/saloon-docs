# 🏗 Building Your Own Plugins

Saloon has plugins which make it easy for you to add logic to your connector or requests in a reusable and elegant way. Plugins are traits that can be added to either a request or a connector and have a special "boot" method which is invoked during the request lifecycle.

### Getting Started

It's easy to create your own plugin for Saloon, just create a trait and add it to either your connector or request. In this example, we will create a logging plugin that when added will log the request being sent to [Ray](https://myray.app/).

Our plugin will be called `HasLogging` . In order for Saloon to execute this plugin, we must create a public method that starts with `boot` followed by the name of the trait. For example: `bootHasLogging`. The method is given an instance of `PendingRequest`.

{% tabs %}
{% tab title="Definition (Trait)" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

trait HasLogging
{
<strong>    public function bootHasLogging(PendingRequest $pendingRequest): void
</strong>    {
        ray($pendingRequest);
    }
}
</code></pre>

{% hint style="warning" %}
You must not use `$this` inside of the trait, as mutating the original request or connector is discouraged. The PendingRequest is the source of truth while a request is being sent.
{% endhint %}
{% endtab %}

{% tab title="Usage (Connector)" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

class ForgeConnector extends Connector
{
<strong>    use HasLogging;
</strong>}
</code></pre>
{% endtab %}
{% endtabs %}

When you send your request, plugins are the first things that are invoked, even before the `boot` method. This is to allow maximum compatibility with [middleware ](../digging-deeper/middleware.md)and [authenticators](../the-basics/authentication.md). This also means that because it's the first process in the chain, other steps like middleware and the boot method will be able to overwrite anything added by a plugin.

<figure><img src="../.gitbook/assets/Saloon v2 (1).png" alt=""><figcaption></figcaption></figure>

Because we added the trait to our connector, every request will use the `HasLogging` plugin. If you would like the plugin to be applied to just one request, you can add the plugin to the request only.&#x20;

{% hint style="warning" %}
Take caution when adding the plugin to both the connector and the request, because the plugin will be executed twice.
{% endhint %}
