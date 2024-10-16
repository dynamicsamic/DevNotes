#### Installing litestar
```bash
uv add litestar[standard]
# this will install litestar and uvicorn
```
---
#### Run litestar app
```bash
litestar --app=src.folder.app:app run -r
# provide the app instance and run the litestar with reload
# source file update option. This runs uvicorn on port 8000

# Alternatively run uvicorn yourself
uvicorn --reload src.folder.app:app
```
---
#### Defining handlers
```python
# Endpoint handlers can be specifed both on app instance, on routers and controllers.

#### Defining handlers on directly on app instance #####
from litestar import Litestar, get

@get("/")
async def index(...):
	...

app = Litestar(route_handlers=[index])

#### Defining handlers with routers. ####
# Using routers is more FastAPI-like way, when you define a router per
# a module and add this module handlers to the router.
# Then this router should be added to apps routers.

# handlers/items.py
from litestar import get, post, Router

@get("/items")
async def get_item(...):
	...

@post("/items")
async def add_item(...):
	...

router = Router(
	path="/warehouse",
	route_handlers=[get_item, add_item],
	dependencies=[...]
)

# handlers/__init__.py
from litestar import Router
from .items import router as item_router
from .things import router as thing_router

router = Router(path="/api", route_handlers=[item_router, thing_router])

# app.py
from litestar import Litestar
from .handlers import router

app = Litestar(route_handlers=[router])

#### Defining handlers with controllers ####
# Controllers in litestar are like class-based views or viewsets in Django.
# They allow define multiple different handlers on one controller sharing
# common properties between them.

# controllers/items.py
from litestar import Controller, post, patch

class ItemController(Controller):
	path = "/tems"
	dependecies = ...
	cache_control = ...
	...

	@post("/")
	async def insert(self, ...):
		...

	@patch("/{pk: int}")
	async def update(self, pk, ...):
		...
		

# app.py
from litestar import Litestar
from .controllers.items import ItemController

app = Litestar(route_handlers=[ItemsController])
```
---
#### Dependecies
```python
# Dependecies are sync or async callables that can be defined simultaneously
# on per-handler, per-controller, per-router, per-app basis.
# By default sync callables run in a separate thread to avoid potetially
# blocking the event loop, which has its cost. If you use small sync callables in
# your dependecies that don't use IO, you may want to use `sync_to_thread=False`
# settings.

from litestar import get, Router
from litestar.di import Provide
# Dependecies are provided to routers with `Provide` class

# This `service` dependecy is provided as named argument to decorated function.
# Type annotation for the argument must be provided.
@get("/", dependencies={"service": Provide(provide_service)})
async def handler(serice: ServiceType, ...):
	...

router = Router(
	route_handlers=[handler], 
	dependencies={"db_session": Provide(provide_db_session)},
)
# `db_session` argument is present on every handler that belong to this router.
# Other depedencies like `provide_service` can also use `db_session`as
# subdependecies. They need to define type-annotated attrbute named `db_session.

# By default `Provide` class validates dependecies, which cost some performance.
# This can be avoided by using the `Dependecy` function
from typing import Annotated
from litestar.params import Dependency

@get("/")
async def handler(
	service: Annotated[ServiceType, Dependency(skip_validation=True)]
):
	...

# Dependecies may be provided as async generators using yield to implement
# clean up behavior. In this case the best practice is to wrap yield part
# try/finally block.
from contextlib import asynccontextmanager
from typing import AsyncGenerator
from litestar.datastructures import State

@asynccontextmanager
async def provide_db_session(state: State) -> AsyncGenerator[None, None]:
	# the `state` arg of type State is always present on every request
	try:
		async with state.db_engine.create_session() as session:
			yield session
	finally:
		await session.close()

```
---
#### Builtin dependencies
```python
# Litestar has predefined dependecies that may be injected in 
# every router handler. All that you need is to declare special kwarg
# with its type annotation.
from litestar import Request
from litestar.connection import WebSocket
from litestar.datastructures.cookie import Cookie
from litestar.datastructures.headers import Header
from litestar.datastructures.state import State
from litestar.param import Parameter

@get("/items")
async def handler(
	request: Request,  # current request object
	state: State,  # a copy of the global app state
	headers: dict[str, Header],  # a dict of request headers
	cookies: dict[str, Cookie],  # a dict of request cookies
	query: dict[str, Parameter], # a dict of query parameters
	socket: WebSocket,  # websocket instance
	body: bytes  # raw request body 
)
```

#### DTOs
```python
# DTO is a mechanism to validate and serialize request input data and
# response output data.

# DTOs are defined in the router method definition.
# After validation and serialization data from DTO gets passed to
# decorated handler as `data` argument.
@get("/", dto=InputModelDTO, return_dto=OutputModelDTO)
async def handler(data: InputModel) -> OutputModel:
	...

# DTOs are type annotations that may recieve dataclasses, sqlalchemy orm models,
# pydantic models and some other data containers.
# To make a DTO from your model, import the appropriate DTO class and 
# create new type annotation.
from litestar.contrib.pydantic import PydanticDTO
from .models import MyModel

MyDTO = PydanticDTO[MyModel]

# Additional configuration for DTO available via `DTOConfig` dataclass.
from typing import Annotated
from litestar.contrib.pydantic import PydanticDTO
from litestar.dto import DTOConfig

MyDTO = PydanticDTO[
	Annotated[
		MyModel,
		DTOConfig(
			# transform attr names in output model
			rename_strategy="camel",
			# rename specific attrs after. Applied after transformation
			rename_fields={"attr_name": "alias"},
			# forbid extra attrs provided in the input data
			forbid_unknown_fields=True
		)
	]
] 
```
---
#### Parameters
```python
# Path parameters are defined in curly braces in router decorator's `path` argument. After validation and serialization this variable is passed to
# the decorated handler.

@get("/items/{item_id: int}")
async def handler(item_id: int) -> Item:
	...

# Query parameters are extracted from the url by parsing the part after `?`
# sign and passed to decorated handler only if the paramater and argument
# names match.

@get("/items")
async def handler(limit: int) -> Item:
	# For the url `/items?limit=10` the `limit` will be passed to the handler,
	# but if we had `/items?limits=10` the `limits` will be ignored.
	...

# Query parameters can be enhanced with `Parameter` class, which adds
# additional validation from litestar and additional info to OpenAPI spec.
from litestar.openapi.spec import Example
from litestar.params import Parameter

@get("/items")
async def handler(
	page_size: int = Parameter(
		int,  # value type
		query="limit",  # parse parameter named limit
		required=False,  # mark as not required
		gt=0,  # validate not negative
		lt=101,   # validate less than 101
		default=10,  # define a default value
		# generate `examples` section for openapi schema 
		examples=[Example(summary="example1", value="limit=10")]		
	)
)
```
---
#### Logging in route handlers
```python
# The best practice is to use litestar's non-blocking logging mechanisms
# available on builtin `request` dependency.
from litestar import Request

@get("/")
async def handler(request: Request) -> Item:
	...
	request.logger.info("log message")
	...
```