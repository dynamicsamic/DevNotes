---
Created: 2024-08-03T11:13
---
- Add callbacks before and after the app starts
    
    ==decorate callables with==`@app.bofore_serving` ==and== `@app.after_serving`
    
- Where to find exceptions
    
    In `werkzueg` library. E.g. `from werkzeug.exceptions import NotFoun, BadRequest`
    
- Useful `quart_schema` decorators
    
    ```Python
    from quart_schema import (
    		validate_querystring, 
    		validate_request,
    		validate response
    )
    
    @app.get("/")
    # vaildates query arguments after the `?` sign in url
    @validate_querystring(MyModel)
    async def get(query_args: MyModel) 
    # In case of successful validation the decorator passes the validated
    # the pydantic model as the first argument of the decorated function named 
    # `query_args`. So you must name your first arg as `query_args`.
    
    @app.post("/")
    # validate request body
    @validate_request(MyModel)
    # In case of successful validation the decorator passes the validated 
    # the model as the first argument of the decorated function named `data`.
    # So you must name your first arg as `data`.
    async def post(data: MyModel)
    
    @app.get("/")
    # validate the response body
    @validate_response(MyModel, 201)
    async def get(obj_id: int):
    	obj = fetch.(obj_id)
    	return MyModel(**obj.to_dict()), 201
    # Note that you can return custom status_code
    ```
    
- Setup per-request context
    
    ```Python
    from quart import request 
    # request object is a proxy to contextvar, it is created on each request
    # and destroyed after it was served.
    
    @blueprint.before_request
    async def setup_request():
    		request.some_attr = some_val
    
    @blueprint.after_request
    async def add_after_logger(req):
    		log_smth
    		return request
    
    @blueprint.get(...)
    async def handler(...):
    		value = request.some_attr
    		return some_func(value)
    ```
    
- Create custom `cli`
    
    ```Python
    # The default Quart app ships with builtin cli based on click library.
    # It is capable of running any code, but it runs the code outside of the application running context.
    # We can provide our own cli in a form of a closure storing several functions
    # in it, and registring via `app.cli.command('command_name')`.
    
    # custom_cli.py
    import asyncio
    import click
    from quart import Quart
    
    async def my_func(*args, **kwargs):
    		...
    
    def custom_cli(app: Quart) -> Quart:
    		@click.option('-a', '--add_all', is_flag=True, default=False) # set first cli option
    		@click.option('-s', '--other') # set second cli option
    		@app.cli.command('make')  # register the callable with a custom name
    		def make(add_all: bool, other: str): # note this arg names must repeat what is in the option secodn parameter without dashes
    				asyncio.get_event_loop().run_until_complete(my_func(*args, **kwargs))
    
    		@app.cli.command('do')
    		def do():
    				...
    		
    		return app
    
    
    # main.py
    from quart import Quart
    from src.custom_cli import custom_cli
    
    def create_app():
    		app = Quart(__name__)
    		app = custom_cli(app)
    		return app
    
    app = create_app()
    ```