# Installation

To get started with Saloon, you will need to install it through Composer. If you are using Laravel, Saloon comes with a useful wrapper package that provides you with Artisan commands and extra mocking functionality.

If you are building an SDK or a package with Saloon, it's recommended that you use the Non-Laravel version.

{% tabs %}
{% tab title="Non-Laravel / PHP Installation" %}
#### Non-Laravel / PHP Installation

```bash
composer require sammyjo20/saloon
```

{% hint style="info" %}
Saloon requires PHP 8.0+
{% endhint %}
{% endtab %}

{% tab title="Laravel" %}
#### Laravel Installation

```bash
composer require sammyjo20/saloon-laravel
```

{% hint style="info" %}
Saloon requires PHP 8.0+ and Laravel 8+
{% endhint %}
{% endtab %}
{% endtabs %}

After you have installed Saloon, you're ready to start building your first Connector!
