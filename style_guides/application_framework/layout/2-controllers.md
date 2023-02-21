# Controllers
Controllers are classes that are used to validate data that is provided by users of the software, including list filters, and creation data.
This document contains details on how these classes work and how to lay them out.

All Controllers defined in Applications should be subclasses of `cloudcix_rest.controllers.ControllerBase`.

Controllers are typically used for `list`, `create` and `update` methods, and there should typically be a controller for each of these methods for each service, provided the service uses the method.

## Table of Contents
- [Constructor Parameters](#contstructor-parameters)
- [Meta Class](#meta-class)
- [Class Variables](#class-variables)
- [Instance Variables](#instance-variables)
- [Properties](#properties)
- [Methods](#methods)
- [Writing Validation Methods](#writing-validation-methods)

## Constructor Parameters
The superclass constructor takes the following parameters;
- `request`: The request object that was passed in to the view. Used for accessing extra request details such as the user
- `instance`: An instance of a Model. This is used during update methods to update the existing row in the database.
- `data`: The data sent by the user to be validated. Usually `request.GET` for list controllers and `request.data` for create / update controllers
- `partial`: A boolean flag stating whether a field should be skipped if it is the validation order but not in the sent data. If the field was not sent and partial is False, the validation method will have `None` passed as it's parameter.
- `span`: A span object that is optionally passed into a controller instance. If this is passed, a child span will be created for each validate method to track the timing of the method.

## Meta Class
The Controller Meta class is a class defined *within* a Controller class, and contains metadata about the controller that is used during validation.

The Controller Meta class should be a subclass of `cloudcix_rest.controllers.ControllerBase.Meta` in order to inherit the default metadata.

The metadata fields and their purpose are as follows;
- `allowed_ordering`: A tuple of strings representing the fields that can be used to sort the data in a list. Used for `list` controllers.
- `max_list_limit`: The maximum limit that can be specified. Numbers higher than this will be replaced with the default. Used for `list` controllers.
- `model`: The model class that is used to create new instances. Used for `create` controllers.
- `normal_list_limit`: The default limit for the number of objects that will be returned in a list call, if the user has either not specified or specified an invalid number. Used for `list` controllers.
- `search_fields`: A mapping of fields in the model to tuples of strings that can be used to further affect searching. Used for `list` controllers.
    - An example of this mapping could be `{'id': ('in',)}`, which allows for search filters `id` and `id__in`.
- `validation_order`: A tuple of strings that represent the fields to be validated in the order they are to be validated. The `is_valid` method will iterate through this tuple and run methods name `validate_<field>` in order. Used for all controllers, but the default is fine for `list`, needs to be redefined for `create` and `update`.

## Class Variables
`ControllerBase` has two class variables that represent usual filters for `search_fields` that can be used instead.

`DEFAULT_STRING_FILTER_OPERATORS`: An alias for `('in', 'icontains', 'iendswith', 'iexact', 'istartswith')`, which are the typical operators for string type fields
`DEFAULT_NUMBER_FILTER_OPERATORS`: An alias for `('gt', 'gte', 'in', 'lt', 'lte', 'range')`, which are the typical operators for numeric type fields.

## Instance Variables
On top of the class variables, there are also instance variables that are handy to know, and will probably need to be used in the validation methods.
These are as follows;

- `cleaned_data`: A dictionary mapping strings to any data type. The keys should be names of fields in the Model in `create` and `update` controllers, since this dictionary is iterated through to set the field values when creating or updating a model
- `data`: Stores the uncleaned data that was passed to the controller from the view. Should never be used without cleaning the data first.
- `get_field`: Shortcut to the Controller's Model get_field method, allowing us to get fields from the model by name to inspect them. Typically used in validation methods to ensure a string will fit into the field.
- `partial`: Stores the partial flag that was passed to the controller from the view. Usually not used in validation methods, don't really see a reason why it ever would be.
- `request`: Stores the request object that was passed to the controller from the view. Commonly used in validation methods to check values on the requesting user.
- `span`: Stores the span object that was passed to the controller from the view. Used by base class methods to form child spans around validation methods.
  - Accessing `self.span` inside the validation method will access the appropriate child span for the method, so `self.span` can be used inside all validation methods
  - The only reason you would use this is for sending api requests from inside in validation methods 
- `validate_funcs`: A mapping of field names to the validation function for the field. Iterated through during the `is_valid` method to validate the user data. Built up using `Meta.validation_order`
- `_errors`: A dict mapping fields to error codes. Used by the `errors` property to generate error messages.
- `_instance`: Stores the instance that was passed to the controller from the view. Stored in a private variable to allow for the `instance` property, which handles updating the passed instance with the cleaned data.
- `_instance_prepared`: A boolean stating whether or not the `instance` property has been successfully accessed once before. This is used inside the `instance` property to avoid updating the fields multiple times.
- `_validated`: A boolean stating that the controller has been validated. Accessing the `instance` property before validating the controller will cause an error.
- `_warnings`: A deque of warnings that were generated during the validation of the user data. If a user sends an invalid search filter, it will be ignored and a warning will be returned to inform them.

## Properties
Below is a list of properties of controller instances.
Properties are methods that are accessed as instance variables, or instance variables that are generated through methods.

- `instance`: Returns an instance of the Model for the controller.
    - In `create` controllers, this property will instantiate a model instance using the `cleaned_data` dict to initialize its fields.
    - In `update` controllers, this property will update the passed instance with new values using the `cleaned_data` dict instead.
    - This property will raise an error in the following scenarios
        - There is no Model specified in the controller meta class
        - The `is_valid` method has not been called
        - Errors were generated during the validation process
- `errors`: Returns a mapping of field names to error dicts
    - Error dicts are of the form `{'code': xxx-xxx-xxx-xxx-xxx', 'message': 'Verbose message explaining the error'}`.
    - These are generated using the `{field: code}` mapping stored within `_errors`
- `warnings`: Returns the warnings field as a proper list so it can be serialized.

## Methods
Below is a list of methods defined within the `ControllerBase` class.
Some of these are validation methods used for the default `validation_order` values, which are used for `list` controllers.

- `is_valid`: Iterates through the `validate_funcs` dictionary, calling each of the validation methods in order to validate the user data, storing any errors that are generated.
    - If a `span` was passed to the instance, this method will wrap each validation call in a child span
- `validate_search`: Used to validate any search parameters sent by the user by checking against `Meta.search_fields`
- `validate_exclude`: Used to validate any exclusion parameters sent by the user by checking against `Meta.search_fields`
- `validate_order`: Used to validate any order parameters sent by the user by checking against `Meta.allowed_ordering`. If no / an invalid order parameter is sent by the user, defaults to `Meta.allowed_ordering[0]`
- `validate_limit`: Used to validate any limit parameters sent by the user by checking against `Meta.max_list_limit`. If the User sends no value, a value less than 0 or a value higher than the max allowed, the method sets it back to `Meta.normal_list_limit`
- `validate_page`: Used to validate any page parameters sent by the user. This is not a smart implementation so there is no specified max. The method only checks that the page value sent (if any) is at least 0. If none is sent, it will be set to 0.

## Writing Validation Methods
Here are some points to keep in mind when writing validation methods for a controller.

- The methods should all be named `validate_<field>`, where `<field>` is replaced with the name of the field that you are validating, since that is the name that will be looked up in the user's data
- All parameters to the method are Optional, since they might not have been sent
- The return type for all validation methods is `Optional[str]`
- If an error with the data is discovered, return the error code to be sent to the user instead of raising an error. This was done to speed up code by not having the overhead of exceptions.