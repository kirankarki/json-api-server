[![Run Status](https://api.shippable.com/projects/59b1544eb54e3d0700e6495f/badge?branch=master)](https://app.shippable.com/bitbucket/swisnl/laravel-api)
[![Coverage Badge](https://api.shippable.com/projects/59b1544eb54e3d0700e6495f/coverageBadge?branch=master)](https://app.shippable.com/bitbucket/swisnl/laravel-api)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://choosealicense.com/licenses/mit/)

# Laravel-Api
Set up a laravel API in just a few minutes with this package. All the standard API functionality is already there for you.

This package strives to save you some time while building your API. 
It already has the basic features an API should have, like:

* A generator to generate your needed files for each model
* An abstract layer to handle your basic CRUD actions
* Creates routes for all your endpoints
* Support for a few useful URL parameters
* Permission and route permission handling
* Responses in json api format (http://jsonapi.org/)
* Automatically translates your models based on your database

## Install

Via Composer

```bash
$ composer require swisnl/laravel-api
```

###Sample

Please see the folder [sample](sample) for a sample of an application using this package.

## Usage

### Base Api Classes
There are a few base classes your classes should/could inherit:

#### BaseApiController
This controller handles your basic CRUD actions as well as permissions if you choose to use permissions.

#### BaseApiRepository
This repository is a standard base repository with a small addition that it figures out your models relationships. 

If you want to use your own BaseRepository, you have to implement the RepositoryInterface. This ensures that you have full compatibility with the BaseApiController.

The BaseApiRepository uses a trait to retrieve a models relationships. You can use this trait if you want to use the existing implementation.

### Generating the required files
After installing the package you can instantly generate all the required files by executing this command:

```bash
$ php artisan laravel-api:generate-all {Model}
```

To override the default path without overriding the laravel_generator config file, you can use the `--path={path}` option. For example:

```bash
$ php artisan laravel-api:generate-all Test --path=app/temp/
```

This generates the following files:

* An eloquent model
* A translation model
* An API controller
    * Should extend the BaseApiController
* A routes file where the all the endpoints are def
* A repository for your model
    * Could extend the BaseApiRepository
* A policy for checking permissions
* 1 test for checking if a user has permissions for the endpoint

You'll be able to do the basic CRUD actions without writing anything.

You also have the ability to generate the files separately:

```bash
$ php artisan laravel-api:generate-controller {name}
$ php artisan laravel-api:generate-model {name}
$ php artisan laravel-api:generate-model-permissions {name}
$ php artisan laravel-api:generate-policy {name}
$ php artisan laravel-api:generate-repository {name}
$ php artisan laravel-api:generate-routes {name}
$ php artisan laravel-api:generate-test {name}
$ php artisan laravel-api:generate-translation {name}
```

### Configuration
If you would like to override the configuration files.

```bash
$ php artisan vendor:publish --tag=laravel-api
$ php artisan vendor:publish --tag=laravel-api-templates
```
If you decide to override the templates, make sure you override the laravel api config too. You have to define where your own templates are in the config.

This is the default configuration:

```php
   return [
     // Generator configuration
     'path' => [
         'model' => app_path('/'),
 
         'model_permissions' => app_path('Permissions/'),
 
         'translation' => app_path('Translations/'),
 
         'controller' => app_path('Http/Controllers/Api/'),
 
         'repository' => app_path('Repositories/'),
 
         'policy' => app_path('Policies/'),
 
         'auth_test' => base_path('tests/Authentication/'),
 
         'templates' => 'vendor/swisnl/laravel-api/resources/templates/',
 
         'routes' => app_path('Http/Routes/')
     ],
 
     'namespace' => [
         'model' => 'App',
 
         'model_permissions' => 'App\Permissions',
 
         'controller' => 'App\Http\Controllers\Api',
 
         'repository' => 'App\Repositories',
 
         'translation' => 'App\Translations',
 
         'policy' => 'App\Policies',
 
         'auth_test' => 'App\Tests\Authentication'
     ],
 
     // Permissions configuration
     'permissions' => [
         'checkDefaultIndexPermission' => false,
 
         'checkDefaultShowPermission' => false,
 
         'checkDefaultCreatePermission' => false,
 
         'checkDefaultUpdatePermission' => false,
 
         'checkDefaultDeletePermission' => false,
     ],
 
     // Load all relationships to have response exactly like json api. This slows down the API immensely.
     'loadAllJsonApiRelationships' => true,
]; 
```

### Requests and responses
All requests and responses are formatted according to the format specified by http://jsonapi.org/.

There are several respond methods at your disposal in your controller. The following respond methods are implemented at this moment:
```php
return $this->respondWithOk($object);
return $this->respondWithPartialContent($object);
return $this->respondWithCreated($object);
return $this->respondWithNoContent($object);
return $this->respondWithCollection($object);
```

These methods automatically converts your objects to json api format and creates a response with the correct status code and body.

### Using policies
If you decide to use policies to check for the user's pemissions you have to add the policies to your Providers\AuthServicePorvider.

```php
 protected $policies = [
     Sample::class => SamplePolicy::class,
 ];
 
 public function boot()
 {
     $this->registerPolicies();
 }    
```

If  you want to redirect the validation to a specific function in your policy.

```php
$this->authorizeAction('show');
```

If you want to check if they can request a specific object you can add that object as the second parameter:

```php
$this->authorizeAction('show', $requestedObject);
```

### URL parameters out of the box
The following URL parameters are supported after installing this package:

* ?include={relationship}: To add all relationship data to the response
* ?page={pageNumber}: To decide which page the pagination should show
* ?per_page={amountToShowPerPage}:To decide how many items you get per page
* ?ids={commaSeperatedIds}: To retrieve a collection of objects belonging to the ids
* ?lang={language}: (Requires the configure-locale middleware) to change the php locale to the desired language and automatically translates all translatable models

### Mandatory middleware
* inspect_content_type: Required. It ensures that the requests should be in json format. If it's in another format it throws a ContentTypeNotSupportedException.

### Optional middleware
There is optional middleware:

* configure-locale: used to configure the language for translating your responses. Also configurable by using the URL paramater ?lang={language}

## Passport installation

**Note: if you want to know more about laravel passport and why these commands should be run go to https://laravel.com/docs/5.5/passport**

```bash
$ composer require laravel/passport
$ php artisan migrate
$ php artisan passport:install
```
After running this command, add the ``Laravel\Passport\HasApiTokens`` trait to your ``App\User`` model

Next, you should call the ``Passport::routes`` method within the boot method of your ``AuthServiceProvider``. 

Finally, in your ``config/auth.php`` configuration file, you should set the driver option of the api authentication guard to passport. This will instruct your application to use Passport's TokenGuard when authenticating incoming API requests.

If you created your own routes make sure you have the ``auth:api`` middlware on all the routes you want to use passport with.
## Packages Laravel-Api uses

##### Laravel framework
* https://laravel.com/docs/5.5/

##### Dimsav / laravel-translatable
* https://github.com/dimsav/laravel-translatable

##### spatie / laravel-permission
* https://github.com/spatie/laravel-permission


## Change log

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) and [CODE_OF_CONDUCT](CODE_OF_CONDUCT.md) for details.

## Security

If you discover any security related issues, please email admin@swis.nl instead of using the issue tracker.

## Credits

- [Dylan de Wit][link-author]

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

[ico-version]: https://img.shields.io/packagist/v/swisnl/laravel-api.svg?style=flat-square
[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square
[ico-travis]: https://img.shields.io/travis/swisnl/laravel-api/master.svg?style=flat-square
[ico-scrutinizer]: https://img.shields.io/scrutinizer/coverage/g/swisnl/laravel-api.svg?style=flat-square
[ico-code-quality]: https://img.shields.io/scrutinizer/g/swisnl/laravel-api.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/swisnl/laravel-api.svg?style=flat-square

[link-packagist]: https://packagist.org/packages/swisnl/laravel-api
[link-travis]: https://travis-ci.org/swisnl/laravel-api
[link-scrutinizer]: https://scrutinizer-ci.com/g/swisnl/laravel-api/code-structure
[link-code-quality]: https://scrutinizer-ci.com/g/swisnl/laravel-api
[link-downloads]: https://packagist.org/packages/swisnl/laravel-api
[link-author]: https://github.com/DylandeWit
