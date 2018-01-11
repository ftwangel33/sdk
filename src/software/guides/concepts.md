---
layout: page
permalink: /software/guides/concepts/
title: Concepts
---

Requests made to the Geotab API are performed over HTTPS. The current API is version 1 — this is denoted in the API endpoint URL the web application will send requests to:

`https://[myserver]/apiv1`

Note: The portions of the examples noted with `[` and `]` (e.g. `[myserver]`) indicate where the user will enter information specific to their requirements.

API request parameters and the results are transported in the lightweight [JSON](http://www.json.org/) format. The [API reference](../../api/reference) contains a listing of the methods that can be invoked, the parameters they expect and the results they return. Below are examples to illustrate the capabilities of the Geotab API.

Requests to the API can be invoked using HTTP GET or POST. HTTP POST requests uses the JSON-RPC standard. The following sections explain how to construct HTTP GET and POST requests to the Geotab API.

The MyGeotab API only allows making requests over secure connections (HTTPS). The minimum SSL/TLS version supported by the MyGeotab API is TLS v1.2.

## HTTP GET request

Methods can be invoked via HTTPS GET request as follows:

`https://[myserver]/apiv1/[methodname]?[parameters]`

Here is a simple example of invoking the method GetVersion. This method does not require any parameters and can be called without credentials:

`https://my3.geotab.com/apiv1/GetVersion`

The HTTP response is returned as JSON. For example:

```json
{"result":"5.7.1508.122"}
```

Where the version will be the current version on the server.

### Make your first API call

Here is a more complex example that requires parameters. For the sake of clarity, the example below is not URI (Universal Resource Identifier) encoded (more on this below). Both POST and GET requests are supported. This example shows a GET request that returns all devices (vehicles) and their properties.

`https://[my3.geotab.com]/apiv1/Get?typeName=Device&credentials={"database":"[demo]","userName":"[bob@geotab.com]","password":"[xxx]"}`

The HTTP response is returned a JSON object (shortened for example purposes). The return will be similar to this format:

```json
{"result":[{"name":"Pickup truck", "id":"b0123"}]}
```

The full set of API methods and objects returned can be viewed in the [API reference](../../api/reference). When utilizing methods which require the user to send their login credentials as part of the URL, it is advised to take precautions regarding visibility of their password to other parties.

To understand how the parameters were passed in the URL, consider the following JSON object that needs to be passed to the method:

```json
{
    "typeName":"Device",
        "credentials": {
            "database":"acme",
            "userName":"bob@acme.com",
            "sessionId":"1234"
        }
}
```

Using an HTTP query string, all the JSON object properties above are serialized after the method name in the query string as key/value pairs, separated by ampersand characters as follows:

Generic:

`https://[...]?property1=value1&property2=value2`

Specific example:

`https://[...]?typeName=Device&credentials={"database":"demo"}`

Note that in the examples above, the property credentials are objects. In this case, stringify the JSON object and set that as the property value. Also take note that the query string should be properly URI encoded; paste the example below [here](http://www.url-encode-decode.com/) and select decode URL. The final HTTP GET request looks as follows:

`https://my3.geotab.com/apiv1/Get?typeName=Device&credentials={"database":"demo","userName":"bob@geotab.com","sessionId":"xxx"}`

> The MyGeotab API also supports JSONP when doing HTTP GET requests from JavaScript. This can be useful when writing a standalone HTML/JavaScript app. See the [Using in JavaScript](../using-in-javascript/) section for more information.

## HTTP POST request

When using HTTP POST request to invoke an API method, the same endpoint as the GET is used:

`https://[myserver]/apiv1/`

However, instead of encoding the method name and parameters in the query string, it is passed in the HTTP body using the [JSON-RPC](http://en.wikipedia.org/wiki/JSON-RPC)format. Geotab API version 1 supports JSON-RPC version 2.0.

The following is a JavaScript example that shows how an HTTP POST can be used to invoke a method. Note that this can be done from any language that has support for HTTP, for example the java.net.HttpUrlConnection class in Java or System.Net.Http.HttpClient in .Net can be utilized.

```javascript
var request = new XMLHttpRequest();
request.open("POST", "https://[myserver]/apiv1", true);
request.setRequestHeader("Content-Type", "application/json");
request.onreadystatechange = function () {
 if (request.readyState === 4) {
  if (request.status === 200) {
   var json = JSON.parse(request.responseText);
   if (json.result) {
    // Work with your result
    // Simple example just alerts its presence
    alert("Received Data");
   }
  }
 }
};

// Send the HTTP BODY in the JSON-RPC format.
// The method being called is "Get".
// The "Get" method's parameters are then passed in the "params" property

var data = {
 "id" : 0,
 "method" : "Get",
 "params" : {
  "typeName" : "Device",
  "credentials" : {
   "database" : "demo",
   "userName" : "bob@geotab.com",
   "sessionId" : "xxx"
  }
 }
};

request.send(JSON.stringify(data));
```

## Results and Errors

Using the example above, a successful call to the server will result in an object with property "result", in this format:

Generic:

```json
{
    "result":"results",
    "jsonrpc":"2.0"
}
```

Specific:

```json
{
    "result":"5.7.1801.122",
    "jsonrpc":"2.0"
}
```

However, when the call is incorrect or an error is triggered on the server, the error will be returned as an object with property "error". For example:

```json
{
    "error":{
        "code":-32000,
        "data":{
            "id":"5531c760-4ff7-485c-bb47-b6ed509b76d6",
            "type":"InvalidUserException",
            "requestIndex":0
        },
        "message":"Incorrect login credentials"
    },
    "jsonrpc":"2.0"
}
```

The properties of the error object are [JsonRpcError](../../api/reference/#T:Geotab.Checkmate.ObjectModel.Web.JsonRpcError) and [JsonRpcErrorData](../../api/reference/#T:Geotab.Checkmate.ObjectModel.Web.JsonRpcErrorData) objects as documented in the API Reference.

See [Example 3](#example-3-dealing-with-a-database-move-or-credential-expiry): Dealing with a database move or credential expiry for an example of a case where it would be useful to catch and handle errors.

## HTTP Compression

The MyGeotab API supports _gzip_ and _deflate_ compression. To utilize either of these supported compression methods, include the HTTP header for "Accept-Encoding". Ex:

Accept-Encoding: gzip, deflate

> If you are using an API wrapper (.Net, JavaScript, Nodejs, etc) this header should be enabled automatically.

## Authentication

Data is stored on one of many servers in our cloud. A group of servers is referred to as a _federation_ of servers. For example, the my.geotab.com federation consists of my1.geotab.com, my2.geotab.com and many other servers.

While it is tempting to simply "hard code" the application to point to a particular server, such as my20.geotab.com, this is the incorrect approach. Over the course of time a database is not guaranteed to remain on the same server. It is common for load balancing to occur and resources to be transferred from one server to another as necessary. To prevent the application from losing its connection to the correct database, authentication calls must be made to the root federation server instead of making the request to a particular server.

As an example; making an authentication call to "my.geotab.com" and authentication occurring. Such a call could be made as:

```js
var data = JSON.stringify({
  "method": "Authenticate",
  "params": {
    "database": "database",
    "userName": "user@geotab.com",
    "password": "password"
  }
});

var xhr = new XMLHttpRequest();

xhr.addEventListener("readystatechange", function () {
  if (this.readyState === 4) {
    console.log(this.responseText);
  }
});

xhr.open("POST", "https://my.geotab.com/apiv1");
xhr.setRequestHeader("content-type", "application/json");
xhr.setRequestHeader("cache-control", "no-cache");

xhr.send(data);
```

Where the database, user and password are set by you.

If a redirect is necessary, the application will be informed and the correct server can be targeted. Below are special cases which should be considered when accessing the federation.

### Example 1: Currently on the correct server

In this example, an authentication call is made to my.geotab.com to log into the database "_acme"_ and the server responds that the server is correct (e.g. no need to redirect).

Steps:

1. The **Authenticate** method is called with the given credentials
2. The response from the server contains two important properties: path and credentials

The path will either contain the URL of a server _or_ the string value _ThisServer_. Since the database "_acme"_ in this example is on my.geotab.com, it simply returns _ThisServer_ as highlighted below. This means that the correct server has been targeted and there is no need for a redirect.

The credentials object contains the username, database and session ID. This object will be used in all subsequent calls (password not required when used).

1. As the authentication method stated the server is correct, other methods can be called. For example _GetCountOf_, to my.geotab.com. Pass the property called _credentials_ and send along the contents of _credentials_ that was returned in step 2
2. The result of the GetCountOf is returned, in this case it is 1234

### Example 2: Redirect to different server

In this example, an authentication call to my.geotab.com is made trying to log into database "_acme"_. Here the server responds that the database is located on my23.geotab.com. Subsequent calls are then directed to my23.geotab.com

Steps:

1. The Authenticate method is called with the given credentials
2. The response from the server contains two properties: path and credentials

The path will either contain the URL of a server _or_ the string value _ThisServer_. Since the database "_acme"_ in this example is on my23.geotab.com, the return is "my23.geotab.com" meaning that all subsequent requests should be directed at my23.geotab.com.

The credentials object contains the username, database and session ID. This object will be required in all subsequent calls to the server for security reasons (password not required when used).

1. As the authentication method stated that _acme_ is on the my23.geotab.com server, other methods can be called, for example _GetCountOf_, to my23.geotab.com. Pass the property called _credentials_ and send along the contents of _credentials_ that was returned in step 2
2. The result of the GetCountOf is returned, in this case it is 1234

![]({{site.baseurl}}/software/guides/concepts_0.png)

### Example 3: Dealing with a database move or credential expiry

The examples in the previous sections demonstrated how to specify which server to communicate with during authentication. There are however two additional situations to consider:

1. the database has moved to a different server
2. the credentials object returned willeventually expire

When this happens the next API call will fail with the following error object:

```json
{
    "error":{
        "code":-32000,
        "data":{
            "id":"5531c760-4ff7-485c-bb47-b6ed509b76d6",
            "type":"InvalidUserException",
            "requestIndex":0
        },
        "message":"Incorrect login credentials"
    },
    "jsonrpc":"2.0"
}
```

When the errors collection contains an object with name _"InvalidUserException"_, the authentication process must be repeated. This will indicate what the new server is and provide a new credentials object. If the authentication process returns an error, the user was likely changed, in this case another user will be required for the login.

## Rate limiting

To safeguard optimal functioning and ensure that poorly written SDK applications do not impact the overall system performance,  a limit is imposed on Authentication calls. Starting in _November 2016_, the MyGeotab API will enforce limits on the number of Authentication requests it will accept. This change will start rolling out in November 2016 for _new_ customers, and will be fully enforced for _all_ customers by end of _January 2017_.

No more than **10 Authentication requests per minute** are permitted for a user. Both successful and unsuccessful Authentication calls count towards the limit. When the Authentication limit has been exceeded, the server will return an _OverLimitException_ until the limit is automatically reset every minute.

Authentication should be avoided by using the **credentials** object that is returned from an Authentication request for subsequent calls. It contains a token that confirms your identity for API operations. If the session expires or a database is moved, a new Authentication call should be made. This approach allows for efficient usage of Authentication requests and is detailed in the Authentication section above.

Credentials provided with password instead of or combined with session ID must be authenticated. Therefor, each request where credentials are provided in this way will tally against a given user's authentication limits.

## Working with dates

When exchanging dates as parameters to API methods, you must ensure that they are formatted properly as an [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) string (format `yyyy-MM-ddTHH:mm:ss.fffZ`). In addition, all dates will have to first be converted to [UTC](http://en.wikipedia.org/wiki/Coordinated_Universal_Time) in order to ensure time zone information and daylight savings times are accounted for correctly.

## Unit of measure

As a general rule, MyGeotab uses the metric system for values such as speed (km/h) and distance (m). For example, if you queried the odometer reading for a vehicle, the value would be returned in meters or if you retrieved the current speed of a vehicle it would be in km/h. It does not matter in which region in the world the vehicle or user of MyGeotab system is located — we always return the values in metric.A simple conversion can be applied to these values should you wish to work in imperial units or other customized units instead.

Please note that MyGeotab also records various other status data (i.e. engine data) from the vehicle and these values can be in various units of measure. The units of measure are not provided by Geotab in all cases. Refer to the applicable [SAE](http://standards.sae.org/automotive/) standard of the specific code for the associated unit of measure.

## Entities

All objects in the MyGeotab system are called entities. Entities have an ID property that is used to uniquely identify that object in the database. The ID is an opaque string value that uniquely identifies the entity and no assumption about the format or length of this ID value should be made when comparing or storing the values.

## ID

An ID is used to uniquely reference entities in the API. IDs are represented by opaque strings. Generally the contents of the IDs are not significant for the user. Building logic around the value of the string should be avoided — unless it is a system ID (see the examples below).

### Example 4

In this example, a vehicle in the system and its ID value will be examined. Here is a partial JSON representation of a device object:

```json
{
    "id": "b0a46",
    "name": "007 - Aston Martin",
    "serialNumber": "GTA9000003EA",
    "deviceType": "GO6",
    "vehicleIdentificationNumber": "1002"
}
```

Note the "id" property with value "b0a46". This is the unique identifier for the device (vehicle) with description "007 - Aston Martin".

To find Trips for this vehicle all of the properties of the device do not have to be passed to the Get method. Instead, only the ID property of the device object is required. Below is an example of a valid parameter object (TripSearch) for passing into Get method. The deviceSearch with the ID property set to the value "b0a46" (as indicated in the example above) is passed.

```json
{
  "typeName":"Trip",
  "search" : {
    "deviceSearch" : {
      "id" : "b0a46"
    }
  }
}
```

Calling the Get method with the parameter defined above will return all trips for the vehicle "007 - Aston Martin".

### Example 5

There are certain IDs that are predefined for system entities. For example the group that has been defined as the root group of all user groups, and called the CompanyGroup, will have an ID of "CompanyGroupId" rather than other characters (such as "b0a46" above). For example:

```json
{
    "id": "CompanyGroupId",
    "name": "The Company Group",
    "children": [..]
}
```

If the system entities do not have any properties then they are specified as strings with their ID's name.  For example the source "Obd" will be identified as "SourceObdId".

```json
{
    "code": "738960445",
    "engineType": {
        "id": "b2715",
    },
    "source": "SourceObdId"
}
```

## Building block approach

The results of a call to our API will only contain literal values and the identities of contained objects — not the actual fully populated child objects. This provides a predictable system that efficiently serializes objects to JSON and back. Additional lookups of the nested objects will be required to retrieve additional properties of the objects.

For example, an engine status data record has a device property. If 1000 engine status data records are retrieved for a device, the status data's device property will only contain the ID of the device. An additional retrieval for the devices object will be required to obtain the status data records. This approach has several benefits:

- Saves bytes over the wire
- Reduces request time
- Avoids redundant copies of entities
- More flexible since the child objects may not always be required

In the example below it can be seen how, by creating a dictionary of devices where the key is the device ID and the value is the device object, devices can be easily "stitched" into the status data records:

```javascript
var statusDatas = [{
        "id": "a1",
        "device": {
            "id": "b1"
        },
        "data": 0.002,
    ...
    },{
        "id": "a2",
        "device": {
            "id": "b1"
        },
        "data": 1.05,
    ...
    }
];

var deviceLookup = {
    "b1": {
        "id": "b1",
        "name": "Device 1",
        ...
    }
};
```

statusDatas[i].device = deviceLookup[statusDatas[i].device.id];

Depending on the process, for some entities like diagnostics, it may be desirable to maintain a local cache from which the status/fault data can be populated. In this case it will be necessary to refresh the cache when the cache is missing the required entity making an API call.This will allow the API to get the required entity and add it to the local cache. An example of maintaining a diagnostic cache would occur when consuming a feed of data from the API. An example of this process is included in both the C# and [JavaScript DataFeed](../js-samples/dataFeed.html) examples.

## MultiCall

A MultiCall is a way to make several API calls against a server with a single HTTP request. This eliminates potentially expensive round trip costs.

Why use a MultiCall?

Making an HTTP request over a network has overhead. This can be in the form of Network overhead, the round trip time to send and receive data over the network and HTTP overhead, the HTTP request and response headers. A MultiCall can be used to reduce amount of overhead in situations where many small requests need to be made to a server.

For example, if we make a request to get the max road speed from a set of points. The request would be constructed in a format similar to:

`https://my.geotab.com/apiv1/GetRoadMaxSpeeds?simplePoints=[{"y":[37.61610412597656],"x":[-121.8139419555664]}]&credentials={"database":"[demo]","userName":"[bob@geotab.com]","sessionId":"[xxx]"}`

Response:

`{"result":[88.513920000000013]}`

Making the assumption that it takes 100 milliseconds for this call round trip (the time from sending request to receiving the response), 40 milliseconds to send the request, 20 ms to process the data on the server and 40 ms for the response to be returned. [Google's SPDY research project](http://dev.chromium.org/spdy/spdy-whitepaper) [white paper](http://dev.chromium.org/spdy/spdy-whitepaper) states that "_typical header sizes of 700-800 bytes is common_". Based on this assumption, we pay a 750 byte cost when making a request. From the example, there would be 80 ms of network overhead and 750 bytes of HTTP overhead, this is accepted as the "cost of doing business" when making a request over a network.

Taking the previous assumptions, what would the overhead be for making 1000 requests for road max speeds? When individual calls are made to the server for 1000 addresses; the base (minimum) HTTP and Network overhead is required for each of these calls. This would result in 80 seconds (80,000 milliseconds) of network overhead and 5.72 MB (750,000 bytes) in headers just going to and from the server. It can be clearly seen that a great deal of overhead can be generated by making small but repeated requests.

By using a MultiCall, the network and HTTP overhead remains at the cost of a single request. This brings the overhead back down to our original 80 milliseconds and 750 bytes. The server processes each request and returns an Array of results when complete.

The above illustration is an extreme example to demonstrate the benefits of using a MultiCall. A MultiCall can (and should) be used to make short running calls of 2 or more requests more efficient than individual calls.

### Basic implementation

Making a MultiCall is very simple, use the method "ExecuteMultiCall" with the parameter "calls" of JSON type Array. Each call should be formatted as an Object with property "method" of type string with the method name as its value and a property "params" of type Object with the method parameters as its properties. The "params" object will also need to contain the user credentials if they are required for the method being called.

`https://my.geotab.com/apiv1/ExecuteMultiCall?calls=[{"method":"GetRoadMaxSpeeds","params":{"simplePoints":[{"y":37.61610412597656,"x":-121.8139419555664}],"credentials":{"database":"demo","userName":"bob@geotab.com","sessionId":"xxx"}}},{"method":"GetRoadMaxSpeeds","params":{"simplePoints":[{"y":38.61610412597656,"x":-122.8139419555664}]}}]`

Response:

```json
{
    "result": [[88.513920000000013],[-1.0]]
}
```

### Errors
In a MultiCall, each request is run on the server in syncronously. If one fails, the error results are returned immediately and **unreached calls are not run**. The error results includes the index of the call in the array that the exception occured.

To illustrate, let's assume an array of calls (api.multicall([call-a, call-b, call-c])) where call-b is formatted incorrectly.

```javascript
var calls = [
  call-a, // ran successfully
  call-b, // error occurred, throw and return error
  call-c  // never ran
]
```

Below is an example of the error result. The `requestIndex` property contains the index of the call that failed.

```javascript
results = {
    "error": {
        "message": "The method 'Foobar' could not be found. Verify the method name and ensure all method parameters are included.",
        "code": -32601,
        "data": {
            "id":"2901ac83-0d7f-41a1-9cca-fd4a68e77ae7",
            "type":"MissingMethodException",
            "requestIndex": 1
        }
    },
    "jsonrpc":"2.0"
}
```

Alternatively, a successful MultiCall would look similar to:

```javascript
calls = [
    call-a, // ran successfully
    call-b, // ran successfully
    call-c  // ran successfully
]
results = {
    "results": [
        [...],
        [...],
        [...]
    ]
}
```

### API wrapper support

All of the [API wrappers](../../api/wrappers/) have native support for making multi-calls. Below are examples of making multi-calls using the Javascript and .Net wrappers:

JavaScript API multi-call example:

```javascript
var calls = [
    ["Get", { typeName: "Diagnostic" }],
    ["Get", { typeName: "Source", search: {id: "SourceGeotabGoId"}}],
    ["Get", { typeName: "Controller" }]
];

api.multiCall(calls, function (results) {
    var diagnostics = results[0];
    var sources = results[1];
    var controllers = results[2];
}, function (errorString) {
    alert(errorString);
});
```

.Net nuget package multi-call example:

```csharp
var calls = new object[] {
    new object[] { "Get", typeof(Diagnostic), typeof(List<Diagnostic>)},
    new object[] { "Get", typeof(Source), new { search = new SourceSearch { Id = KnownId.SourceGeotabGoId } }, typeof(List<Source>)},
    new object[] { "Get", typeof(Controller), typeof(List<Controller>)},
};

var results = api.MultiCall(calls);

var diagnostics = (List<Diagnostic>)results[0];
var sources = (List<Source>)results[1];
var controllers = (List<Controller>)results[2];
```

### MultiCall FAQ

*Can I use a search in a multicall?*

Yes, it is possible to use a search in a multicall.

*When shouldn't I use a multicall?*

1. If you need to make a few requests that are long running and return a large amount of data.In these casesit may be preferable to make these requests singularly instead of running one request that continues for a very long time before completion. When the connection is held open for a long period of time you become increasingly susceptible to network interference that could terminate the request.
2. Manipulating data (Add,Set,Remove) is not recommended via a multicall. A muilticall is not transactional. Therefore, if call 1 of 3 to Add succeeds and call 2 of 3 fails, call 3 of 3 is not executed and call 1 is not rolled back. See "What if an error occurs in one of the MultiCall requests?" below for illustration.

*How many request can I put in a multicall?*

There is no limit on the number of requests that can be made in a multicall. When making a large number of requests it may be desirable to "chunk" the requests into several requests of a smaller and more manageable size.

*What if the call doesn't return a result?*

The index in the array of results will have a **null** value.