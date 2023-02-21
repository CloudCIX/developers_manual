# Errors

Error messages are defined in the individual applications.

Each service will have it's own errors/file.py where file is the name of the service. 

The a structure is as follows (using Membership as an example);
```
errors/
    __init__.py
    address.py
    address_link.py
    ...
```

Inside in `address.py` will be a set of variables as follows;

```python
membership_address_create_001 = 'The "member_id" parameter is invalid. "member_id" should be an integer'
membership_address_create_002 = 'The "member_id" parameter is invalid. "member_id" does not correspond with a valid Member'
...
```

The error codes should be grouped together by method and follow the same order as everything else; `list` -> `create` -> `read` -> `update` -> `delete`

The patten for naming error codes is application_service_method_XXX


## Use Of Error Codes
When using error codes, we will use the name of the variable in place of an error code, and the response handler will import that variable from the `errors` package to get a message for it.

## Ordering
The order for error code numbers should be as follows;

1. Codes thrown from the view (`0XX` codes)
2. Codes thrown from the controller (if any) (`1XX` codes)
3. Codes thrown from the permissions (if any) (`2XX` codes)

