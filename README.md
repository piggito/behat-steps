# Behat Steps
Collection of Behat steps for Drupal 9, Drupal 8 and Drupal 7.

[![CircleCI](https://circleci.com/gh/drevops/behat-steps.svg?style=shield)](https://circleci.com/gh/drevops/behat-steps)
[![Latest Stable Version](https://poser.pugx.org/drevops/behat-steps/v/stable)](https://packagist.org/packages/drevops/behat-steps)
[![Total Downloads](https://poser.pugx.org/drevops/behat-steps/downloads)](https://packagist.org/packages/drevops/behat-steps)
[![License](https://poser.pugx.org/drevops/behat-steps/license)](https://packagist.org/packages/drevops/behat-steps)

# Why traits?
Usually, such packages implement own Drupal driver with several contexts, service containers and a lot of other useful architectural structures.
But for this simple library, using traits helps to lower entry barrier for usage, maintenance and support. 
This package may later be refactored to use proper architecture. 

# Installation
`composer require --dev drevops/behat-steps`

# Usage
Add required traits to `FeatureContext.php`:

```
<?php

use Drupal\DrupalExtension\Context\DrupalContext;
use DrevOps\BehatSteps\ContentTrait;

/**
 * Defines application features from the specific context.
 */
class FeatureContext extends DrupalContext {

  use ContentTrait;

}
```

## Exceptions
- `\Exception` is thrown for all assertions.
- `\RuntimeException` is thrown for any unfulfilled requirements within a step. 

## Available steps

Use `behat -d l` to list all available step definitions.

There are also several pre and post scenario hooks that perform data alterations 
and cleanup. 

### Skipping before scenario hooks
Some traits provide `beforeScenario` hook implementations. These can be disabled
by adding `behat-steps-skip:METHOD_NAME` tag to your test. 

For example, to skip `beforeScenario` hook from `JsTrait`, add 
`@behat-steps-skip:jsBeforeScenarioInit` tag to the feature.

## Development

### Local environment setup
- Make sure that you have latest versions of all required software installed:
  - [Docker](https://www.docker.com/)
  - [Pygmy](https://pygmy.readthedocs.io/)
  - [Ahoy](https://github.com/ahoy-cli/ahoy)
- Make sure that all local web development services are shut down (Apache/Nginx, Mysql, MAMP etc).
- Checkout project repository (in one of the [supported Docker directories](https://docs.docker.com/docker-for-mac/osxfs/#access-control)).  
- `pygmy up`
- `ahoy build` for Drupal 9 build, or `DRUPAL_VERSION=8 ahoy build` for Drupal 8, or `DRUPAL_VERSION=7 ahoy build` for Drupal 7.
- Access built site at http://behat-steps.docker.amazee.io/  

Please note that you will need to rebuild to work on a different Drupal version.

Use `ahoy --help` to see the list of available commands.

### Apple M1 adjustments

The AIO supported version of Pygmy does not appear to receive updates any longer,
appears pygmy-go is the de-facto standard going forward. There is not a stable
release with Apple Silicon support yet, however the following dev branch is
entirely functional, so we need to build from sources.

```
pygmy down && pygmy-go down
git clone --branch arm_testing git@github.com:tobybellwood/pygmy-go.git ./pygmy-go-dev
cd ./pygmy-go-dev
make build
cd builds
cp pygmy-go-darwin-arm64 $(which pygmy)
pygmy version # should output 'Pygmy version unidentifiable.'
pygmy up
```

Copy `default.docker-compose.override.yml` to `docker-compose.override.yml`.

### Behat tests
After every `ahoy build`, a new installation of Drupal is created in `build` directory.
This project uses fixture Drupal sites (sites with pre-defined configuration)
in order to simplify testing (i.e., the test does not create a content type
but rather uses a content type created from configuration during site installation).

- Run all tests: `ahoy test-bdd`
- Run all scenarios in specific feature file: `ahoy test-bdd path/to/file`
- Run all scenarios tagged with `@wip` tag: `ahoy test-bdd -- --tags=wip`
- Tests tagged with `@d7` or `@d8` or `@d9` will be run for Drupal 7, Drupal 8 and Drupal 9 respectively.
- Tests tagged with `@d7`, `@d8` and `@d9` are agnostic to Drupal version and will run for all versions. 

To debug tests from CLI:
- `ahoy debug`
- Set breakpoint and run tests - your IDE will pickup incoming debug connection.

To update fixtures:
- Make required changes in the installed fixture site
- Run `ahoy drush cex -y` for Drupal 8/9 or `ahoy drush fua -y` for Drupal 7
- Run `ahoy update-fixtures` for Drupal 9, or `DRUPAL_VERSION=8 ahoy update-fixtures` for Drupal 8, or `DRUPAL_VERSION=7 ahoy update-fixtures` for Drupal 7 to export configuration changes from build directory to the fixtures directory. 
