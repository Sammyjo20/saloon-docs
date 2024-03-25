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

