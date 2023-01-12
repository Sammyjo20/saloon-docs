# Installation

To get started with Saloon, you will need to install it through Composer.&#x20;

If you are building an SDK or a package with Saloon, it's recommended that you use the standard PHP version.

If you are using Laravel, you can install the **saloon-laravel** package which installs Saloon and also provides you with Artisan commands to help you get up and running quickly, as well as better testing functionality.

{% tabs %}
{% tab title="Non-Laravel / PHP Installation" %}
#### Non-Laravel / PHP Installation

```bash
composer require sammyjo20/saloon "^1.0"
```

{% hint style="info" %}
Saloon requires PHP 8.0+
{% endhint %}
{% endtab %}

{% tab title="Laravel" %}
#### Laravel Installation

```bash
composer require sammyjo20/saloon-laravel "^1.0"
```

{% hint style="info" %}
Saloon requires PHP 8.0+ and Laravel 8+
{% endhint %}
{% endtab %}
{% endtabs %}

After you have installed Saloon, you're ready to start building your first Connector!
