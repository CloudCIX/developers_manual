# Models
This is the folder where Models are located.
A Model is a representation of a database table in the form of a Python class.
There are a couple of things we do in addition to normal Django Models to improve the system, which are detailed below;

## Super Classes
In the `cloudcix_rest.models` file, there are two classes defined that are used for base classes; `BaseModel` and `BaseManager`.

### BaseModel
The `cloudcix_rest.models.BaseModel` class is an abstract model containing four fields; `created`, `deleted`, `extra`, and `updated`.
These fields were present on almost every model in the py2 version of the applications, so we moved them into a base class that is used in all applications.

Most models in an application will be subclasses of this class, but there will be some situations which will not require this inheritance.

On top of these fields, this `BaseModel` class also redefines the `objects` field with an instance of `cloudcix_rest.models.BaseManager`.

### BaseManager
The `cloudcix_rest.models.BaseManager` class is an extension of Django's Manager class, which handles retrieving objects from the database.
This extension overrides the `get_queryset` method to ignore any entry where `deleted` is not null in the database.
This was done manually in every single model request in the py2 framework, so we decided to abstract it into the manager instead.

For models with foreign key relationships, we have extended the base manager to use `select_related` and `prefetch_related` calls to prevent lazy loading in serializers.
These models will have their own `<Model>Manager` class defined in the same file, and will override the `objects` variable with an instance of their specific Manager.
These `<Model>Manager` classes must be subclasses of `cloudcix_rest.models.BaseManager`
