# ⛰ Built-In Plugins

### Built-in Plugins

Saloon comes with a few plugins out of the box, you may have already used them.&#x20;

#### AcceptsJson

This plugin will add the `Accept: application/json` header to your pending request. This is useful when dealing with JSON APIs.

#### AlwaysThrowOnErrors

This plugin will call the `$response->throw()` method. This method will throw an exception if the response has failed, rather than just returning a failed response.

#### HasTimeout

This plugin allows you to define a `connectTimeout` and `requestTimeout` property on your request or connector.
