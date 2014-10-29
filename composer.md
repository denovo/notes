**Installing Composer on the Mac**

In the command line run

```
curl -sS https://getcomposer.org/installer | php
```

This will install Composer locally

To check if Composer is working  run

```
php composer.phar
```

You should see the Composer commands

```
vi composer.json
```

And add the following to the files

```
{
    "require": {
    },
    "require-dev": {
        "phpunit/phpunit": "*",
        "phpunit/phpunit-selenium": ">=1.2",
        "phpmd/phpmd" : "@stable"
    },
    "config": {
        "bin-dir": "bin"
    },
    "autoload": {
        "psr-0": {
            "stats": "",
            "": "src"
        }
    }
}

```

To install the depenedencies run

```
php composer.phar install
```
