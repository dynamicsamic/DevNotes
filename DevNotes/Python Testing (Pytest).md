---
Created: 2024-07-24T22:53
---
- Mark a Pytest fixture async
    
    ```Python
    @pytest_async.mark.fixture
    async def some_fixture(): pass
    ```
    
- Run pytest_asyncio tests in single event loop
    
    ```Python
    # pytest_asyncio runs each testcase in a separate event loop 
    # to provide maximum isolataion level between tests. Meanwhile to use 
    # asyncpg postgres driver's pool funcionality you need to provide a single
    # event loop for all connections in the pool, ohterwise an InterfaceError
    # is raised. We can make all tests to be collected by pytest_asyncio as
    # session scoped to guarantee a single event loop. But you should be careful
    # since now changes in one test will be will be visible in all other tests.
    # https://pytest-asyncio.readthedocs.io/en/latest/concepts.html
    
    # project/tests/conftest.py
    import pytest
    from pytest_asyncio import is_async_test
    
    def pytest_collection_modifyitems(items):
        pytest_asyncio_tests = (item for item in items if is_async_test(item))
        session_scope_marker = pytest.mark.asyncio(scope="session")
        for async_test in pytest_asyncio_tests:
            async_test.add_marker(session_scope_marker, append=False)
    ```
    
- Mark all Pytest test in a module async
    
    `pytestmark = pytest.mark.asyncio # define this global constant at the top of a module`
    
- Pytest test verbosity options
    - `-s` ==output all logs and prints to== `stdout`
    - `-v | -vv | -vvv` ==different verbosity levels==
    - `--log-cli-level=INFO` ==send all logs to== `stdout`
    - `-q` ==run tests in quiet mode==
- Mock an async instance method with AsyncMock
    
    ```Python
    from unittest.mock import AsyncMock
    
    # suppose we need to test `get_obj` method of MyClass class
    # `get_obj` in turn calls `fetch` method of `Repository` class
    # `fetch` method has already been tested in different scenarios, trus its 
    # output, so there is no point in testing it again. We want to test
    # the functionality of our `get_obj` method. In addition `fetch` method is 
    # expensive so skipping it means we can save time and resources.
    # what we need to do is to mock `fetch` method and check if our `get_obj` 
    # function calls it with expected arguments and correctly processed its output  
    
    # import the class that own `get_obj` method
    from my_module import MyClass 
    
    # define a mock class that would pretend it owns `fetch` method we need to mock
    class Repository:
    		pass
    
    # define your testcase
    async def test_my_class_get_obj_method():
    		repo = Repository() # define a mock object
    		# attach mocked method to the mock object and pass it a return value
    		# that your `get_obj` tested function expects to process
    		repo.fetch = AsyncMock(return_result=model_obj)
    		
    		# define an object to invoke `get_obj` method
    		service = MyClass(repo)
    		
    		# await your method with predefined arguments
    		result = await service.get_obj(*args, **kwrags)
    		
    		# assert that the mocked `fetch` method was called once with passed args
    		repo.fetch.assert_awaited_once_with(args, kwargs)
    		
    		# make further assertion about the result of calling your 
    		# `get_obj` method   
    		assert result ...
    ```
    
- Mock an async instance method with patch
    
    ```Python
    from unittest.mock import patch
    import MyModel
    
    # using patch as method decorator
    @patch.object(MyModel, "method_name", return_value=some_value) 
    async def test_func(mock): # this mock argument comes from the decorator
    		...
    		mock.assert_awaited_with(some_value)
    
    # using patch as context manager
    async def test_func():
    		some_value = ...
    		with patch.object(MyModel, "method_name", return_value=some_value) as mock:
    				...
    				mock.assert_awaited_with(some_value)
    ```
    
      

#### If `patch` is not applied?
```python
# For patches to be applied you need to patch those objects or functions
# that being used directly in the calling code.

# Example.
# helpers.py
def helper(...):
	...

# main.py
from .helpers import helper

def main(...):
	...
	val = helper(...)
	...

# tests.py
from unittes.mock import patch

# This mock won't be applied, because you not the exact object that is
# being used.
@patch("helpers.helper")
def test(mock):
	...

# Instead do this
@patch("main.helper")
def test(mock):
	...
```

- Fixture `autouse` in every method of a `test class`
    
    ```Python
    # Suppose there exists a `global` fixture with a module scope
    # tests/fixtures.py
    @pytest_asyncio.fixture(scope="module")
    async def fixt():
    		perform_some_action()
    		yield
    
    # tests/unit/api/test_api.py
    from tests.fitures import fixt
    
    class TestAPI:
    		@pytest_asyncio.fixture(autouse=True, scope="module")
    		async def setup(self, fixt):
    				self.__class__.fixture = fixt
    		
    		async def test_foo(self):
    				await self.fixt()
    				
    ```
    
- Module level `autouse mock` 
    
    ```Python
    # Sometimes you need a mock object that will be static for the lifetime 
    # of the whole test module or even session which all your test functions 
    # will use. And you also need to make it not clutter all your functions 
    # with useless arguments. You can create a pytest fixture with `autouse` 
    # option and a mock object.
    from unittest.mock import patch, AsyncMock
    import pytest
    
    @pytest.fixture(scope="module", autouse=True)
    def connection_pool():
    		with patch.object(app, "connection_pool", create=True, new_callable=AsyncMock) as mock:
    				yield mock
    
    # Note: this thing may produce incorrect results with pytest-asyncio fixtures
    ```
    
- `Pytest` configs in `pyproject.toml`
    
    ```TOML
    [tool.pytest.ini_options]
    pythonpath = "." # set to run tests from root directory
    asyncio_default_fixture_loop_scope = "module" # set global loop_scope for fixtures 
    ```