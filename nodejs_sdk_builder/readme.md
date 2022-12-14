# nodejs_sdk_builder.py
A Python 3 script that generates a Meraki Dashboard API SDK for Node.js.

# Project overview
The project consists of a Python 3 script and two template files, `sdk_core.template` and `endpoint.template`. It uses these files, as well as information retrieved from the Meraki Dashboard API to create a Javascript SDK.

Key features:
* Handles request timeouts, rate limiter status codes and multi-page responses in a tidy manner
* Includes a retry function for when hitting the rate limiter
* Returns a Promise that is resolved or rejected based on the API response, and can be handled with ".then" and ".catch"

# Running the script
* Install Python 3 if you have not done so already. If installing on Windows, it is recommended to select the "Add to PATH" option during installation
* Install the requests Python 3 module. The easiest way to do this is to run:
```
Windows:
pip install requests

Linux/Mac:
pip3 install requests
```

* Run the script, replacing `<api_key>` with your Meraki Dashboard API key:
```
Windows:
python nodejs_sdk_builder.py -k <api_key>

Linux/Mac:
python3 nodejs_sdk_builder.py -k <api_key>
```

* By default, the script will pull the list of available endpoints from the first organization accessible by your administrator account. If you want to specify which organization to pick, use this form instead, replacing `<api_key>` with your Meraki Dashboard API key and `<org_name>` with the name of the organization you want the script to use:
```
Windows:
python nodejs_sdk_builder.py -k <api_key> -o <org_name>

Linux/Mac:
python3 nodejs_sdk_builder.py -k <api_key> -o <org_name>
```

# Using the generated SDK
The script creates a Meraki Dashboard API SDK module for Node.js as a single file in the same directory as the script itself. By default, the output file will be named `Meraki_<timestamp>.js`.

The SDK requires the Axios package: https://www.npmjs.com/package/axios
    
How to use the output module:
* Copy the module to the same directory as your Node.js code
* Rename the file to `Meraki.js`
* Add the following lines to your code, replacing `<api_key>` with your Meraki Dashboard API key:
```
    const Meraki = require('./Meraki');
    var api = new Meraki.MerakiClass("<api_key>");
```
* After that, you can use the endpoint methods like in this example:
```
    api.getOrganizations()
        .then(function(response){
            // Code for handling request success
            console.log(response);
        }).catch(function (error) {
            // Code for handling request error
            console.log(error);
        });
```
        
How to find endpoints in the module file:
* Go to the Meraki Dashboard API documentation page: https://developer.cisco.com/meraki/api-v1/
* Find the endpoint that you want to use
* Copy its Operation Id and locate it in the module file using the search function of your text editor

General structure of endpoints in the generated SDK:
```
MerakiClass.<operation_id>(<url_param_1>, <url_param_2>, <query>, <body>)

```
    
Every endpoint method has a unique name that corresponds to the operation to be carried out by the Meraki Dashboard:
* `<operation_id>`: This is the Operation Id of the endpoint, as specified in the Meraki Dashboard API documentation page
    
Depending on the endpoint, it can require additional arguments to function. Refer to the particular endpoint method for its additional arguments. They can be the following:
* `<url_param_1>`, `<url_param_2>`: The URL of the endpoint you are using might contain variable parts. For example, getOrganizationNetworks requires an organizationId. If needed, these are mandatory
* `<query>`: If the endpoint you are using has the option to receive additional parameters as a query string, they can be provided using this argument object. See example below on how to use it
* `<body>`: If the endpoint you are using has the option to receive additional parameters as a request body, they can be provided using this argument object. See example below on how to use it
        
Using an endpoint that has query string parameter options:
```
    const Meraki = require('./Meraki');
    var api = new Meraki.MerakiClass("12345678");
    var serial = "AAAA-BBBB-CCCC";
    var query = { timespan: 10000 };
    api.getDeviceClients(serial, query)
        .then(function(response){
            // Code for handling request success
            console.log(response);
        }).catch(function (error) {
            // Code for handling request error
            console.log(error);
        });
```
        
Using an endpoint that has request body parameter options:
```
    const Meraki = require('./Meraki');
    var api = new Meraki.MerakiClass("12345678");
    var organizationId = "87654321";
    var body = { name: "New network" };
    api.createOrganizationNetwork (organizationId, body)
        .then(function(response){
            // Code for handling request success
            console.log(response);
        }).catch(function (error) {
            // Code for handling request error
            console.log(error);
        });
```

The endpoint methods return a Promise that is resolved or rejected with the following structure:
```
    {
        success: <boolean>,
        status: <integer>,
        data: <object>,
        errors: <object>
    }
```
* `success`: This is a boolean that flags if the request was successful or not. True indicates that the request was sent and the response had a HTTP status code of 2xx. False indicates failure to communicate with the Meraki Dashboard or a HTTP status code of 4xx/5xx
* `status`: The HTTP status code returned by the Meraki Dashboard
* `data`: The response body returned by the Meraki Dashboard, if the request was successful
* `errors`: The error explanations returned by the Meraki Dashboard, if any

# Useful links
* The official Meraki API developer page: https://developer.cisco.com/meraki
* Meraki Dashboard API quick reference with alpha/beta endpoint highlighting: https://github.com/mpapazog/meraki-diff-docs
