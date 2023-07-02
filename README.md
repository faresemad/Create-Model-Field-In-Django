# Create Model Field In Django
Create or Customize model field in Django

## Create IntegerListField in Django
```python
from django.db import models

class IntegerListField(models.TextField):
    description = "List of integers"

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def from_db_value(self, value, expression, connection):
        if value is None:
            return []
        return [int(i) for i in value.split(',')]

    def to_python(self, value):
        if isinstance(value, list):
            return value
        if value is None:
            return []
        return [int(i) for i in value.split(',')]

    def get_prep_value(self, value):
        return ','.join(str(i) for i in value)
```

> Let's go through each of the methods:

- description is an optional attribute that provides a human-readable description of the field.
- __init__ is the constructor for the field. In this case, we just call the constructor of the parent class (TextField) with the same arguments.
- from_db_value is called when the field is loaded from the database. It takes the raw value as input and should return a Python object that represents the field value. In this case, we split the string value by commas and convert each item to an integer.
- to_python is called when the field value is accessed as a Python object. It takes the raw value as input and should return a Python object that represents the field value. In this case, we do the same conversion as in from_db_value.
- get_prep_value is called when the field value is saved to the database. It takes the Python object as input and should return the raw value that will be saved. In this case, we join the list of integers with commas and convert each integer to a string.

> Once you have defined the custom field, you can use it in your models like any other field:

```python
class MyModel(models.Model):
    my_field = IntegerListField()
```

## Create PhoneNumberField in Django

```python
from django.db import models
from phonenumbers import parse, format_number, PhoneNumberFormat
from django.core.exceptions import ValidationError

class PhoneNumberField(models.CharField):
    description = "Phone number field"

    def __init__(self, *args, **kwargs):
        kwargs['max_length'] = 20
        super().__init__(*args, **kwargs)

    def from_db_value(self, value, expression, connection):
        if value is None:
            return None
        try:
            return format_number(parse(value), PhoneNumberFormat.INTERNATIONAL)
        except:
            return value

    def to_python(self, value):
        if value is None:
            return None
        try:
            return format_number(parse(value), PhoneNumberPhoneNumberFormat.INTERNATIONAL)
        except:
            raise ValidationError('Invalid phone number format')

    def get_prep_value(self, value):
        if value is None:
            return None
        try:
            return format_number(parse(value), PhoneNumberFormat.E164)
        except:
            raise ValidationError('Invalid phone number format')
```

## Create SlugIDField in Django

```python
from django.db import models
from django.utils.text import slugify

class SlugIDConverter:
    regex = '[a-zA-Z0-9]+'

    def to_python(self, value):
        return value

    def to_url(self, value):
        return str(value)

class SlugIDField(models.SlugField):
    description = "Slugified ID"

    def __init__(self, *args, **kwargs):
        kwargs['editable'] = False
        kwargs['unique'] = True
        kwargs['max_length'] = 32
        kwargs['allow_unicode'] = True
        kwargs['default'] = self._default
        super().__init__(*args, **kwargs)

    def deconstruct(self):
        name, path, args, kwargs = super().deconstruct()
        del kwargs['editable']
        del kwargs['unique']
        del kwargs['max_length']
        del kwargs['allow_unicode']
        del kwargs['default']
        return name, path, args, kwargs

    def _default(self):
        return slugify(str(self.model.objects.latest('pk').pk), allow_unicode=True)

    def get_converter(self):
        return SlugIDConverter()

    def get_prep_value(self, value):
        return value if value else self._default()
```