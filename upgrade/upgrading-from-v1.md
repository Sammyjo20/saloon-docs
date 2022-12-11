# 🌿 Upgrading from v1

### Introduction

There have been several changes in version two, it's recommended that you read through [what's new in v2](whats-new-in-v2.md) before starting the upgrade.

### Estimated Upgrade Time

Simple applications/Laravel integrations: \~15-30 minutes

Advanced integrations and SDKs: \~30-45 minutes

### Installation

Firstly, run the following composer command to install Saloon v2

```bash
composer require sammyjo20/saloon ^2.0
```

#### Rename namespaces

The `Sammyjo20` username has been dropped from all namespaces. You should run a find and replace for the following strings in your application and tests.

| Find                                           | Replace                              |
| ---------------------------------------------- | ------------------------------------ |
| <pre><code>use Sammyjo20\Saloon\
</code></pre> | <pre><code>use Saloon\
</code></pre> |

#### Rename class names

The word `Saloon` has also been dropped from many of its classes. You should run a find and replace for the following strings in your application and tests.

| Find                                                      | Replace                                             |
| --------------------------------------------------------- | --------------------------------------------------- |
| <pre><code>use Saloon\Http\SaloonConnector;
</code></pre> | <pre><code>use Saloon\Http\Connector;
</code></pre> |
| <pre><code>use Saloon\Http\SaloonRequest;
</code></pre>   | <pre><code>use Saloon\Http\Request;
</code></pre>   |
|                                                           |                                                     |

#### Moving request data to the new body API

#### Other

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
