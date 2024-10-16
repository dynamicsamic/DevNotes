---
Created: 2024-07-31T23:22
---
- Declare additional status codes to be included to OpenAPI schema
    
    ```Python
    # It is common to include the `status_code` argument to route decorator
    # for a func in FastAPI. For example
    @app.post(response_model=MyModel, status_code=201)
    async def func(...):
    		...
    
    # In this case OpenAPI schema will include two possible status codes
    # for this path: 201-defined by you and 422-defined by FastAPI.
    # But if you raise exception in the body of a handler, with a 400 status code
    # this logic will not automatically transfer to OpenAPI schema generated
    # by FastAPI. We need to implicitly include this in `responses` argument of
    # route decorator.
    @app.post(
    		response_model=MyModel,
    		status_code=201, 
    		responses={
    				409: {"model": ErrorDetail,
    							"description": "Object with this properties already exists"
    							},
    				500: {"model": ErrorDetail,
    							"description": "Server unable to perform the query"
    							}
    )
    async def func(...):
    		...
    
    class ErrorDetail(pydantic.BaseModel):
    		detail: str
    
    # Serialization of arguments passed in `responses` would not be handled
    # automatically so we need a separate model to serialize the results 
    # of our exceptions. That's why we define an `ErrorModel` model that has
    # a single attribute of `detail`, since fastapi.HttpException accepts 
    # `detail` argument to pass in additional info about the error. And this data
    # we need to provide to OpenAPI schema.
    ```
    
- Apply constraints to query parameters 
    
    ```Python
    from typing import Annotated
    from fastapi import Query
    
    
    @app.get(...)
    async def handler(
    		limit: Annotated[int, Query(gt=0)], # int > 0
    		offset: Annotated[int, Query(ge=0)], # int >= 0
    		name: Annotated[str, Query(max_length=20)] # len(str) <= 20
    ) 
    ```