# Test Styling
This is a minor style guide to keep in mind when writing tests.

As part of the upgrade to Python 3, we're trying our best to keep things as similar and neat as possible, which includes our test cases.

Below are some of the things to keep in mind when writing test suites for Applications;

## File Names
Each file in the `tests` directory should be related to a single Service.

Each file should be named `test_{service}.py`, with `{service}` replaced with what ever service you are writing tests for.

## Test Suite Classes
There should be a test suite class for each of the classes in the view file for the service (typically `ServiceCollection` and `ServiceResource`).

In the test file, the classes will match the names of the classes in the view file.

Inside of these classes should be the tests for each of the methods inside the corresponding view class.

## Method Layout and Naming
The methods in each of the test suite should follow the following naming convention;

- For each of the view methods, there should be a `test_{method}` in the test class, which tests the successful operation(s) of the view method.
    - View methods here means `list / create / read / update / delete`, and not `GET / POST / PUT / PATCH / DELETE`
    - This means that `test_update` will test the partial update at least once in order to get coverage for the method
- For every possible error code that the view method can return, there should be a `test_{method}_{error_code}` in the test class, which tests every possible instance of the chosen error code
    - The exception to this is error `404`. This is because the test for 404s is simple and therefore there can be a single `test_404` method at the end of the `ServiceResource` class
- The ordering of the test methods should follow the ordering of the view methods (`list -> create -> read -> update -> delete`)
    - Inside these groups, the methods should be ordered in order of response code (`200 -> 400 -> 403`)
- If there is to be a `test_404`, then it should be the last method of the test class

## Other Stylistic Bits
- If there are common objects in test cases, create them in `setUp`, and do all cleaning up in `tearDown` using `Model.objects.all().delete()`
- All of the test cases should have a docstring stating what is being tested and the step by step test plan
    - For methods that are checking error codes, a section explaining what causes the error and the expected error code should also be added
    - If this section is included, ensure all error codes are left aligned with each other to make it look nice
    - Also, the error codes should be in order of when they appear in the controller / permissions file, and not in order of code
- Even if it could fit on one line, break request data up to have one key: value pair per line
- Call the request data variable `data`, not anything else
- Call the url variable `url`, not `uri` or anything else
- When checking response data, we have chosen to use `response.json()` and not `response.data`, so please use `.json()`
- When using `self.assert*` methods, put the response data as the first parameter and the error code / local data as the second parameter
- Break up logical parts of test cases with a blank line and then a comment describing the next section (if there is more than one request being made in the test)
    - If the next section is testing for a large number of error codes, the comment can just be the last triplet of all the codes being tested, separated by commas (usually in `test_{method}_400`)
        - If it is possible to fit all of the full cause descriptions in the comment while keeping it on one line, this is recommended (when 2 codes being tested for example)
    - If one error code is being tested, then use the string that explains what causes the error instead (usually in `test_{method}_403`)
