---
Created: 2024-07-24T22:44
---
#### Define `UUID` filed
```python
from sqlalchemy import func
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy.types import Uuid

class MyModel(Base):
	guid: Mapped[uuid.UUID] = mapped_column(
		Uuid, 
		# set postgres function to generate default values
		server_default=func.gen_random_uuid(),
	    # or generate default valuess on python's side 
	    default=uuid.uuid4
	)
```
---
#### `Async Session` notes
* *session generated with `async_sessionmaker` are bound to an async engine which spawns a connection from its connection pool (we can eliminate the pool passing `poolclass=NullPool` argument go `create_async_engine`).
* when session is being used as a context manager `async with async_session_maker() as session:` it will automatically invoke await `session.close()` method after leaving the indented block (or await `session.rollback` if an exception occurred). After closing the session, all data that was bound to this session will be discarded, and the connection will be returned to the pool.
* session also may spawn a sqlalchemy transaction async with `session.begin():` ... which will automatically invoke await `session.commit()` / `await session.rollback` and `await session.close()` upon leaving the context.
* async sesssion’s `add` and `add_all` methods are not coroutines, they invoked without the `await` keyword, while `commit`, `refresh`, `execute`, `rollback`, `close` and some others are coroutines and need the `await`.
---
#### Difference between `CORE` and `ORM` DML operations
###### `Insert`
```python
# ORM mode, makes two separate queries for insert and retrieve
async with session_maker() as session:
	instance = ModelType(**create_kwargs)
	session.add(instance)
	await session.commit()
	await session.refresh(instance)
	return instance
	
# Core mode, makes one query with RETURNING instruction
async with session_maker() as session:
# also works with engine.connection()
result = await session.execute(
	sa_insert(ModelType)
	.values(**create_kwargs:dict[str, Any])
	.returning(ModelType)
await session.commit()
return result.scalars()

# Brief insert many examples

# ORM mode
session.add_all(Iterable[ModelInstance])
await session.commit()

# Core mode
result = await session.execute(
	sa_insert(ModelType)
	.values(*create_kwargs_list: Iterable[dict[str, Any]])
	.returning(ModelType)
await session.commit()
return result.scalars()
```
###### `Delete`
```python
# ORM mode
await session.delete(ModelInstance)
await session.commit()

# Core mode
await session.execute(
	sa_delete(ModelType)
	.where(*filters)
	# can return result with .returning instruction
```
###### `Update`
```python
# there is no session.update method, so no difference here really
await session.execute(
	sa_update(ModelType)
	.where(*filters)
	.values(**update_kwargs)
	.returning(ModelType)
```
---
#### Setting `default` values on `sqlalchemy models`
```Python
# Default values can be set either from Python side or from RDBMS side

# Default value from Python side may be a constant or a callable
def set_default_day():
	return datetime.date.today()

class MyModel(Base):
	name: Mapped[str] = mapped_column(String(60), default="DefaultName")
	date: Mapped[datetime.date] = mapped_column(Date, default=set_default_day)

# Default value from RDBMS side is set via schema definition and may be
# either a scalar value or a valid sql function
class MyModel(Base):
	name: Mapped[str] = mapped_column(String(60), server_default=func.now())
	col1: Mapped[int] = mapped_column(server_default=text("0"))
```
---
#### Setting `one-to-many relationship` between two tables
```Python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import Mapped, mapped_column,relationship

class Parent(Base):
	id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
	children: Mapped[list["Child"]] = relationship(
		# Set `parent` attr on child's instance
		back_populates="parent",
		# Prevent accidental load of related objects. 
		# Also possible `joined`, `selectin`, `raise`
		lazy="noload"
		 # All children will be updated at parent update or deleted 
		 # on parent delete.
		cascade="all, delete-orphan"
		# No need to load children when deleting parent. 
		# Works only when cascade is set on the other side of relationship.
		passive_deletes=True
	)

class Child(Base):
	id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
	# set parent_id foreign key and cascading behavior
	parent_id: Mapped[int] = mapped_column(
		ForeignKey(Parent.id, ondelete="CASCADE")
	)
	parent: Mapped[Parent] = relationship(
		back_populates="children",
		lazy="noload"
	)
```
---
#### `Join` realted objects in select query
```python
# Eager loading - is a technique for loading realted objects defined on
# ORM model. It can be defined as such

class MyModel(Base):
	pk: Mapped[int]
	another: Mapped[AnotherModel] = relationship(lazy="joined")
	third: Mapped[ThirdModel] = relationship(lazy="joined")

# Using this setting emits automatic join statements for related data 
# when querying the model. This behavior may be undesirable due to
# low control of the relation loading process.

# To allow selective relation loading we can use `noload` on relationship
# definition. And load relations individually for each query.

from sqlalchemy import select
from sqlalchemy.orm import joinedload, load_only

from .models import MyModel

async def query():
	select(MyModel)
	.options(
		# join AnontherModel and load only its name (and foreign key)
		joinedload(MyModel.another).options(load_only(AnotherModel.name)),
		# join ThirdModel and load only its pk (and foreign key)
		joinedload(MyModel.third).options(load_only(ThirdModel.pk)),
	)
	.where(MyModel.pk==12)
	.order_by(MyModel.name)

# If you want exclude some fields you can use `defer` instead of `load_only`.
# Accessing attributes of related object that was not loaded results in 
# additional query for loading them, or in async mode results in detached error.
```
---
#### Compare model attribute's expressions
```python
# To compare attrbute expressions use `compare` method.
class MyModel(Base):
	pk: Mapped[int] = mapped_column()
	...

MyModel.pk.in_([1,2,3]).compare(MyModel.pk>1)  # -> False
```
---
#### ORM model fields declaration
```python
from datetime import datetime
from typing import Optional
from sqlalchemy import DateTime
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
	pass

class MyModel(Base):
	# Declare INT NOT NULL column
	num: Mapped[int]
	# Declare INT column
	num: Mapped[Optional[int]]
	# Declare INT column with additional properties
	pk: Mapped[int] = mapped_column(autoincrement=True, primary_key=True)
	# Declare timestamp column with additional properties
	created: Mapped[datetime] = mapped_column(DateTime(timezone=True))
```
---
#### Load relations with `limit`
```python
# When using `joinedload` or `selectinload` for loading related objects
# there is no way to limit the amount of loaded items. By default all realted
# items will be loaded. To have a possibility to limit them we need to
# emit a separate select statement for items that should be limite and
# then we should use this select in the `JOIN` clause.

# We have a Parent class, which has its children items assigned to the `children`
# attribute. The Child class has far to many items to be loaded at once
# thus we would want to limit the amount. Additionally the Child class
# has realted `tag` attrbiute which is a reference to the `Tag` class, which
# in turn has two more related attrbutes `category` and `grade` that reference
# to `Category` and `Grade` classes respectively. We need to load them too.

from sqlalchemy import select
from sqlalchemy.orm import joinedload, contains_eager, defer, load_only

# First we define a query for the `Child` table.
child_qry = (
	select(Child)
	.where(...)
	.order_by(...)
	.limit(10)
	.offset(2)
	.subquery('subqry_name')
	.lateral()
)

main_qry = (
	select(Parent)
	.where(...)
	.limit(...)
	.outerjoin(child_qry)
	.options(
		contains_eager(Parent.children, alias=child_qry).options(
			defer(Child.created_at),
			defer(Child.updated_at),
			joinedload(Child.tag).options(
				defer(Tag.created_at),
				joinedload(Tag.category),
				joinedload(Tab.grade).options(
					load_only(Grade.value)
				)
			)
		)
	)
)

(await session.scalars(main_qry)).unique()

```
---
