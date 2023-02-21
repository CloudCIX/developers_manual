# Tests
Tests are where we test things, naturally.

We aim for 100% coverage, ignoring lines that we can't really test for, such as network based errors or special cases for user 1.

See [here](/style_guides/application_framework/style/4-tests.md) for the style guide on testing.

## Note for Django 2.2
In Django 2.2+, you need to declare a list of databases used at the top of the test classes. This will usually be just the application name.

Here's an example from `membership`
```python
class AddressCollection(TestBase):
    databases = ['membership']
```

An example test file is as followa
```python

# stdlib
from datetime import datetime
# libs
from cloudcix_rest.test import TestBase
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APIClient
# local
from membership.models import (
    Address,
    Country,
    Currency,
    Department,
    Language,
    Member,
    User,
)
from membership.serializers import DepartmentSerializer


class DepartmentCollection(TestBase):
    """
    Test the methods in views.DepartmentCollection -> list, create
    """
    databases = ['membership']

    def setUp(self):
        """
        Set up records that are necessary for all tests, before each test is run
        """
        # Create a currency which is necessary for a Member object
        self.currency = Currency.objects.create(
            pk=1,
            name='Euro',
            symbol='EUR',
        )

        # Create a Member which is needed as Member in test
        self.member = Member.objects.create(
            pk=46036,
            name='Company A',
            currency=self.currency,
            self_managed=True,
        )

        # Call the super set up method
        super(DepartmentCollection, self).setUp()

    def tearDown(self):
        """
        Delete all records that were created during setUp after a test case has finished.
        Please note that sometimes, other objects will be created during a test.
        These objects should be deleted in the test in which they were created directly.
        """
        # Be careful of ordering here; delete in the opposite order of creation in setUp
        Department.objects.all().delete()
        Member.objects.all().delete()
        Currency.objects.all().delete()

        # Call the super tearDown method
        super(DepartmentCollection, self).tearDown()

    def test_list(self):
        """
        Test the list method
        Test Plan:
            - Create a couple of Department records
            - Specify list parameters and submit using the APIClient
            - Ensure that response yields correct results
        """
        member2 = Member.objects.create(
            pk=12345,
            name='Company A',
            currency=self.currency,
            self_managed=True,
        )
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        Department.objects.create(
            name='Dep1',
            member=self.member,
        )
        Department.objects.create(
            name='Dep2',
            member=self.member,
        )
        Department.objects.create(
            name='Dep3',
            member=member2,
        )
        data = {
            'page': 0,
            'limit': 30,
            'order': '-name',
        }
        url = reverse('department_collection')
        response = client.get(url, data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.json()['content']), 2)
        self.assertEqual(response.json()['_metadata']['total_records'], 2)
        # Do some simple searching
        data = {
            'page': 0,
            'limit': 30,
            'order': 'name',
            'search[name__icontains]': 'dep',
            'exclude[name]': 'Dep2',
        }
        response = client.get(url, data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.json()['content']), 1)
        self.assertEqual(response.json()['_metadata']['total_records'], 1)

    def test_create(self):
        """
        Test that creation of a Department works
        Test Plan:
            - Prepare data for a new Department record
            - Use the API Client to submit the data
            - Ensure that creation was successful
        """
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        data = {
            'name': 'test create',
        }
        url = reverse('department_collection')
        response = client.post(url, data)
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.json()['content']['name'], data['name'])
        self.assertEqual(response.json()['content']['member']['id'], self.member.pk)

    def test_create_400(self):
        """
        Test that the API returns 400 errors when invalid data is sent to create a Department record
        Test Plan:
            - Provide data with invalid params
            - Submit the data using the API Client
            - Ensure that the correct error codes are returned in the response
        Error Codes:
            name not sent:  membership_department_create_101
            name too long:  membership_department_create_102
            duplicate name: membership_department_create_103
        """
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        url = reverse('department_collection')

        # Name not sent
        data = {}
        response = client.post(url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        errors = response.json()['errors']
        self.assertEqual(errors['name']['error_code'], 'membership_department_create_101')

        # Check that the error messages are correctly returned
        for k in errors:
            self.assertNotEqual(errors[k]['error_code'], errors[k]['detail'])

        # Name too long
        data = {
            'name': 'abcde' * 500,
        }
        response = client.post(url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        errors = response.json()['errors']
        self.assertEqual(errors['name']['error_code'], 'membership_department_create_102')

        # Check that the error messages are correctly returned
        for k in errors:
            self.assertNotEqual(errors[k]['error_code'], errors[k]['detail'])

        # Duplicate name
        other = Department.objects.create(
            name='Test Name',
            member=self.member,
        )
        data = {
            'name': other.name,
        }
        response = client.post(url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        errors = response.json()['errors']
        self.assertEqual(response.json()['errors']['name']['error_code'], 'membership_department_create_103')

        # Check that the error messages are correctly returned
        for k in errors:
            self.assertNotEqual(errors[k]['error_code'], errors[k]['detail'])

    def test_create_403(self):
        """
        Test that the API returns a 403 when a user tries to create a Department when they don't have permissions
        Test Plan:
            - Provide data to make a new Department record
            - Try to send the data with users without permission
            - Ensure the API responds with 403 errors each time
        Permission Errors:
            - User's member not self managed: membership_department_create_201
            - User is not admin:              membership_department_create_202
        """
        client = APIClient()
        url = reverse('department_collection')

        # User's member is not self managed
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(4143))
        response = client.post(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
        self.assertEqual(response.json()['error_code'], 'membership_department_create_201')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])

        # User is not admin
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(40334))
        response = client.post(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
        self.assertEqual(response.json()['error_code'], 'membership_department_create_202')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])


class DepartmentResource(TestBase):
    """
    Test the methods in views.DepartmentResource -> read, update, delete
    """
    databases = ['membership']

    def setUp(self):
        """
        Set up records that are necessary for all tests, before each test is run
        """
        # Create a currency which is necessary for a Member object
        self.currency = Currency.objects.create(
            pk=1,
            name='Euro',
            symbol='EUR',
        )

        # Create a Member which is needed as Member in test
        self.member = Member.objects.create(
            pk=46036,
            name='Company A',
            currency=self.currency,
            self_managed=True,
        )

        # Call the super set up method
        super(DepartmentResource, self).setUp()

    def tearDown(self):
        """
        Delete all records that were created during setUp after a test case has finished.
        Please note that sometimes, other objects will be created during a test.
        These objects should be deleted in the test in which they were created directly.
        """
        # Be careful of ordering here; delete in the opposite order of creation in setUp
        Department.objects.all().delete()
        Member.objects.all().delete()
        Currency.objects.all().delete()

        # Call the super tearDown method
        super(DepartmentResource, self).tearDown()

    def test_read(self):
        """
        Test a valid use of the read command
        Test Plan:
            - Create a new object in the DB
            - Read it via the API
            - Ensure read data matches DB
        """
        obj = Department.objects.create(
            member=self.member,
            name='Test',
        )
        data = DepartmentSerializer(instance=obj).data
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        response = client.get(obj.get_absolute_url())
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertDictEqual(response.json()['content'], data)

    def test_update(self):
        """
        Test updating a Department record
        Test Plan:
            - Create a new Department record
            - Turn it into a dict
            - Use the dict to update the record using the API
            - Ensure the changes are reflected in the DB
        """
        obj = Department.objects.create(
            member=self.member,
            name='Test',
        )
        data = DepartmentSerializer(instance=obj).data
        data['name'] = 'Update'
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        response = client.put(obj.get_absolute_url(), data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertDictEqual(response.json()['content'], data)

        # Test it with patch
        data = {
            'name': 'Patch',
        }
        response = client.patch(obj.get_absolute_url(), data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.json()['content']['name'], data['name'])

    def test_update_400(self):
        """
        Test that the update method returns a 400 when invalid data is sent
        Test Plan:
            - Provide data with missing or invalid parameters
            - Submit the data using the API Client and ensure the status is 400, and the error codes are correct
        Error Codes:
            - no name sent:   membership_department_update_101
            - name too long:  membership_department_update_102
            - duplicate name: membership_department_update_103
        """
        obj = Department.objects.create(
            member=self.member,
            name='Test',
        )
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        url = obj.get_absolute_url()

        # Name not sent
        data = {}
        response = client.put(url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        errors = response.json()['errors']
        self.assertEqual(errors['name']['error_code'], 'membership_department_update_101')

        # Check that the error messages are correctly returned
        for k in errors:
            self.assertNotEqual(errors[k]['error_code'], errors[k]['detail'])

        # Name too long
        data = {
            'name': 'abcde' * 500,
        }
        response = client.put(url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        errors = response.json()['errors']
        self.assertEqual(errors['name']['error_code'], 'membership_department_update_102')

        # Check that the error messages are correctly returned
        for k in errors:
            self.assertNotEqual(errors[k]['error_code'], errors[k]['detail'])

        # Duplicate name
        obj2 = Department.objects.create(
            member=self.member,
            name='Dupe',
        )
        data = {
            'name': obj2.name,
        }
        response = client.put(url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        errors = response.json()['errors']
        self.assertEqual(errors['name']['error_code'], 'membership_department_update_103')

        # Check that the error messages are correctly returned
        for k in errors:
            self.assertNotEqual(errors[k]['error_code'], errors[k]['detail'])

    def test_update_403(self):
        """
        Test that the API returns a 403 when a user tries to update an Department when they don't have permissions
        Test Plan:
            - Provide data to update a Department record
            - Try to send the data with users without permission
            - Ensure the API responds with 403 errors each time
        Permission Errors:
            - User's member not self managed: membership_department_update_201
            - User is not admin:              membership_department_update_202
        """
        obj = Department.objects.create(
            member=self.member,
            name='403',
        )
        Member.objects.create(
            pk=54376,
            name='Company B',
            currency=self.currency,
            self_managed=True,
        )
        client = APIClient()
        url = obj.get_absolute_url()
        # User's member is not self managed
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(206724))
        response = client.patch(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
        self.assertEqual(response.json()['error_code'], 'membership_department_update_201')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])

        # User is not admin
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(40334))
        response = client.patch(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
        self.assertEqual(response.json()['error_code'], 'membership_department_update_202')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])

    def test_delete(self):
        """
        Test the deletion of a Department record
        Test Plan:
            - Create a new department record
            - Delete it via the API
            - Ensure that the object can not be retrieved from the DB anymore
        """
        obj = Department.objects.create(
            member=self.member,
            name='403',
        )
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        url = obj.get_absolute_url()
        response = client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        with self.assertRaises(Department.DoesNotExist):
            Department.objects.get(pk=obj.pk)

    def test_delete_403(self):
        """
        Test that the API returns a 403 whenever a User attempts to delete a Department without permission
        Test Plan:
            - Create a Department record
            - Try to delete it when the User isn't allowed to
            - Ensure the API responds with 403 errors each time
        Permission Errors:
            - User's member not self managed: membership_department_delete_201
            - User is not admin:              membership_department_delete_202
            - Department isn't empty:         membership_department_delete_203
        """
        mem = Member.objects.create(
            pk=206263,
            name='Company B',
            currency=self.currency,
            self_managed=True,
        )
        obj = Department.objects.create(
            member=mem,
            name='403',
        )
        client = APIClient()
        url = obj.get_absolute_url()

        # User's member is not self managed
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(206724))
        response = client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
        self.assertEqual(response.json()['error_code'], 'membership_department_delete_201')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])

        # User is not admin
        obj.member = self.member
        obj.save()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(40334))
        response = client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
        self.assertEqual(response.json()['error_code'], 'membership_department_delete_202')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])

        # Department not empty
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42051))
        country = Country.objects.create(
            pk=999,
            english_name='Ireland',
            alpha_2_code='IE',
            alpha_3_code='IRE',
        )
        language = Language.objects.create(
            pk=1,
            native_name='English',
            english_name='English',
        )
        addr = Address.objects.create(
            pk=45791,
            member=self.member,
            name='Test Address',
            address1='delete',
            city='test',
            country=country,
            currency=self.currency,
        )
        user = User.objects.create(
            member=self.member,
            address=addr,
            first_name='Test',
            surname='Test',
            email='test@cix.ie',
            language=language,
            department=obj,
            start_date=datetime.now(),
            expiry_date=datetime.now(),
        )
        response = client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
        self.assertEqual(response.json()['error_code'], 'membership_department_delete_203')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])

        # Clean up objects not in tearDown
        user.delete()
        addr.delete()
        language.delete()
        country.delete()

    def test_404(self):
        """
        Test that invalid urls will return HTTP 404 errors
        Test Plan:
            - Send requests to invalid urls
            - Ensure the API responds with 404s
        """
        client = APIClient()
        client.credentials(HTTP_X_AUTH_TOKEN=self.get_token_for_user_id(42052))
        url = reverse('department_resource', kwargs={'pk': 420})
        response = client.get(url)
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
        self.assertEqual(response.json()['error_code'], 'membership_department_read_001')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])
        response = client.put(url)
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
        self.assertEqual(response.json()['error_code'], 'membership_department_update_001')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])
        response = client.patch(url)
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
        self.assertEqual(response.json()['error_code'], 'membership_department_update_001')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])
        response = client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
        self.assertEqual(response.json()['error_code'], 'membership_department_delete_001')
        self.assertNotEqual(response.json()['error_code'], response.json()['detail'])

```