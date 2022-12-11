# 🌿 Upgrading from v1

### Introduction

There have been several changes in version two, it's recommended that you read through [what's new in v2](whats-new-in-v2.md) before starting the upgrade.

### Estimated Upgrade Time

Simple applications/Laravel integrations: \~15-30 minutes

Advanced integrations and SDKs: \~30-45 minutes

### Installation

Firstly, run the following composer commands to install Saloon v2, depending on your installation.

{% tabs %}
{% tab title="Non-Laravel" %}
```bash
composer require sammyjo20/saloon ^2.0
```
{% endtab %}

{% tab title="Laravel" %}
```bash
composer remove sammyjo20/saloon-laravel

composer require sammyjo20/saloon ^2.0

composer require sammyjo20/saloon-laravel-plugin ^1.0
```
{% endtab %}
{% endtabs %}

#### Rename namespaces

The `Sammyjo20` username has been dropped from all namespaces. You should run a find and replace for the following strings.

| Find                                           | Replace                              |
| ---------------------------------------------- | ------------------------------------ |
| <pre><code>use Sammyjo20\Saloon\
</code></pre> | <pre><code>use Saloon\
</code></pre> |

#### Rename class names

The word `Saloon` has also been dropped from many of its classes. You should run a find and replace for the following strings.

| Find                                                                  | Replace                                                                  |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| <pre><code>use Saloon\Http\SaloonConnector;
</code></pre>             | <pre><code>use Saloon\Http\Connector;
</code></pre>                      |
| <pre><code>use Saloon\Http\SaloonRequest;
</code></pre>               | <pre><code>use Saloon\Http\Request;
</code></pre>                        |
| <pre><code>use Saloon\Http\SaloonResponse;
</code></pre>              | <pre><code>use Saloon\Http\Responses\Response;
</code></pre>             |
| <pre><code>use Saloon\Exceptions\SaloonRequestException
</code></pre> | <pre><code>use Saloon\Exceptions\Request\RequestException;
</code></pre> |
| <pre><code>extends SaloonConnector
</code></pre>                      | <pre><code>extends Connector
</code></pre>                               |
| <pre><code>extends SaloonRequest
</code></pre>                        | <pre><code>extends Request
</code></pre>                                 |
| <pre><code>extends SaloonResponse
</code></pre>                       | <pre><code>extends Response
</code></pre>                                |
| <pre><code>extends SaloonRequestException
</code></pre>               | <pre><code>extends RequestException
</code></pre>                        |

#### Replace method properties and methods

There have been a few properties and methods that have been renamed, or their visibility has changed. You should run a find and replace for the following strings.

| Find                                                              | Replace                                                            |
| ----------------------------------------------------------------- | ------------------------------------------------------------------ |
| <pre><code>public function defineBaseUrl(): string
</code></pre>  | <pre><code>public function resolveBaseUrl(): string
</code></pre>  |
| <pre><code>public function defineEndpoint(): string
</code></pre> | <pre><code>public function resolveEndpoint(): string
</code></pre> |
| <pre><code>protected ?string $response = 
</code></pre>           | <pre><code>protected string $response = 
</code></pre>             |
| <pre><code>protected ?string $method = 
</code></pre>             | <pre><code>protected Method $method = 
</code></pre>               |

#### Using the new headers, query, config API

#### Moving request data to the new body API

#### Other

* toException on the response has a changed return type from SaloonRequestException to Throwable
* getGuzzleException on the response has changed to getSenderException

### Changes

* Namespace renaming&#x20;
* Class renaming
* Changing data to body and using WithBody interface
* Changing location of body traits
* Changing defineEndpoint to resolveEndpoint
* Changing defineBaseUrl to resolveBaseUrl
* Removing the connector property on requests or adding back HasConnector
* Changing response interceptors to middleware
* Moving where Guzzle handlers are added
* withAuth renamed to authenticate
* Authenticators are executed at the end of a request preparation lifecycle rather than at the beginning
* toPsrResponse renamed to getPsrResponse on responses
* defineXMLBody renamed to defaultBody
* Request collections and magic request registration removed
* MockResponse::fromRequest has been removed
* Carbon has been replaced with DateTime instances
* MockClient and MockResponse moved to a different folder
* Request changed to PendingRequest inside of plugins and boot methods

### High Impact Changes

### Medium Impact Changes

### Low Impact Changes
