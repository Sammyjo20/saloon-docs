# 🏭 SDK Generator

The Saloon SDK Generator is a third-party Saloon plugin written and maintained by Crescat-IO (Helge Sverre) and allows people to easily generate Saloon SDKs from an OpenAPI file or Postman collection. The generator can automatically create connectors, requests and responses for you to help save you time when starting a new SDK.&#x20;

{% hint style="info" %}
Note: This tool helps you set up the foundation for your SDK, but it might not create a complete, ready-to-use solution.
{% endhint %}

<figure><img src="../.gitbook/assets/header (1).png" alt=""><figcaption></figcaption></figure>

### Installation

You can install the Saloon SDK Generator through Composer. You can install it globally on your machine and have a simple CLI to generate SDKs.

```sh
composer global require crescat-io/saloon-sdk-generator
```

{% hint style="info" %}
The library requires PHP 8.1 and above
{% endhint %}

### Basic Usage

To generate the PHP SDK from an API specification file, run the following command:

```sh
sdkgenerator generate:sdk API_SPEC_FILE.{json|yaml|yml}
     --type={postman|openapi} 
     [--name=SDK_NAME] 
     [--output=OUTPUT_PATH] 
     [--namespace=Company\\Integration] 
     [--force] 
     [--dry] 
     [--zip]
```

Replace the placeholders with the appropriate values:

* `API_SPEC_FILE`: Path to the API specification file (JSON or YAML format).
* `--type`: Specify the type of API specification (`postman` or `openapi`).
* `--name`: (Optional) Specify the name of the generated SDK (default: Unnamed).
* `--namespace`: (Optional) Specify the root namespace for the SDK (default: `App\\Sdk`).
* `--output`: (Optional) Specify the output path where the generated code will be created (default: ./Generated).
* `--force`: (Optional) Force overwriting existing files.
* `--dry`: (Optional) Perform a dry run. It will not save generated files, only show a list of them.
* `--zip`: (Optional) Use this flag to generate a zip archive containing all the generated files.

### Full Documentation

The full documentation can be found by visiting the Github repository here:

{% embed url="https://github.com/crescat-io/saloon-sdk-generator" %}

### Issue Tracking

As this is not directly affiliated with Saloon.Please report any issues you have directly on the repository linked above.
