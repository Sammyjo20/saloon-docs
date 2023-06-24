# 📦 Request Body/Data

When sending HTTP requests, a common requirement is to attach a payload/body to POST, PUT, or PATCH requests, like JSON, XML or multipart data. Saloon makes this easy for you with built-in body traits.

### Getting Started

To get started, you will need to add the `HasBody` interface to your request. This interface is required as it tells Saloon to look for a `body()` method supplied by one of the body traits. Without this interface, Saloon will not send any request body to the HTTP client. Also make sure to change your method to POST, PUT or PATCH depending on the requirements of the API.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;

<strong>class CreateServerRequest extends Request implements HasBody
</strong>{
    protected Method $method = Method::POST;
}
</code></pre>

Next, you will need to add a trait to provide an implementation for the missing `body()` method. Saloon has a trait for all the common types of request bodies.

Continue reading below to understand more about the specific body type that you need.

{% content-ref url="json-body.md" %}
[json-body.md](json-body.md)
{% endcontent-ref %}

{% content-ref url="multipart-body.md" %}
[multipart-body.md](multipart-body.md)
{% endcontent-ref %}

{% content-ref url="stream-body.md" %}
[stream-body.md](stream-body.md)
{% endcontent-ref %}

{% content-ref url="form-body.md" %}
[form-body.md](form-body.md)
{% endcontent-ref %}

{% content-ref url="xml-body.md" %}
[xml-body.md](xml-body.md)
{% endcontent-ref %}

{% content-ref url="string-plain-text-body.md" %}
[string-plain-text-body.md](string-plain-text-body.md)
{% endcontent-ref %}

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}
