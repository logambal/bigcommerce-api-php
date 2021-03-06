Bigcommerce REST API V2
=======================

PHP package for connecting to the Bigcommerce V2 REST API.

To find out more, visit the official documentation website:
http://developer.bigcommerce.com/

Requirements
------------

- PHP 5.2.4 or greater
- cUrl extension enabled

To connect to the API, you need the following credentials:

- Secure URL pointing to a Bigcommerce store
- Username of an authorized admin user of the store
- API key for the user

To generate an API key, go to Control Panel > Users > Edit User and make sure
the 'Enable the XML API?' is ticked.

Installation
------------

Download the required PHP code for the Bigcommerce REST API client and copy it
to your PHP include path, or use the following command to install the package
directly (note: you may need to use sudo):

```
 $ pear channel-discover bigcommerce.github.com/bigcommerce-api-php
 $ pear install bigcommerce/Bigcommerce_Api-beta
```

Configuration
-------------

To use the API client in your PHP code, require the package from your include
path and provide the required credentials as follows:

```
require_once 'Bigcommerce/Api.php';

Bigcommerce_Api::configure(array(
	'store_url' => 'https://store.mybigcommerce.com',
	'username'	=> 'admin',
	'api_key'	=> 'd81aada4c19c34d913e18f07fd7f36ca'
));
```

Connecting to the store
-----------------------

To test that your configuration was correct and you can successfully connect to
the store, ping the getTime method which will return a DateTime object
representing the current timestamp of the store if successful or false if
unsuccessful:

```
$ping = Bigcommerce_Api::getTime();

if ($ping) echo $ping->format('H:i:s');
```

Accessing collections and resources (GET)
-----------------------------------------

To list all the resources in a collection:

```
$products = Bigcommerce_Api::getProducts();

foreach($products as $product) {
	echo $product->name;
	echo $product->price;
}
```

To access a single resource and its connected sub-resources:

```
$product = Bigcommerce_Api::getProduct(11);

echo $product->name;
echo $product->price;
```

To view the total count of resources in a collection:

```
$count = Bigcommerce_Api::getProductsCount();

echo $count;
```
Paging and Filtering
--------------------

All the default collection methods support paging, by passing
the page number to the method as an integer:

```
$products = Bigcommerce_Api::getProducts(3);
```
If you require more specific numbering and paging, you can explicitly specify
a limit parameter:

```
$filter = array("page" => 3, "limit" => 30);

$products = Bigcommerce_Api::getProducts($filter);
```

To filter a collection, you can also pass parameters to filter by as key-value
pairs:

```
$filter = array("is_featured" => true);

$featured = Bigcommerce_Api::getProducts($filter);
```
See the API documentation for each resource for a list of supported filter
parameters.

Updating existing resources (PUT)
---------------------------------

To update a single resource:

```
$product = Bigcommerce_Api::getProduct(11);

$product->name = "MacBook Air";
$product->price = 99.95;
$product->update();
```

You can also update a resource by passing an array or stdClass object of fields
you want to change to the global update method:

```
$fields = array(
	"name"  => "MacBook Air",
	"price" => 999.95
);

Bigcommerce_Api::updateProduct(11, $fields);
```

Creating new resources (POST)
-----------------------------

Some resources support creation of new items by posting to the collection. This
can be done by passing an array or stdClass object representing the new
resource to the global create method:

```
$fields = array(
	"name" => "Apple"
);

Bigcommerce_Api::createBrand($fields);
```

You can also create a resource by making a new instance of the resource class
and calling the create method once you have set the fields you want to save:

```
$brand = new Bigcommerce_Api_Brand();

$brand->name = "Apple";
$brand->create();
```

Deleting resources and collections (DELETE)
-------------------------------------------

To delete a single resource you can call the delete method on the resource object:

```
$category = Bigcommerce_Api::getCategory(22);
$category->delete();
```

You can also delete resources by calling the global wrapper method:

```
Bigcommerce_Api::deleteCategory(22);
```

Some resources support deletion of the entire collection. You can use the
deleteAll methods to do this:

```
Bigcommerce_Api::deleteAllOptionSets();
```

Using The XML API
-----------------

By default, the API client handles requests and responses by converting between
JSON strings and their PHP object representations. If you need to work with XML
you can switch the API into XML mode with the useXml method:

```
Bigcommerce_Api::useXml();
```

This will configure the API client to use XML for all subsequent requests. Note
that the client does not convert XML to PHP objects. In XML mode, all object
parameters to API create and update methods must be passed as strings
containing valid XML, and all responses from collection and resource methods
(including the ping, and count methods) will return XML strings instead of PHP
objects. An example transaction using XML would look like:

```
Bigcommerce_Api::useXml();

$xml = "<?xml version="1.0" encoding="UTF-8"?>
		<brand>
		 	<name>Apple</name>
		 	<search_keywords>computers laptops</search_keywords>
		</brand>";

$result = Bigcommerce_Api::createBrand($xml);
```

Handling Errors And Timeouts
----------------------------

For whatever reason, the HTTP requests at the heart of the API may not always
succeed.

Every method will return false if an error occurred, and you should always
check for this before acting on the results of the method call.

In some cases, you may also need to check the reason why the request failed.
This would most often be when you tried to save some data that did not validate
correctly.

```
$orders = Bigcommerce_Api::getOrders();

if (!$orders) {
	$error = Bigcommerce_Api::getLastError();
	echo $error->code;
	echo $error->message;
}
```

Returning false on errors, and using error objects to provide context is good
for writing quick scripts but is not the most robust solution for larger and
more long-term applications.

An alternative approach to error handling is to configure the API client to
throw exceptions when errors occur. Bear in mind, that if you do this, you will
need to catch and handle the exception in code yourself. The exception throwing
behavior of the client is controlled using the failOnError method:

```
Bigcommerce_Api::failOnError();

try {
	$orders = Bigcommerce_Api::getOrders();

} catch(Bigcommerce_Api_Error $error) {
	echo $error->getCode();
	echo $error->getMessage();
}
```

The exceptions thrown are subclasses of Bigcommerce_Api_Error, representing
client errors and server errors. The API documentation for response codes
contains a list of all the possible error conditions the client may encounter.

Specifying the SSL cipher
-------------------------

The API requires that all client SSL connections use the RC4-SHA (rsa_rc4_128_sha) cipher.
The client will set this cipher to be used by default.

The setCipher method can be used to override this setting if required.

```
Bigcommerce_Api::setCipher('RC4-SHA');
```

Verifying SSL certificates
--------------------------

By default, the client will attempt to verify the SSL certificate used by the
Bigcommerce store. In cases where this is undesirable, or where an unsigned
certificate is being used, you can turn off this behavior using the verifyPeer
switch, which will disable certificate checking on all subsequent requests:

```
Bigcommerce_Api::verifyPeer(false);
```

Connecting through a proxy server
---------------------------------

In cases where you need to connect to the API through a proxy server, you may
need to configure the client to recognize this. Provide the URL of the proxy
server and (optionally) a port to the useProxy method:

```
Bigcommerce_Api::useProxy("http://proxy.example.com", 81);
```
