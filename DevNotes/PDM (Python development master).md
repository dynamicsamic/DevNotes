---
Created: 2024-08-21T22:42
---
- `Install` pdm
    - using pip `pip install -U pdm`
    - using installation script `curl -sSL` `[https://pdm-project.org/install-pdm.py](https://pdm-project.org/install-pdm.py)` `| python3 -`
    - after installation add `pdm` to the `$PATH` to make it available in the shell by appending `PATH=$PATH:/home/<username>/.local/bin` to the `$HOME/.bashrc` file
- `Init` pdm
    - for interactive init run `pdm init` and answer several questions
    - for non-interactive init run `pdm init -n (or --non-interactive)` default values will be used
    - after initialization will be added: `pyproject.toml`, `pdm.lock`, `.pdm.python` files, as well as `tests` and `src` directories with `__init__` and `__pycache__` files in them. If you already have the `src` and `tests` directories, they won’t be overwritten
- `Install` packages
    - `pdm add <package_name>` will install a package locally and add it to `dependecies` section in `pyproject.toml`
    - To install several packages list them space-separated like `pdm add "flask=2.0" sqlalchemy`
    - `pdm add -dG test <package_name>` will install the package locally and add it to `test` subsection of `[tool.pdm.dev-dependecies]` in `pyproject.toml`
    - `pdm install --prod` will install only prod packages listed in `dependecies` section in `pyproject.toml`
- `Update` packages
    - `pdm update <package>` update one package
    - `pdm upadte <package> <package>` update multiple packages
    - `pdm update` update all packages in the lock file
    - `pdm update -dG test` update only packages listed in the `test` subseciton
- `Remove` packages
    - `pdm remove <package>` uninstall one package and remove it from `pyrpoject.toml`
    - `pdm remove -dG test pytest-cov` uninstall and remove `pytest-cov` from `test` subsection of `[tool.pdm.dev-dependencies]`
- `Run` scripts
    
    - define your scripts in `[tool.pdm.scripts]` section of `pyproject.toml` and the run them with `pdm run <script_name>`
    
    ```TOML
    # pyproject.toml
    ...
    [tool.pdm]
    [tool.pdm.scripts]
    init_db = "quart --app=main:app init_db"
    migrate = "quart --app=main:app migrate -a"
    run_app = {composite = ["init_db", "migrate"]}
    ```
    
    - by default composite scripts will not proceed if one of the parts fails. To change this behavior use `script.keep_going = True` attribute.
- `Virtual environments`
    - init virtual environment `pdm venv create --name [python.version]`
    - list virtual environments `pdm venv list`
    - `pdm` does not have a functionality to run virtual environments out of the box, but we can use a little hack first generating the virtual env command and passing it as `bash variable` to the `eval` utility like so `eval $(pdm venv activate <venv_name>)`
    - to run commands inside virtual environment use the `run` command: `pdm run --venv <venv_name> <script_name>`. E.g. `pdm run --venv todo pytest -vv -sx`