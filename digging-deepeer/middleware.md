# 💂 Middleware

* Mention Guzzle’s Handlers
  * They should always be used in the connector’s constructors and not in boot methods / plugins as they will be duplicated on each use.
