# 🌿 Upgrading from v1

### Introduction

There have been quite a number of changes in version two, it's recommended that you read through w[hat's new in v2](whats-new-in-v2.md) before starting the upgrade.

### Estimated Upgrade Time

Simple applications/Laravel integrations: \~15-30 minutes

Advanced integrations and SDKs: \~30-45 minutes

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
