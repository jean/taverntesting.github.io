# Plugins

Since 0.10.0, Tavern has a simple plugin system which lets you change how
requests are made. By default, all HTTP tests use the
[requests](http://docs.python-requests.org/en/master/) library and all MQTT
tests use the [paho-mqtt](https://www.eclipse.org/paho/clients/python/docs/)
library. 

However, there are some situations where you might not want to run tests against
something other than a live server, or maybe you just want to use curl to
extract some better usage statistics out of your requests. Tavern's plugin
system can be used to override this default behaviour (note however that it
still ONLY supports HTTP and MQTT requests at the time of writing).

The best way to introduce the concepts for making a plugin is by using an
example. For this we will be looking at a plugin used to run tests against a
local flask server called
[tavern_flask](https://github.com/taverntesting/tavern-flask).

## The entry point

Plugins are loaded using two setuptools entry points, namely `tavern_http` for
HTTP tests and `tavern_mqtt` for MQTT tests. The built-in requests and paho-mqtt
functionality is implemented using plugins, so looking at the `_plugins` folder
in the Tavern repository will also be useful as a reference when writing a
plugin.

The entry point needs to point to either a class or a module which defines a
preset number of variables.

### Extra schema data

If your plugin needs extra metadata in each test to be able to make a request,
extra schema data can be added with a `schema` key in your entry point. This
should be a dictionary which is just merged into the [base
schema](https://github.com/taverntesting/tavern/blob/master/tavern/schemas/tests.schema.yaml)
for tests.

There is currently only one key supported in the schema dictionary,
`initialisation`. This defines a top level key in each test which your session
or request classes can use to set up the test (see the [mqtt
documentation](https://taverntesting.github.io/documentation#testing-with-mqtt-messages)
for an example of how this is used to connect to an MQTT broker).

Examples:

- The
  [paho-mqtt](https://github.com/taverntesting/tavern/blob/master/tavern/_plugins/mqtt/schema.yaml)
  plugin defines the `client`, `connect`, etc. keys which are used to connect to
  an MQTT broker.
- [tavern-flask](https://github.com/taverntesting/tavern-flask/blob/master/tavern_flask/schema.yaml)
  just requires a single key that points to the flask application that will be
  used to create a test client (see below).

### Session type

`session_type` should return a class which describes a "session" which will be
used throughout the entire test. It should be a class that fulfils two
requirements:

1. It must take the same keyword arguments as the 'base' session object to
create an instance for testing. For
HTTP tests this is the same arguments as a
[requests.Session](http://docs.python-requests.org/en/master/user/advanced/#session-objects)
object, and for MQTT tests it is the same arguments as specified in the
[MQTT documentation](https://taverntesting.github.io/documentation#mqtt-connection-options).
If your plugin does not support some of these arguments, raise a
`NotImplementedError` which a short message explaining that it is not supported.

2. After creating the instance, it must be able to be used as a [context
manager](https://docs.python.org/3/library/stdtypes.html#typecontextmanager).
If you don't need any functionality provided by this, you can define empty
`__enter__` and `__exit__` methods on your class like so:

```python
class MySession(object):

    def __enter__(self):
        pass

    def __exit__(self, *args):
        pass
```

Examples:

- [tavern-flask](https://github.com/taverntesting/tavern-flask/blob/master/tavern_flask/client.py)
  is fairly simple, it just creates a flask test client from the `flask::app`
  defined for the test (see schema documentation above) and dumps the body data
  for later use when making the request.

### Request

`request_type` is a class that encapsulates the concept of a 'request' for your
plugin. It takes 3 arguments:

- `session` is the session instance created as described above, *for that
  request type at that stage*. There may be multiple request types per **test**,
  but only one request is made per **stage**.

- `rspec` is a dictionary corresponding to the request at that stage. If you are
  writing a HTTP plugin, the dictionary will contain the keys as described in
  the [http request
  documentation](https://taverntesting.github.io/documentation#request). If it
  is an MQTT plugin, it will contain keys described in the [MQTT publish
  documentation](https://taverntesting.github.io/documentation#mqtt-publishing-options).

- `test_block_config` is the global configuration for that test. At a minimum it
  will contain a key called `variables`, which contains all of the current
  variables that are available for formatting.

In the constructor, this request type should validate the input data and format
the request variables given the test block config.

The class should also have a `run` method, which takes no arguments and is
called to run the test. This should return some kind of class encapsulating
response data which can be verified by your plugin's response verifier class.

Tavern knows which request keyword (eg `request`, `mqtt_publish`) corresponds to
your plugin by matching it to the plugin's `request_block_name`. For the moment,
this should be hardcoded to `request` for HTTP tests.

### Getting the expected response

`get_expected_from_request` should be a function that takes 3 arguments:

- `stage` is the entire test stage (ie, including the request block, test name,
  response block, etc) as a dictionary

- `test_block_config` is as above

- `session` is as above

This function should use this input data to calculate the expected response and
perform any extra things that need doing based on the request or expected
response. This will normally just be formatting the response block based on the
variables in the test block config, but you may need to do extra things (such as
subscribing to an MQTT topic).

### Response

`verifier_type` is a class that encapsulate the concept of verifying a response
for your plugin. It should inherit from `tavern.response.base.BaseResponse`, and
take 4 arguments:

- `session` is as above

- `name` is the name of the test stage currently being run. This can be used for
  logging debug information.

- `expected` is the return value from `get_expected_from_request`.

- `test_block_config` is as above.

It should also define a couple of methods:

- `verify` takes one argument, which is the return value from the `run` method
  on your request class. It should read whatever information is relevant from
  this response object and verify that it is as expected. There are some
  utilities on `BaseResponse` to help with this, including printing errors and
  checking return values. The easiest way to 

- `__str__` should return a human-readable string describing the response. This
  is mainly for debugging, and should only give as much information as you think
  is required. For example, a HTTP response might be printed as (