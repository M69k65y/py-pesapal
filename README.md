# py-pesapal

## About
py-pesapal is a package that helps Python developers easily integrate their applications with PesaPal.


## Installation
    pip install py-pesapal


## Basic Usage
There are two ways to use the package, one specifically geared towards Flask applications.

### General Usage
```python
import uuid
from py_pesapal import Pesapal

pesapal = Pesapal(
    consumer_key = "12345",
    consumer_secret = "12345"
)

transaction_data = {}
line_items = []
transaction_total = []

x = 1
while(x <= 5):
    items_dict = {}
    items_dict["item_id"] = x
    items_dict["item_name"] = "Product {}".format(x)
    items_dict["item_count"] = 1
    items_dict["unit_cost"] = 10 * x
    items_dict["subtotal"] = 1 * 10 * x
    transaction_total.append(1 * 10 * x)
    
    line_items.append(items_dict)
    x += 1

transaction_data["line_items"] = line_items
transaction_data["amount"] = sum(transaction_total)
transaction_data["description"] = "Test transaction"
transaction_data["reference"] = str(uuid.uuid4())
transaction_data["email"] = "mail@example.com"
transaction_data["currency"] = "KES"

# POST A DIRECT ORDER
callback_url = "https://example.com/callback-url"

# Build URL to post a direct order
post_order = pesapal.post_order(callback_url = callback_url, transaction_data = transaction_data)


# QUERY PAYMENT STATUS
merchant_ref = "your-unique-transaction-identifier"
pesapal_tracking_id = "pesapal-provided-id"

# Build URL to query payment status
payment_status = pesapal.query_payment_status(merchant_ref = merchant_ref, pesapal_tracking_id = pesapal_tracking_id)


# QUERY PAYMENT STATUS BY MERCHANT REF
merchant_ref = "your-unique-transaction-identifier"

# Build URL to query payment status
payment_status = pesapal.query_payment_status_by_merchant_ref(merchant_ref = merchant_ref)


# QUERY PAYMENT DETAILS
merchant_ref = "your-unique-transaction-identifier"
pesapal_tracking_id = "pesapal-provided-id"

# Build URL to query payment details
payment_details = pesapal.query_payment_details(merchant_ref = merchant_ref, pesapal_tracking_id = pesapal_tracking_id)
```

### Usage in Flask
The basic usage of the package follows the same pattern as highlighted above with the main difference coming in creating a new instance of PesapalFlask.

```python
from flask import Flask
from py_pesapal import PesapalFlask

app = Flask(__name__)

pesapal = PesapalFlask(app, config = {
	"PESAPAL_CONSUMER_KEY": "12345",
	"PESAPAL_CONSUMER_SECRET": "12345",
	"PESAPAL_CALLBACK_URL": "https://example.com/callback-url"
})
```


## Advanced Usage (Flask)
Using PesapalFlask in a Flask application makes it possible to have some advanced usage that is not possible if using Pesapal.

Currently, this advanced usage is limited to rendering a [payment iframe](https://developer.pesapal.com/how-to-integrate/php-sample) in a Jinja2 template page.
```python
import uuid
from flask import Flask, render_template
from py_pesapal import PesapalFlask

app = Flask(__name__)

pesapal = PesapalFlask(app, config = {
	"PESAPAL_CONSUMER_KEY": "12345",
	"PESAPAL_CONSUMER_SECRET": "12345",
	"PESAPAL_CALLBACK_URL": "https://example.com/callback-url"
})

@app.route("/payment", methods = ["GET])
def return_payment_page():
    transaction_data = {}
    line_items = []
    transaction_total = []

    x = 1
    while(x <= 5):
        items_dict = {}
        items_dict["item_id"] = x
        items_dict["item_name"] = "Product {}".format(x)
        items_dict["item_count"] = 1
        items_dict["unit_cost"] = 10 * x
        items_dict["subtotal"] = 1 * 10 * x
        transaction_total.append(1 * 10 * x)
        
        line_items.append(items_dict)
        x += 1

    transaction_data["line_items"] = line_items
    transaction_data["amount"] = sum(transaction_total)
    transaction_data["description"] = "Test transaction"
    transaction_data["reference"] = str(uuid.uuid4())
    transaction_data["email"] = "mail@example.com"
    transaction_data["currency"] = "KES"

    return render_template("payment.html", data = transaction_data)
```

```html
<!-- payment.html -->
<!DOCTYPE html>
<html>
    <head>
        <title> Fancy Page Title </title>
    </head>

    <body>
        {{ render_iframe(data)|safe }}
    </body>
</html>
```


## Configuration
There are a number of configuration options when creating a new instance as highlighted below.

### Pesapal
```python
pesapal = Pesapal(
    consumer_key = "12345",
    consumer_secret = "12345",
    testing = True, # Optional
    prettyprint_xml = True, # Optional
    save_xml_file = False, # Optional
    xml_output_dir = None, # Optional
    param_validation = True # Optional
)
```
- `consumer_key` **(Required)**: The consumer_key value provided by PesaPal.
- `consumer_secret` **(Required)**: The consumer_secret value provided by PesaPal.
- `testing` **(Optional)**: Default value is `True`. Sets the base PesaPal URL as [https://demo.pesapal.com](https://demo.pesapal.com) if `True` or [https://pesapal.com](https://pesapal.com) if `False`.
- `prettyprint_xml` **(Optional)**: Default value is `True`. Sets whether to prettyprint the generated XML data.
- `save_xml_file` **(Optional)**: Default value is `False`. Sets whether the generated XML data should also be saved to a file on disc.
- `xml_output_dir` **(Optional)**: Default value is `None`. If `save_xml_file` is set to `True`, this sets the desired output directory. If this param is not set, then the default directory used is `xml/` in the current working directory.
- `param_validation` **(Optional)**: Default value is `True`. By default, py-pesapal requires the email address of whoever it is transacting. This is in contrast to PesaPal which requires one of either the email address or phone number to be provided. Use this param to switch off the validation that is handled by py-pesapal and default to PesaPal's native field validation.

### PesapalFlask
```python
from flask import Flask
from py_pesapal import PesapalFlask

app = Flask(__name__)

pesapal = PesapalFlask(app, config = {
	"PESAPAL_CONSUMER_KEY": "12345",
	"PESAPAL_CONSUMER_SECRET": "12345",
    "PESAPAL_TESTING": True, # Optional
	"PESAPAL_CALLBACK_URL": "https://example.com/callback-url",
    "PESAPAL_PRETTYPRINT_XML": True, # Optional
    "PESAPAL_SAVE_XML": False, # Optional
    "PESAPAL_OUTPUT_DIR": None, # Optional
    "PESAPAL_PARAM_VALIDATION": True # Optional
})
```

Alternatively, the configuration values can be associated with the application instance.
```python
from flask import Flask
from py_pesapal import PesapalFlask

app = Flask(__name__)
app.config["PESAPAL_CONSUMER_KEY"] = "12345",
app.config["PESAPAL_CONSUMER_SECRET"] = "12345",
app.config["PESAPAL_TESTING"] = True, # Optional
app.config["PESAPAL_CALLBACK_URL"] = "https://example.com/callback-url",
app.config["PESAPAL_PRETTYPRINT_XML"] = True, # Optional
app.config["PESAPAL_SAVE_XML"] = False, # Optional
app.config["PESAPAL_OUTPUT_DIR"] = None, # Optional
app.config["PESAPAL_PARAM_VALIDATION"] = True # Optional

pesapal = PesapalFlask(app)
```


## Utils
The py-pesapal package exposes various utilities should a developer want more control over the process of integrating with PesaPal.

```python
from py_pesapal.utils import generate_oauth_url

# BUILD OAUTH1 URL
url = generate_oauth_url(
    consumer_key = "12345",
    consumer_secret = "12345",
    url = "https://demo.pesapal.com",
    request_params = {
        "oauth_callback": "https://example.com/callback-url"
    }
)
```
- `consumer_key` **(Required)**: The consumer_key value.
- `consumer_secret` **(Required)**: The consumer_secret value.
- `url` **(Required)**: This is the base URL to which the various params are appended.
- `request_params` **(Required)**: The params that are appended to the base URL.

```python
from py_pesapal.utils import validate_params

# VALIDATE PARAMS
required_params = {
    "email": True,
    "phone_number": True,
    "first_name": False
}

params_to_be_validated = {
    "email": "mail@example.com",
    "phone_number": "phone-number"
}

# Raises a KeyError if one or more params are missing
validate_params(params = params_to_be_validated, required_params = required_params)
```
- `params` **(Required)**: This is a dictionary of the params one wishes to validate.
- `required_params` **(Required)**: This is a dictionary containing the name of the param and a Boolean indicating whether it is required or not.

```python
import uuid
from py_pesapal import generate_xml

transaction_data = {}
transaction_data["amount"] = 120
transaction_data["description"] = "Test transaction"
transaction_data["reference"] = str(uuid.uuid4())
transaction_data["email"] = "mail@example.com"
transaction_data["currency"] = "KES"

# GENERATE XML
xml = generate_xml(
    transaction_data = transaction_data,
    prettyprint = True,
    generate_xml_file = False,
    xml_output_directory = None
)
```
- `transaction_data` **(Required)**: The transaction data to be posted
- `prettyprint` **(Optional)**: Sets whether or not to prettyprint the resultant XML.
- `generate_xml_file` **(Optional)**: Sets whether or not to save the XML data to a file on disc.
- `xml_output_directory` **(Optional)**: Sets the output directory for XML files if `generate_xml_file` is set to `True`. The default directory is `xml/` in the current working directory.


## Licence
[MIT](https://choosealicense.com/licenses/mit/)