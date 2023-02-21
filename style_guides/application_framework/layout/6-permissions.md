# Permissions
Permissions are classes with static methods for each service method that might raise a 403.
These static methods are named the same as the service method that they are for, and check permissions for the method by looking at the request and any other data that is necessary.

If a service method doesn't check permissions at all then no method will be defined in the Permissions class.
Otherwise, the method will check whatever permissions it needs to.
If a permission error is found, the method must return an instance of `cloudcix_rest.exceptions.Http403`, passing in the `error_code` kwarg to show what error it is.
If nothing is returned from the permissions method, it is assumed that the permissions all passed and the view can continue on.

We expect detailed docstrings to be provided in the methods which outline all the checks that will be run inside the method in verbose English.

## Example
The following is the Permissions file for `Membership.address`;

```python
# stdlib
from typing import Optional
# libs
from cloudcix_rest.exceptions import Http403
from rest_framework.request import Request
# local
from membership.models import Address, AddressLink, Member


__all__ = [
    'Permissions',
]


class Permissions:

    @staticmethod
    def create(request: Request, member: Member) -> Optional[Http403]:
        """
        The request to create an Address is valid if;
        - The requesting User's Member is self-managed
        - The request is creating an Address in either the User's Member or a non self-managed partner Member
        """
        # The requesting User's Member is self-managed
        if not request.user.member['self_managed']:
            return Http403(error_code='membership_address_create_201')

        # The request is creating an Address in either the User's Member or a non self-managed partner Member
        if request.user.member['id'] != member.pk and member.self_managed:
            return Http403(error_code='membership_address_create_202')

        return None

    @staticmethod
    def read(request: Request, obj: Address) -> Optional[Http403]:
        """
        The request to read an Address is valid if;
        - The requesting User's Address is linked to the Address being read
        """
        # API User Allowance
        if request.user.id == 1:  # pragma: no cover
            return None

        # The requesting User's Address is linked to the Address being read
        try:
            AddressLink.objects.get(
                address_id=request.user.address['id'],
                contra_address=obj.id,
            )
        except AddressLink.DoesNotExist:
            return Http403(error_code='membership_address_read_201')

        return None

    @staticmethod
    def update(
        request: Request,
        obj: Address,
        current_cloud_region: bool,
        new_cloud_region: bool,
    ) -> Optional[Http403]:
        """
        The request to update an Address is valid if;
        - The requesting User's Member is self-managed
        - The requesting User is updating an Address in their own Member or the requesting User is updating an Address
            in a non self-managed partner Member
        - There is a link between the Address being updated and the requesting User's Address
        """
        if request.user.id == 1:  # pragma: no cover
            return None

        if current_cloud_region != new_cloud_region:
            return Http403(error_code='membership_address_update_201')

        # The requesting User's Member is self-managed
        if not request.user.member['self_managed']:
            return Http403(error_code='membership_address_update_202')

        # The requesting User is updating an Address in their own Member the requesting User is updating an Address in
        # a non self-managed partner Member
        if request.user.member['id'] != obj.member.pk and obj.member.self_managed:
            return Http403(error_code='membership_address_update_203')

        # There is a link between the Address being updated and the requesting User's Address
        try:
            AddressLink.objects.get(
                address_id=request.user.address['id'],
                contra_address=obj,
            )
        except AddressLink.DoesNotExist:
            return Http403(error_code='membership_address_update_204')

        return None
```
