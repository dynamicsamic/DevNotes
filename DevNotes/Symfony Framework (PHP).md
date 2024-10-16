#### Overview
---
#### Installing
```bash
# symfony requires php-fpm, php-mysql, php-common and php-curl (optionally). If you have php-psr installed, uninstall it, cause it will destroy symfony's caching mechanism.

# Install symfony cli for convinience
$ curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
$ sudo apt install symfony-cli

# Check if your php components installation satisfy symfony's requiremets
$ symfony check:requirements

# Create new project layout
$ symfony new MyProjectName --version="7.1.*" --webapp

# You can also use php composer to accomplish the same
$ composer create-project symfony/skeleton:"7.1.*" my_project_directory
$ cd my_project_directory
$ composer require webapp

# If you haven't installed composer globally use inside directory with composer.phar
$ php composer.phar ...

# Run dev server
$ cd my-project/
$ symfony server:start
```