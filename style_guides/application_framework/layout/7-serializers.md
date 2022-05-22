# Serializers
Serializers are used to turn objects into dictionaries, which can then be sent out as Responses.
We use the [Serpy](https://github.com/clarkduvall/serpy) library for serializers instead of the Django Rest Framework's Serializers because they're faster, but they require you to specify all of the fields that need to be returned.

An example serializer is as follows;
```python
# libs
import serpy
# local
from membership.serializers.address_link import AddressLinkSerializer
from membership.serializers.country import CountrySerializer
from membership.serializers.currency import CurrencySerializer
from membership.serializers.language import LanguageSerializer
from membership.serializers.member import MemberSerializer
from membership.serializers.subdivision import SubdivisionSerializer


__all__ = [
    'AddressSerializer',
]


class AddressSerializer(serpy.Serializer):
    """
    Serializes models.Address instances into dict representations
    """
    address1 = serpy.Field()
    address2 = serpy.Field()
    address3 = serpy.Field()
    billing_address_id = serpy.Field()
    city = serpy.Field()
    country = CountrySerializer()
    currency = CurrencySerializer()
    email = serpy.Field()
    full_address = serpy.Field()
    gln = serpy.Field(required=False)
    id = serpy.Field()
    language = LanguageSerializer()
    link = AddressLinkSerializer(required=False)
    linked = serpy.Field(required=False)
    member = MemberSerializer()
    name = serpy.Field()
    phones = serpy.Field()
    postcode = serpy.Field()
    subdivision = SubdivisionSerializer(required=False)
    uri = serpy.Field(attr='get_absolute_url', call=True)
    vat_number = serpy.Field()
    website = serpy.Field()

    # Backwards Compatibility
    idMember = serpy.Field(attr='member_id')
    idCountry = serpy.Field(attr='country_id')
    idSubdivision = serpy.Field(attr='subdivision_id')
    idCurrency = serpy.Field(attr='currency_id')
    idLanguage = serpy.Field(attr='language_id')
    idAddressBilling = serpy.Field(attr='billing_address_id')
    fullAddress = serpy.Field(attr='full_address')
    idAddress = serpy.Field(attr='id')
    companyName = serpy.Field(attr='name')
    vatNumber = serpy.Field(attr='vat_number')
```