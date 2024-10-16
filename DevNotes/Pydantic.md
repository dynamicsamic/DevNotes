
#### Useful configs
```python
from pydantic import BaseModel, ConfigDict
    
class MyModel(BaseModel):
	model_config = ConfigDict(
		# allow validate various objects with BaseModel.model_validate(object)
		from_attributes=True 
		# allow using user defined classes in model field type hints
		arbitrary_types_allowed=True
		# raise ValidationError if extra fields present
		extra="forbid"
		# prevent type coercion, e.g str '33' to int 33
		strict=True
		# any callable that will modify attr names after validation
		# applied to all attrs
		alias_generator=to_camel
	)
```
---
#### Tips
```python
# validate object
MyModel.model_validate(obj)

# convert to dict
model_obj.model_dump(exclude_none=True) # removes all None fields

# field validators
from pydantic import EmailStr # validates emails
from pydantic import StringConstraints(min_length, max_length, ...)

# Create field with default callable
from pydantic import Field
from datetime import datetime

class MyModel(BaseModel):
	due: datetime = Field(default_factory=datetime.now) 

# Create custom constraint fields
from typing import Annotated
from pydantic import Field

PositiveInt = Annotated[int, Field(gt=0)]
```
---
#### Validate the whole model not just a field
```python
from pydantic import BaseModel, model_validator

class MyModel(BaseModel):

	@model_validator(mode='before')
	@classmethod
	def validate_some_thing(cls, data: Any)-> Any:
		# do something with data and return it
		# note that data may be anything that you pass into model
		# such as a dict, another model, any object with attributes,
		# so be careful with handling this argument
```
---
#### Create reusable validators
```python
# We can create composite types to enhance fileds with additional validation
# or new functionality. 
# E.g. we can add a timezone to a datetime received from user.
from datetime import datetime
from typing import Annotated
from pydantic import BaseModel
from pydantic.functional_validators import AfterValidator
import settings

def add_timezone(dt: datetime) -> datetime:
	return dt.astimezone(settings.TIMEZONE)

TimezoneDatetime = Annotated[datetime, AfterValidator(add_timezone)]

class MyModel(BaseModel):
	created_at: TimezoneDatetime | None = None
```
---
#### Remove inherited parent field from a subclass
```python
# We have a parent class with several fields
from pydantic import BaseModel

class Parent(BaseModel):
	username: str
	email: str
	user_id: int
	
# We need to create a subclass but want to remove the `user_id` field
# both from validation and the OpenAPI spec
from pydantic import Field
from pydantic.json_schema import SkipJsonSchema

class Child(Parent):
	user_id: SkipJsonSchema[int] = Field(default=0, exclude=True)

child = Child(username='bob', email='...') 
# The `user_id` is not validated nor present in the `child` attributes.
```
---
#### Pydantic settings
```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
	model_config = SettingsConfigDict(enf_file='.env')
	# This will use python_dotenv library to load env vars
	
	TIMEZONE: TZ = ...
	SECRET: str = ...
```
---
#### Create a field alias
```python
from pydantic import BaseModel, Field

class MyModel(BaseModel):
	item_id: Annotated[int, Field(alias="pk")]
	# now MyModel accepts `pk` field of int type and converts it
	# to `item_id` field after validation
```
---



