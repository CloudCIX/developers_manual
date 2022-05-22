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
        Check that a given request to create an Address record is valid
        Valid if:
            - The user's member is self managed
            - Is for the users menber or a non self managed partner
        :param request: The request object representing the user's request
        :param member: The member object the Address is associated with
        """
        # The user's member is self managed
        if not request.user.member['self_managed']:
            return Http403(error_code='010-114-002-001-026')
        # Is for the users menber or a non self managed partner
        if request.user.member['id'] != member.pk and member.self_managed:
            return Http403(error_code='010-114-002-001-033')

        return None

    @staticmethod
    def read(request: Request, obj: Address) -> Optional[Http403]:
        """
        Check that a given request to read an Address record is valid
        Valid if:
            - The user is persoa
            - The user's address is linked to the address object
        :param request: The request object representing the user's request
        :param obj: The Address that is trying to be read
        """
        if request.user.id == 1:  # pragma: no cover
            return None
        # The user's address is linked to the address object
        try:
            AddressLink.objects.get(
                address_id=request.user.address['id'],
                contra_address=obj.id,
            )
        except AddressLink.DoesNotExist:
            return Http403(error_code='010-114-002-003-003')
        
       return None

    @staticmethod
    def update(request: Request, obj: Address) -> Optional[Http403]:
        """
        Check that a given request to update an Address record is valid.
        Valid if:
            - The user's member is self managed
            - The user is updating an address in their own member
            - The user is updating an address in a non self managed partner
            - There is a link between the address and the user's address
        :param request: The request object representing the sent request
        :param obj: The Address record trying to be updated
        """
        # The user's member is self managed
        if not request.user.member['self_managed']:
            return Http403(error_code='010-114-002-004-006')
        # The user is updating an address in their own member or an address in a non self managed partner
        if request.user.member['id'] != obj.member.pk and obj.member.self_managed:
            return Http403(error_code='010-114-002-004-007')
        # There is a link between the address and the user's address
        try:
            AddressLink.objects.get(
                address_id=request.user.address['id'],
                contra_address=obj,
            )
        except AddressLink.DoesNotExist:
            return Http403(error_code='010-114-002-004-030')

        return None
```