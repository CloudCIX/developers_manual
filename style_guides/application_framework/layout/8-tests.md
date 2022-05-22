# Tests
Tests are where we test things, naturally.

We aim for 100% coverage, ignoring lines that we can't really test for, such as network based errors or special cases for user 1.

See [here](/style/tests) for the style guide on testing.

## Note for Django 2.2
In Django 2.2+, you need to declare a list of databases used at the top of the test classes. This will usually be just the application name.

Here's an example from `membership`
```python
class AddressCollection(TestBase):
    databases = ['membership']
```
