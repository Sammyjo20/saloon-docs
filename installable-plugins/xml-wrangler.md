# 🏇 XML Wrangler

XML Wrangler is a first-party library written by the maintainer of Saloon and is designed to modernise reading and writing XML in PHP. XML Wrangler can detect errors in XML, read gigabytes of XML files and convert XML results into Laravel Collections. Its powerful reader supports dot notation and XPath queries.

### Key Features

* Convert XML into an array
* Write XML from an array
* Memory-safe reader for large XML documents
* Dot notation and XPath support for reading

{% tabs %}
{% tab title="Reader" %}
```php
use Saloon\XmlWrangler\XmlReader;

$reader = XmlReader::fromString($xml);

// Convert XML to an array

$reader->values(); // ['breakfast_menu' => [['name' => '...'], ['name' => '...'], ['name' => '...']]

// Query the XML

$reader->value('food.0')->sole(); // ['name' => 'Belgian Waffles', 'price' => '$5.95', ...]

$reader->xpathValue('//food[@bestSeller="true"]/name')->get(); // ['Belgian Waffles', 'Berry-Berry Belgian Waffles']

$reader->element('food.0')->sole()->getAttributes(); // ['soldOut' => false, 'bestSeller' => true]
```
{% endtab %}

{% tab title="XML" %}
```xml
<breakfast_menu>
  <food soldOut="false" bestSeller="true">
    <name>Belgian Waffles</name>
    <price>$5.95</price>
    <description>Two of our famous Belgian Waffles with plenty of real maple syrup</description>
    <calories>650</calories>
  </food>
  <food soldOut="false" bestSeller="false">
    <name>Strawberry Belgian Waffles</name>
    <price>$7.95</price>
    <description>Light Belgian waffles covered with strawberries and whipped cream</description>
    <calories>900</calories>
  </food>
  <food soldOut="false" bestSeller="true">
    <name>Berry-Berry Belgian Waffles</name>
    <price>$8.95</price>
    <description>Light Belgian waffles covered with an assortment of fresh berries and whipped cream</description>
    <calories>900</calories>
  </food>
</breakfast_menu>
```
{% endtab %}
{% endtabs %}

### Documentation

Click the card below to read the documentation

{% embed url="https://github.com/saloonphp/xml-wrangler" %}

### Saloon Integration

XML Wrangler is also integrated with Saloon! Once installed, you'll be able to use the `xmlReader()` method on your responses to instantiate an XML Wrangler reader.

```php
$response = $connector->send($request);

$reader = $response->xmlReader();

// $reader->values();
// etc...
```

#### Request Body

You may also choose to use XML Wrangler to create the request body when sending XML requests in Saloon.

```php
<?php

use Saloon\Http\Request;
use Saloon\XmlWrangler\XmlWriter;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasXmlBody;
use Saloon\XmlWrangler\Data\Element;

class CreateServerRequest extends Request implements HasBody
{
    use HasXmlBody;

    protected Method $method = Method::POST;
    
    public function __construct(
        protected readonly string $ubuntuVersion,
        protected readonly string $type,
        protected readonly string $provider
    ){}
    
    protected function defaultBody(): string
    {
        return XmlWriter::make()->write('root', [
            'ubuntu-version' => $this->ubuntuVersion,
            'type' => $this->type,
            'provider' => $this->provider,
        ]);
        
        // Same as...
        
        // <?xml version="1.0" encoding="utf-8"?>
        // <root>
        //   <ubuntu-version>ubuntuVersion</ubuntu-version>
        //   <type>type</type>
        //   <provider>provider</provider>
        // </root>
    }
}
```
