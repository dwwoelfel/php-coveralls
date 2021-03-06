php-coveralls
=============

[![Build Status](https://travis-ci.org/satooshi/php-coveralls.png?branch=master)](https://travis-ci.org/satooshi/php-coveralls)
[![Coverage Status](https://coveralls.io/repos/satooshi/php-coveralls/badge.png?branch=master)](https://coveralls.io/r/satooshi/php-coveralls)
[![Dependencies Status](https://d2xishtp1ojlk0.cloudfront.net/d/9407192)](http://depending.in/satooshi/php-coveralls)

Retina-ready badge is small…
<img src="https://travis-ci.org/satooshi/php-coveralls.png?branch=master" height="10">
<img src="https://coveralls.io/repos/satooshi/php-coveralls/badge.png?branch=master" height="10">

PHP client library for [Coveralls](https://coveralls.io).

# Prerequisites

- PHP 5.3 or later
- On [GitHub](https://github.com/)
- Building on [Travis CI](http://travis-ci.org/), [CircleCI](https://circleci.com/) or [Jenkins](http://jenkins-ci.org/)
- Testing by [PHPUnit](https://github.com/sebastianbergmann/phpunit/) or other testing framework that can generate clover style coverage report

# Installation

To install php-coveralls with Composer, just add the following to your composer.json file:

```js
// composer.json
{
    "require-dev": {
        "satooshi/php-coveralls": "dev-master"
    }
}
```

Then, you can install the new dependencies by running Composer’s update command from the directory where your `composer.json` file is located:

```sh
# install
$ php composer.phar install --dev
# update
$ php composer.phar update satooshi/php-coveralls --dev

# or you can simply execute composer command if you set it to
# your PATH environment variable
$ composer install --dev
$ composer update satooshi/php-coveralls --dev
```

You can see this library on [Packagist](https://packagist.org/packages/satooshi/php-coveralls).

Composer installs autoloader at `./vendor/autoloader.php`. If you use php-coveralls in your php script, add:

```php
require_once 'vendor/autoload.php';
```

If you use Symfony2, autoloader has to be detected automatically.

Or you can use git clone command:

```sh
# HTTP
$ git clone https://github.com/satooshi/php-coveralls.git
# SSH
$ git clone git@github.com:satooshi/php-coveralls.git
```

# Configuration

Currently support clover style coverage report. php-coveralls collect coverage information from `clover.xml`.

## PHPUnit

Make sure that `phpunit.xml.dist` is configured to generate "coverage-clover" type log named `clover.xml` like the following configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit ...>
    <logging>
        ...
        <log type="coverage-clover" target="build/logs/clover.xml"/>
        ...
    </logging>
</phpunit>
```

### clover.xml

php-coveralls collects `count` attribute in a `line` tag from `clover.xml` if its `type` attribute equals to `stmt`. When `type` attribute equals to `method`, php-coveralls excludes its `count` attribute from coverage collection because abstract method in an abstract class is never counted though subclasses implement that method which is executed in test cases.

```xml
<!-- this one is counted as code coverage -->
<line num="37" type="stmt" count="1"/>
<!-- this one is not counted -->
<line num="43" type="method" name="getCommandName" crap="1" count="1"/>
```

## Travis CI

Add `php vendor/bin/coveralls` to your `.travis.yml` at `after_script`.

*Please note that `--dev` must be set to `composer install` option.*

```yml
# .travis.yml
language: php
php:
    - 5.5
    - 5.4
    - 5.3

matrix:
    allow_failures:
        - php: 5.5

before_script:
    - curl -s http://getcomposer.org/installer | php
    - php composer.phar install --dev --no-interaction

script:
    - mkdir -p build/logs
    - php vendor/bin/phpunit -c phpunit.xml.dist

after_script:
    - php vendor/bin/coveralls
```

## CircleCI

Add `pecl install xdebug` to your `circle.yml` at `dependencies` section since currently Xdebug extension is not pre-installed. `composer` and `phpunit` are pre-installed but you can install them manually in this dependencies section. The following sample uses default ones.

```yml
machine:
  php:
    version: 5.4.10

## Customize dependencies
dependencies:
  override:
    - mkdir -p build/logs
    - composer install --dev --no-interaction
    - pecl install xdebug
    - cat ~/.phpenv/versions/5.4.10/etc/conf.d/xdebug.ini | sed -e "s/;//" > xdebug.ini
    - mv xdebug.ini ~/.phpenv/versions/5.4.10/etc/conf.d/xdebug.ini

## Customize test commands
test:
  override:
    - phpunit -c phpunit.xml.dist
```

Add `php vendor/bin/coveralls` Test commands textarea on Web UI (Edit settings > Tests > Test commands textarea).

```sh
COVERALLS_REPO_TOKEN=your_token php vendor/bin/coveralls
```

*Please note that `COVERALLS_REPO_TOKEN` should be set in the same line before coveralls command execution. You can not export this variable before coveralls command execution in other command since each command runs in its own shell and does not share environment variables ([see reference on CircleCI](https://circleci.com/docs/environment-variables)).*

## From local environment

If you would like to call Coveralls API from your local environment, you can set `COVERALLS_RUN_LOCALLY` envrionment variable. This configuration requires `repo_token` to specify which project on Coveralls your project maps to. This can be done by configuring `.coveralls.yml` or `COVERALLS_REPO_TOKEN` environment variable.

```sh
$ export COVERALLS_RUN_LOCALLY=1

# either env var
$ export COVERALLS_REPO_TOKEN=your_token

# or .coveralls.yml configuration
$ vi .coveralls.yml
repo_token: your_token # should be kept secret!
```

php-coveralls set the following properties to `json_file` which is sent to Coveralls API (same behaviour as the Ruby library will do except for the service name).

- service_name: php-coveralls
- service_event_type: manual

## .coveralls.yml

php-coveralls can use optional `.coveralls.yml` file to configure options. This configuration file is usually at the root level of your repository, but you can specify other path by `--config (or -c)` CLI option. Following options are the same as Ruby library ([see reference on coveralls.io](https://coveralls.io/docs/ruby)).

- `repo_token`: Used to specify which project on Coveralls your project maps to. This is only needed for repos not using CI and should be kept secret
- `service_name`: Allows you to specify where Coveralls should look to find additional information about your builds. This can be any string, but using `travis-ci` or `travis-pro` will allow Coveralls to fetch branch data, comment on pull requests, and more.

Following options can be used for php-coveralls.

- `src_dir`: Used to specify where the root level of your source files directory is. Default is `src`. 
- `coverage_clover`: Used to specify the path to `clover.xml`. Default is `build/logs/clover.xml`
- `json_path`: Used to specify where to output `json_file` that will be uploaded to Coveralls API. Default is `build/logs/coveralls-upload.json`.

```yml
# .coveralls.yml example configuration

# same as Ruby lib
repo_token: your_token # should be kept secret!
service_name: travis-pro # travis-ci or travis-pro

# for php-coveralls
src_dir: src
coverage_clover: build/logs/clover.xml
json_path: build/logs/coveralls-upload.json
```

# Planned

- `environment` in `json_file` (not documented but implemented in ruby lib)
- Refactor test cases
- Support commands
    - `push` to run locally
    - `open` to open "https://coveralls.io/repos/${token}"
    - `service` to open "https://coveralls.io/repos/${token}/service"
    - `last` to open "https://coveralls.io/repos/${token}/last_build"

# Versions

## 0.4

- Replace REST client implementation by [guzzle/guzzle](https://github.com/guzzle/guzzle)
- Change: `repo_token` is required on CircleCI, Jenkins

## 0.3

- Better CLI implementation by using [symfony/Console](https://github.com/symfony/Console) component
- Support `--dry-run`, `--config (-c)` CLI option

## 0.2

- Support .coveralls.yml

## 0.1

- First release
- Support Travis CI (tested)
- Implement CircleCI, Jenkins, local environment (but not tested on these CI environments)
- Collect coverage information from clover.xml
- Collect git repository information