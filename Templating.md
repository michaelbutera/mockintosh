# Response Templating

## Response Fields

In response template, one can specify `status`, `headers` and `body` of HTTP message. Here's quick example:

```yaml
services:
  - port: 8080
    endpoints:
      - path: /api-call
        response:
          status: 201
          headers:
            content-type: application/json
            x-custom-id: 12345
          body: '{"result": "created"}'            
```

Any of those fields allows using dynamic template that will be evaluated for each request. Like this:

```yaml
services:
  - port: 8080
    endpoints:
      - path: /api-call
        response:
          status: "{{request.queryString.rc}}"
          headers:
            content-type: '{{request.headers.accept}}'
            x-custom-id: '{{random.int 0 1000}}'
          body: '{"result": "created", "name": "{{fake.lastname}}" }'            
```

_Note: for `headers`, only the value part is subject for templating. Mind
the [global headers](Configuring.md#global-settings) feature, too._

The `body` can be direct response string:

```yaml
response:
  body: 'hello world'
```

or a string that starts with `@` sign to indicate a file on disk:

```yaml
response:
  body: '@some/path/my_template.json.hbs'
```

_Note: The template file path has to be relative to the directory of the config file._

## Dynamic Values

By default, dynamic templates use [Handlebars](https://handlebarsjs.com/guide/) syntax that looks like
this: `{{namedValue}}` or `{{request.path}}` or `{{fake.address}}` etc.

TODO: Can I use nested Handlebars expressions?
TODO: Can we keep expression as-is if we were unable to evaluate it or it was misconfigured?

_Note: To switch into [Jinja2](https://jinja.palletsprojects.com/en/2.11.x/) as templating engine, use
the `templatingEngine` option of [configuration syntax](Configuring.md#advanced-templating-with-jinja2)._

Below is the reference of available dynamic value generators.

### Random

- `random.int 10 20` - random integer between 10 and 20 (inclusive)
- `random.float '-0.5' '20.7' 3` - random float (between `-0.5` and `20.7` , inclusive) with 3 digit precision, you have to keep range values in quotes
- `random.ascii 5` - sequence of ASCII characters, length 5
- `random.alphanum 5` - sequence of alphanumeric characters, length 5
- `random.hex 16` - sequence of hexadecimal characters, length 16
- `random.uuid4` - UUID4 generator

For random names, addresses etc, please refer to [Faker's](#faker) functionality.

### Faker

[Faker](https://faker.readthedocs.io/en/master/providers.html) library is provided for generating some dynamic data. 
It is available as `fake` object. Refer to the [official docs](https://faker.readthedocs.io/en/master/providers.html) for all capabilities. Below are some examples:

- `fake.first_name`
- `fake.last_name`

### Counters

There is special kind of template helper, offering named counters like `{{counter 'counterName'}}`. The counters are global and identified by name. You can also refer to last value of counter by its name like this: `{{counterName}}`

### Request Object

You can reference parts of HTTP request in response templates. It is available as `request` object. These are the most
useful attributes:

- `request.path` - The path part of the URI e.g. `/some/path`.
- `request.path.<n>` - The specific component part of the URI
- `request.headers.<key>` - A request header e.g. `request.headers.accept`.
- `request.queryString.<key>` - A query parameter e.g. `request.queryString.a` is `hello world`.
- `request.body` - The raw request body as a whole. Can be `str`, `bytes` or `dict`.
- `request.formData.<key>` - The `POST` parameters sent in a `application/x-www-form-urlencoded` request
  e.g. `request.formData.param1` is `value1`.
- `request.files.<key>` - The fields in a `multipart/form-data` request.


And some less frequently used:

- `request.method` - [HTTP method](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
- `request.uri` - URI, full path segments including the query string e.g. `/some/path?a=hello%20world&b=3`.
- `request.host` - Full address of host e.g. `localhost:8001`.
- `request.hostName` - Only the hostname e.g. `localhost`.
- `request.protocol` - The HTTP protocol e.g. `http` or `https`.
- `request.remoteIp` - The IP address of the client e.g. `127.0.0.1`.
- `request.version` - HTTP version e.g. `HTTP/1.1`, [see](https://tools.ietf.org/html/rfc2145).