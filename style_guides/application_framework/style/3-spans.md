# Spans
Spans are used by Jaeger to track requests through the system.
A span is automatically created by middleware when a request arrives at the server.

However, this span can be broken down into more chunks in the body of each request.

The following document contains details on how requests will be broken down.

## General
The blocks outlined below are related to both API and UI views and as such should be used in both also.

### Logical Blocks
Break view code down into it's constituent parts, surrounding each part with a child span like so;
```python
from django.conf import settings
...
def get(self, request: Request) -> Response:
    tracer = settings.TRACER
    with tracer.start_span('validating_controller', child_of=request.span) as span:
            controller = AddressListController(data=request.GET, request=request, span=span)
            controller.is_valid()
    ...
```

Some logical blocks that should be surrounded are as follows;
- `validating_controller` (use `as span` and pass the span into the controller, it will create a span for each validate method in the controller)
- `retrieve_requested_objects`
- `generating_metadata`
- `serializing_data`
- `checking_permissions`
- `saving_object`

If more are required for individual views, these can be done by the person writing the view, or requested by the reviewer in the MR.

### API Requests
If a view makes a request to our api (typically frontend but also some applications) then the current span should be passed through the wire to the next request.

According to the docs on our middleware, this is done as follows;

```python
request = request
headers = {}
settings.TRACER.inject(request.span, opentracing.Format.TEXT_MAP, headers):
for k, v in headers.items():
    request.add_header(k, v)
# Make your request
```
but we have no working example of this yet, so this is very much a WIP section.

## API Views
The blocks outlined below only affect API views, and as such should only be used when writing backend code.

### Controllers
When wrapping the `validating_controller` span, save it as a variable and pass it into the constructor of the Controller as `span=span`, as shown in the first code block.
This allows us to get a span for each of the `validate_` methods in each controller for a bit more information about the view.